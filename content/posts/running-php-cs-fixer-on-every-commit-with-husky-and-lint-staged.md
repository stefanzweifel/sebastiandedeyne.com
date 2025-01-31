---
date: 2019-05-02
title: Running PHP CS Fixer on every commit with husky and lint-staged
categories: ["articles"]
tags:
    - tooling
---

Last month, I wrote a [post](https://sebastiandedeyne.com/keeping-your-assets-prettier-on-every-commit) about automatically running prettier before every commit. This ensures that all JavaScript and CSS are formatted correctly before they're stored in the project's repository.

Husky and lint-staged have been working hard keeping our front-end assets clean as [Spatie](https://spatie.be), so we decided to expand their responsibilities to keep our PHP files clean too. This is a modified version of my previous post using PHP CS Fixer instead of prettier. There's also an example of a conmbined configuration to run prettier and PHP CS Fixer simultaneously at the end of the post.

<!--more-->

As an aside, this is what our `.php_cs` file generally looks like:

```php
<?php

$finder = PhpCsFixer\Finder::create()
    ->in(__DIR__)
    ->exclude(['bootstrap', 'storage', 'vendor'])
    ->name('*.php')
    ->name('_ide_helper')
    ->notName('*.blade.php')
    ->ignoreDotFiles(true)
    ->ignoreVCS(true);

return PhpCsFixer\Config::create()
    ->setRules([
        '@PSR2' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sortAlgorithm' => 'alpha'],
        'no_unused_imports' => true,
    ])
    ->setFinder($finder);
```

Before we get started, we need to ensure our dependencies are installed:

```
composer require friendsofphp/php-cs-fixer

npm install husky lint-staged --save-dev
```

## Running prettier with lint-staged

Lint-staged is an npm package that will run a script on your staged files, in other words, on the files you want to commit. You can filter the staged files you want to run the scripts on with a glob.

Create a `lint-staged.config.js` file in your project root.

```js
module.exports = {
    '**/*.php': ['php ./vendor/bin/php-cs-fixer fix --config .php_cs --allow-risky=yes'],
};
```

This will target all PHP files in your project. Note that the `fix` command will be run once per staged `.php` file, and will only apply to that file, which makes this process a _lot_ faster than running `php ./vendor/bin/php-cs-fixer fix` over the entire project.

## Git hooks with Husky

Husky is pretty straightforward. It registers some git hooks for you after `npm install`. The hook we care about is pre-commit: we want to format our assets before they get committed to the repository.

We'll add the husky configuration to our `package.json` file.

```json
{
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged"
        }
    }
}
```

This will run the `lint-staged` command on pre-commit.

## Combined with prettier

If you're also using prettier, here's what the combined `linters` configuration looks like:

```js
module.exports = {
    'resources/**/*.{css,js}': ['prettier --write'],
    '**/*.php': ['php ./vendor/bin/php-cs-fixer fix --config .php_cs --allow-risky=yes'],
};
```

It may sound odd to use an npm package to ensure PHP files stay conform to our coding standards, especially while tools like [GrumPHP](https://github.com/phpro/grumphp) already exist in the PHP ecosystem. I prefer the husky + lint-staged setup because they're not as opinionated towards a single ecosystem, and are useable throughout multiple contexts in a single project.
