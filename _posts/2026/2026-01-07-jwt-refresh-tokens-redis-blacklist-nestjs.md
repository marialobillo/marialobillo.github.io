---
layout: post
title: "JWT Refresh Tokens with Redis Blacklist in NestJS"
date: 2026-01-07
categories: [nestjs, security, redis, backend]
tags: [jwt, redis, nestjs, auth, backend, typescript]
---

While building the auth layer for a document management platform I was working on, 
I ran into a question that sounds simple 
but isn't: what actually happens when a user logs out?

With basic JWT, the answer is uncomfortable: not much. You delete 
the token from the client, sure. But the token itself is still valid. 
If someone grabbed it before you logged out, they can keep using it 
until it expires. That could be hours.

That didn't feel right. So I started thinking about how to actually 
invalidate a token on logout.

## The approach: two tokens + a blacklist

The solution I landed on uses two tokens instead of one:

- **Access token** — lives for 15 minutes, sent on every request
- **Refresh token** — lives for 7 days, only used to get a new access token

Short-lived access tokens limit the damage if one gets stolen. 
And when the user logs out, the refresh token goes into a Redis 
blacklist — so even if someone has it, they can't use it to get 
new access tokens.

## Implementing it in NestJS

### Generating both tokens on login
```typescript
private generateTokens(user: UserDocument) {
  const payload = { sub: user._id, email: user.email, role: user.role };

  const accessToken = this.jwtService.sign(payload);

  const refreshToken = this.jwtService.sign(payload, {
    secret: this.config.get('JWT_REFRESH_SECRET'),
    expiresIn: '7d',
  });

  return { accessToken, refreshToken };
}
```

Two secrets, two expiries. The refresh token uses a different 
secret so you can validate them independently.

### Logout — with a detail that matters

The obvious approach is to store the refresh token in Redis with 
a fixed 7-day TTL. But I noticed a problem: if the user logs out 
on day 6, you're storing a token that has 1 day left as if it had 
seven. Small thing, but at scale it adds up — you're keeping tokens 
in Redis longer than necessary.

So instead, I calculate the remaining TTL dynamically from the 
token's own expiry:
```typescript
async logout(refreshToken: string) {
  const payload = this.jwtService.decode(refreshToken) as any;
  const ttl = payload.exp - Math.floor(Date.now() / 1000);
  
  if (ttl > 0) {
    await this.cacheManager.set(refreshToken, 'blacklisted', ttl * 1000);
  }
  
  return { message: 'Logged out successfully' };
}
```

The token expires in Redis exactly when it would have expired 
anyway. No cleanup needed, no wasted memory.

### Checking the blacklist on refresh
```typescript
async refresh(refreshToken: string) {
  const isBlacklisted = await this.cacheManager.get(refreshToken);
  if (isBlacklisted) throw new UnauthorizedException('Token revoked');

  try {
    const payload = this.jwtService.verify(refreshToken, {
      secret: this.config.get('JWT_REFRESH_SECRET'),
    });

    const user = await this.userModel.findById(payload.sub);
    if (!user) throw new UnauthorizedException('User not found');

    return this.generateTokens(user);
  } catch {
    throw new UnauthorizedException('Invalid refresh token');
  }
}
```

Before doing anything with the refresh token, we check Redis. 
If it's there — 401, token revoked. Fast, simple, works.

## What we ended up with

The full flow looks like this:
```
POST /auth/register  →  { accessToken (15min), refreshToken (7d) }
POST /auth/login     →  { accessToken (15min), refreshToken (7d) }

Every API request    →  Authorization: Bearer <accessToken>
Access token expires →  POST /auth/refresh  →  new tokens
POST /auth/logout    →  refreshToken added to Redis with dynamic TTL
Blacklisted refresh  →  401 Token revoked
```

Logout actually works now. The token is gone from the client 
and invalid on the server. If someone had grabbed it — tough luck.

The dynamic TTL was a small thing to add but it felt like the 
right call. Redis memory is cheap, but there's no reason to 
store something longer than you need to.
