### Post #1 — borrador listo (en inglés, puedes publicar tal cual)

**Title:** Shipping a compact Catalog Service: TDD, pragmatic TS, and AWS App Runner

**Context**

I’m building a small “reference” service to show how I work as a senior full-stack engineer: clean layering, tests-first, pragmatic TypeScript, and a simple deploy path.

**What I tried**

I sketched a thin domain (`Product` rules), an application layer with ports, and infrastructure via NestJS + Postgres. I started from tests: first unit tests for domain rules, then e2e against a real DB.

**What worked**

- Keeping domain pure made unit tests fast and focused.
- Using Zod at the boundary gave me runtime validation and compile-time types via `z.infer`.
- A tiny `Result<T,E>` type clarified error handling without throwing everywhere.

**Snippet (domain + DTO)**

```tsx
// domain/product.ts
export type Brand<T,B> = T & { __brand: B };
export type ProductId = Brand<string, "ProductId">;

export interface Product {
  id: ProductId;
  name: string;
  price: number; // >= 0
  stock: number; // int, >= 0
}

export function createProduct(dto: ProductDTO): Result<Product,"ValidationError"> {
  if (dto.price < 0 || dto.stock < 0) return err("ValidationError");
  return ok({ ...dto, id: dto.id as ProductId });
}

// infra/http/dto.ts
const ProductSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  price: z.number().nonnegative(),
  stock: z.number().int().nonnegative(),
});
export type ProductDTO = z.infer<typeof ProductSchema>;

```

**Takeaways**

- “Small and solid” beats “big and unfinished”.
- Zod + `z.infer` keeps runtime and compile-time aligned.
- Tests first (unit → e2e) make refactors safer and faster.

**What’s next**

Wire e2e tests with Testcontainers/Postgres, add JSON logs with `requestId`, expose `/metrics` (p50/p95), and ship a first deploy to AWS App Runner.