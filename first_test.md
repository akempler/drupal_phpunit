---
layout: default
title: A first test
nav_order: 3
---

# Creating a first test

*A simplified scenario for a test:*

Imagine you've created a custom entity type called "Topic" and you want to make sure that the topic add/edit form can be reached.

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
    'text',
    'block',
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
  }
```

For a more detailed example, and slightly refactored version of this, see the Functional Tests page.