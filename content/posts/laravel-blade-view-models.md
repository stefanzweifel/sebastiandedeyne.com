---
title: "Laravel Blade & View Models"
slug: laravel-blade-view-models
date: 2022-01-10
categories: ["articles"]
tags:
  - Laravel
  - Blade
summary: An overview on view models in Laravel
---

A view model represents data for a specific view or page. In its simplest form, a view model is a plain PHP object with a bunch of (typed) properties.

```php
class ProfileViewModel
{
    public function __construct(
        public User $user,
        public array $companies,
        public string $action,
    ) {}
}
```

View models make the dependencies of a view explicit. With a view model, you don't need to dig into the view's markup to find out which data it requires.

To use a view model, instantiate it in your controller action, and pass it to the view. I've built the habit to pass view models as `$view` variables to keep them consistent across all templates.

```php
class ProfileController
{
    public function edit()
    {
        $viewModel = new ProfileViewModel(
            user: Auth::user(),
            companies: Companies::all(),
            action: action([ProfileController::class, 'update']),
        );

        return view('profile.edit', ['view' => $viewModel]);
    }
}
```

```html
<form action="{{ $view->action }}" method="POST">
    <input name="name" value="{{ old('name', $view->user->name) }}">
    <select name="company_id">
        @foreach($view->companies as $company)
            …
        @endforeach
    </select>
</form>
```

## IDE superpowers

And now for the fun part: type hint the variable in a `@php` block at the start of your Blade view, and start writing HTML.

```html
@php
    /** @var $view App\Http\ViewModels\ProfileViewModel */
@endphp

<form action="{{ $view->action }}" method="POST">
    <input name="name" value="{{ old('name', $view->user->name) }}">
    <select name="company_id">
        @foreach($view->companies as $company)
            …
        @endforeach
    </select>
</form>
```

This is where view models shine. An IDE will recognize the `@var` declaration and provide autocompletion for the view data. In addition, if you rename one of the view's properties using an IDE's refactoring capabilities, it'll also rename the usages in the views.

Refactor rename `ProfileViewModel` in PhpStorm:

```diff
  class ProfileViewModel
  {
      public function __construct(
          public User $user,
-         public array $companies,
+         public array $organizations,
          public string $action,
      ) {}
  }
```

`ProfileController` and `profile/edit.blade.php`  will be automatically updated:

```diff
  class ProfileController
  {
      public function edit()
      {
          $viewModel = new ProfileViewModel(
              user: Auth::user(),
-             companies: Companies::all(),
+             organizations: Companies::all(),
              action: action([ProfileController::class, 'update']),
          );

          return view('profile.edit', ['view' => $viewModel]);
      }
  }
```

```diff
    <select name="company_id">
-       @foreach($view->companies as $company)
+       @foreach($view->organizations as $company)
            …
        @endforeach
    </select>
```

## Computed Data

View models are also a great place to compute data before sending it to a view. This keeps complex statements outside of controllers and views.

```php
class ProfileViewModel
{
    public bool $isSpatieMember;

    public function __construct(
        public User $user,
        public array $companies,
        public array $organizations,
        public string $action,
    ) {
        $this->isSpatieMember =
            $user->organization->name === 'Spatie';
    }
}
```

You could implement this as `$user->isSpatieMember()`, but I prefer view models for one-off things.

## Ergonomics

You can standardize the `$view` variable by creating a base `ViewModel` class that implements `Arrayable`.

```php
use Illuminate\Support\Arrayable;

abstract class ViewModel implements Arrayable
{
    public function toArray()
    {
        return ['view' => $this];
    }
}
```

```diff
- class ProfileViewModel
+ class ProfileViewModel extends ViewModel
  {
      // …
  }
```

```diff
  class ProfileController
  {
      public function edit()
      {
          // …

-         return view('profile.edit', ['view' => $viewModel]);
+         return view('profile.edit', $viewModel);
      }
  }
```

## Granularity

Alternatively, you could be more granular and specify the exact data you want to display instead of passing models.

This approach requires more boilerplate. However, it makes the view's dependencies more explicit and reusable. For example, this `ProfileViewModel` could be reused for other models than `User`.

```php
class ProfileViewModel extends ViewModel
{
    public function __construct(
        public string $name,
        public string $email,
        public int $jobId,
        public array $jobs,
        public string $storeUrl,
    ) {}
}
```
