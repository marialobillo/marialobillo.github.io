---
layout: post
title:  "Factory States on Laravel 8"
date:   2021-08-18 20:24:00 +0100
categories: php github laravel states factory 
permalink: /factory-states-laravel
---

I am using factories for testing concerts, the concerts could be published or unpublished, and depend on that users can see them on the list or not, pretty simple. ( I'm following the Adam Wathan about TDD and Laravel.) So I'm going to use the Factory States for published concerts and unpublished, and use those on my tests.

The Laravel documentation about Factory States is [here](https://laravel.com/docs/8.x/database-testing#factory-states) The use is very easy, I did for my published and unpublished concerts [here.](https://github.com/marialobillo/ticketbeast/blob/main/database/factories/ConcertFactory.php) the definition.

<pre>
    /**
     * Define the published at attribute value.
     *
     * @return array
     */
    public function unpublished()
    {
        return $this->state(function (array $attributes) {
            return [
                'published_at' => null,
            ];
        });
    }
</pre>


And for incooporate for the Factory, I really like it because is a very clean way to do it.

<pre>
     $concert = Concert::factory()->unpublished()->create();
</pre>

