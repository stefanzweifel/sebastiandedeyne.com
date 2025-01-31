---
date: 2017-03-27
title: A package for snapshot testing in PHPUnit
subtitle: Works with PHPUnit 5.7+
categories: ["articles"]
tags:
  - PHP
  - PHPUnit
  - testing
---

The gist of snapshot testing is asserting that a set of data hasn't changed compared to a previous version, which is a _snapshot_ of the data, to prevent regressions. The difference between a classic and an is that you don't write the expectation yourself when snapshot testing.

When a snapshot assertion happens for the first time, it creates a snapshot file with the actual output, and marks the test as incomplete. Every subsequent run will compare the output with the existing snapshot file to check for regressions.

Snapshot testing is most useful larger datasets that can change over time, like serializing an object for an XML export or a JSON API endpoint.

<!--more-->

Our package, which exposes a trait to add snapshot testing capabilities to your tests, can be installed via [composer](https://packagist.org/packages/spatie/phpunit-snapshot-assertions) and is available on [GitHub](https://github.com/spatie/phpunit-snapshot-assertions).

_I couldn't find any formal origin of snapshot testing. The oldest library I found was [one written by Facebook](https://github.com/facebook/ios-snapshot-test-case) to snapshot test iOS user interfaces. [Jest](https://facebook.github.io/jest/) — a JavaScript testing framework which is also made by Facebook — recently popularised snapshot testing, since it provides and excellent workflow for testing user interfaces built with virtual dom libraries like React._

### Basic Example

Let's do a snapshot assertion for a simple string, "foo".

```php
<?php

public function test_it_is_foo() {
    $this->assertMatchesSnapshot('foo');
}
```

The first time the assertion runs, it doesn't have a snapshot to compare the string with. The test runner generates a new snapshot and marks the test as incomplete.

```
> ./vendor/bin/phpunit

There was 1 incomplete test:

1) ExampleTest::test_it_matches_a_string
Snapshot created for ExampleTest__test_it_matches_a_string__1

OK, but incomplete, skipped, or risky tests!
Tests: 1, Assertions: 0, Incomplete: 1.
```

Snapshot ids are generated based on the test and testcase's names. Basic snapshots return a `var_export` of the actual value.

```
<?php return 'foo';
```

Let's rerun the test. The test runner will see that there's already a snapshot for the assertion and do a comparison.

```
> ./vendor/bin/phpunit

OK (1 test, 1 assertion)
```

If we change actual value to "bar", the test will fail because the snapshot still returns "foo".

```php
<?php

public function test_it_is_foo() {
    $this->assertMatchesSnapshot('bar');
}
```

```
> ./vendor/bin/phpunit

1) ExampleTest::test_it_matches_a_string
Failed asserting that two strings are equal.
--- Expected
+++ Actual
@@ @@
-'foo'
+'bar'

FAILURES!
Tests: 1, Assertions: 1, Failures: 1.
```

When we expect a changed value, we need to tell the test runner to update the existing snapshots instead of failing the test. This is possible by adding a `-d --update-snapshots` flag to the `phpunit` command.

```
> ./vendor/bin/phpunit -d --update-snapshots

1) ExampleTest::test_it_matches_a_string
Snapshot updated for ExampleTest__test_it_matches_a_string__1

OK, but incomplete, skipped, or risky tests!
Tests: 1, Assertions: 1, Incomplete: 1
```

As a result, our snapshot file contains "bar" instead of "foo".

```
<?php return 'bar';
```

## Methods

Assertions are done using the `assertMatchesSnapshot` method.

```php
<?php

public function it_matches_something()
{
    $something = new Something();

    $this->assertMatchesSnapshot($something);
}
```

If you're working with JSON or XML data, you're better off using a dedicated `assertMatchesJsonSnapshot` or `assertMatchesXmlSnapshot` method, which will save snapshots as `.json` or `.xml` files, and provide a better diff when the snapshot doesn't match.

```php
<?php

public function it_matches_something_json()
{
    $something = new Something();

    $this->assertMatchesJsonSnapshot($something->toJson());
}
```

## Snapshot files

Be default, snapshots are stored in a `__snapshots__` directory at the same level of the test
class.

```
__snapshots__/
    ExampleTest__test_it_matches_a_string.php
ExampleTest.php
```

Snapshot ids and the snapshot directory's name can be changed in by overriding `getSnapshotId` and `getSnapshotDirectory`. Take a look at [the readme](https://github.com/spatie/phpunit-snapshot-assertions#customizing-snapshot-ids-and-directories) for a more detailed explanation.

## Drivers

Drivers make the package extendable, without the `Driver` interface snapshot assertions would be limited to JSON, XML and generic values with `var_export`. A driver handles serializing and matching snapshot data. For example, if your application would make extensive use of YAML files, you could write a `YamlDriver` to save snapshots as real YAML files and improve PHPUnit's diff output.

Custom drivers can be applied by passing them to `assertMatchesSnapshot`.

```php
<?php

public function it_matches_yaml()
{
    $order = new Order();

    $this->assertMatchesSnapshot(
        $order->toYaml(),
        new YamlDriver()
    );
}
```

If you're interested in a detailed explanation on writing custom drivers, they have [a dedicated section](https://github.com/spatie/phpunit-snapshot-assertions#writing-custom-drivers) in the readme.

## Road To v1.0

<strike>There are a some tidbits that require polishing before releasing a first major version.</strike> We've decided to tag v1.0 already since we're using this package without issues in a few projects already. The missing features can be added in a a later release.

### Cleaning Up Unused Snapshots ([#17](https://github.com/spatie/phpunit-snapshot-assertions/issues/17))

At the moment, there's no way to determine which snapshots aren't used and can be deleted. Old snapshots need to be deleted manually, a "cleanup" task would be welcome to automate this.

### Hack-free Update Flag ([#22](https://github.com/spatie/phpunit-snapshot-assertions/issues/22))

The `--update-snapshots` flag needs to be specified after `-d`, which is meant to set custom php.ini values. PHPUnit doesn't support custom CLI options, but it might be added in a future release ([sebastianbergmann/phpunit#2271](https://github.com/sebastianbergmann/phpunit/issues/2271))

Despite not having a stable version number, there most likely won't be any large breaking changes anymore heading to v1.0.

---

_Thanks [@AlexVanderbist](https://twitter.com/alexvanderbist) for helping out with the integration tests for this package!_

- [spatie/phpunit-snapshot-assertions on GitHub](https://github.com/spatie/phpunit-snapshot-assertions)
- [spatie/phpunit-snapshot-assertions on Packagist](https://packagist.org/packages/spatie/phpunit-snapshot-assertions)
- An overview of our open source packages can be found [on our website](https://spatie.be/en/opensource)
