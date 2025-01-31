---
title: "Handling authorization in a Laravel and Inertia application"
date: 2019-11-05
categories: ["articles"]
tags:
    - Laravel
    - Inertia.js
---

I last blogged about [handling routes in a Laravel and Inertia app](https://sebastiandedeyne.com/handling-routes-in-a-laravel-inertia-application/). The premise was that we don't have access to Laravel's URL generator functions with Inertia, so we need to pass our application's routes down differently.

The same problem exists with authorization: we don't have access to the `can` helper and other `Gate` methods. Here's a short post about dealing with authorization on the frontend.

<!--more-->

I've split this up in two sections: authorization that refers to a resource type, and authorization that refers to a resource entity (the difference will become clear). The examples use Vue.js, but the principles apply to any other frontend framework.

I feel obliged to mention that this is about using authorization to determine whether or not something should be displayed. This is not a replacement for proper authorization checks in controllers, as everything on the frontend can be tampered with.

## Authenticating resource types

Say you're building an blog application with a posts module. Resource type related authentication would answer questions like "can the authenticated user view the posts module" or "can the authenticated user create a new post". They're not tied to a specific post entity.

First ensure that you have access to a user object. Use Inertia's share method to `share` the authenticated user with your frontend.

```php
<?php

Inertia::share('auth.user', function () {
    return Auth::user();
});
```

The user object is now accessible through Inertia's page variable.

```html
<template>
  <p>Hello, {{ $page.auth.user.name }}</p>
</template>
```

Next we'll add a permissions key to our user object. The permissions key will hold an associative array containing a user's general permissions.

{{< highlight php "hl_lines=20-22 24-32" >}}
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    protected $hidden = [
        'password', 'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
    ];

    protected $appends = [
        'permissions',
    ];

    public function getPermissionsAttribute()
    {
        return [
            'posts' => [
                'view' => $this->can('posts.view'),
                'create' => $this->can('posts.create'),
            ],
        ];
    }
}
{{</ highlight >}}

In this example we've added the permission data on the model itself, but transforming data with Eloquent resources before passing it to the frontend might be a good idea for larger apps.

We can now easily check for permissions on the frontend.

```html
<template>
  <a
    v-if="$page.auth.user.permissions.posts.create"
    href="/posts/create"
  >
    Create post
  </a>
</template>
```

This strategy is sufficient for apps that have simple authorization requirements. If you need more fine-grained control, read on.

## Authenticating resource entities

Resource entity related authorization ensires that the authenticated user can access or modify an entity, for example edit a specific `Post`.

Instead of adding the permissions attribute to the `User` model, we'll add it to the `Post` model. These permissions are about the specific `Post` instance, not posts in general.

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    protected $appends = [
        'permissions',
    ];

    public function getPermissionsAttribute()
    {
        return [
            'update' => $this->can('posts.update', $this),
            'delete' => $this->can('posts.delete', $this),
        ];
    }
}
```

To do a permission check, read out the permissions attribute in the posts object.

```html
<template>
  <table>
    <tr v-for="post in posts" :key="post.id">
      <td>{{ post.title }}</td>
      <td>
        <a v-if="post.permissions.update" href="…">
          Edit
        </a>
        <a v-if="post.permissions.delete" href="…">
          Delete
        </a>
      </td>
    </tr>
  </table>
</template>

<script>
export default {
  props: ['posts'],
};
</script>
```

These authorization approaches turned out to be very similar to URL generation, it's all about transforming data before it hits your view layer.
