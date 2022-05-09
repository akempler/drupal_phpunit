---
layout: default
title: A First Test
nav_order: 3
---

# Creating a first test

*A simplified scenario for a test:*

Imagine you've created a custom entity type called "Topic" and you want to make sure that the administrative topic type list page can be reached.

First create the necessary directories. Each type of unit test gets its own directories. You should have a structure similar to this, assuming a custom module named "topics".

`/myproject/web/modules/custom/topics/tests/src/Functional`

Now create your test file in the Functional directory, for example: `TopicTypeTest.php`

```php
<?php

namespace Drupal\Tests\topics\Functional;

use Drupal\topics\Entity\TopicType;

/**
 * Tests the topic type UI.
 *
 * @group topics
 */
class TopicTypeTest extends BrowserTestBase {

  /**
   * Modules to enable.
   *
   * @var array
   */
  protected static $modules = [
    'topics',
  ];

  /**
   * {@inheritdoc}
   */
  protected $defaultTheme = 'stark';

  /**
   * {@inheritdoc}
   */
  protected function setUp(): void {
    parent::setUp();

    $adminUser = $this->drupalCreateUser([
      'administer topic types',
      'administer topics',
      'access topic overview',
      'create topic',
      'edit topic',
      'delete topic',
      'view topic',
    ], NULL, TRUE);

    $this->drupalLogin($adminUser);
  }

  /**
   * Tests topic type list page.
   */
  public function testTopicTypeList() {
    $this->drupalGet('/admin/structure/topic_types');
    $this->assertSession()->statusCodeEquals(200);

    $this->assertSession()->pageTextContains('No topic types available.');
  }
```

## Walking through the code

### @group
```php
/**
 * Tests the topic type UI.
 *
 * @group topics
 */
class TopicTypeTest extends BrowserTestBase {
```

The @group annotation allows you to group related tests. 
You can the run all tests in a specified group.

### $modules
```php
/**
   * Modules to enable.
   *
   * @var array
   */
  protected static $modules = [
    'topics',
  ];
```
These are the modules that need to be enabled in order to test your code. 

### $defaultTheme
```php
  /**
   * {@inheritdoc}
   */
  protected $defaultTheme = 'stark';
```
You must specify the default theme that will be used for the browser based test. 
If you were relying on some core markup for some functionality, you might want to use another theme.

### setup()
```php
/**
   * {@inheritdoc}
   */
  protected function setUp(): void {
    parent::setUp();

    $adminUser = $this->drupalCreateUser([
      'administer topic types',
      'administer topics',
      'access topic overview',
      'create topic',
      'edit topic',
      'delete topic',
      'view topic',
    ], NULL, TRUE);

    $this->drupalLogin($adminUser);
  }
```
Every test function will build a completely new Drupal instance. So for example, if you add a function to test the topic type add/edit form, when that is run, it will get a new Drupal instance.

For tasks that you want run on each of the test functions, you can use `setup()`.

In this case for each thing we'll want to test, we create and login an admin user.

### testTopicTypeList()
```php
/**
   * Tests topic type list page.
   */
  public function testTopicTypeList() {
    $this->drupalGet('/admin/structure/topic_types');
    $this->assertSession()->statusCodeEquals(200);

    $this->assertSession()->pageTextContains('No topic types available.');
  }
```
This is our test function. It uses drupalGet() to request the path for our list of topic types, and then asserts that the page returns a 200 response code. If the page was not working, and returned anything other than a 200, the assertion would fail, and the test would be marked as failed.

We also check that the expected default text for the page when no topic types have been created yet is displayed. If this text is not displayed, it may be a sign that something is wrong with the page.

## Running the test
You can run the test with phpunit:  

```php
// Run all tests in the topics group
phpunit --group topics
```

```php
// Run all tests in the topics module
phpunit web/modules/custom/topics
```

```php
// Run just this test
phpunit web/modules/custom/topics/tests/src/Functional/TopicTypeTest.php
```

Full list of phpunit runner command line options:
https://phpunit.readthedocs.io/en/9.5/textui.html

---

For a more detailed example, and slightly refactored version of this, see the Functional Tests page.