---
layout: default
title: Functional Tests
nav_order: 4
---

# Functional Tests

## BrowserTestBase

The following examples use the BrowserTestBase as the base class. 
If you need to test javascript/ajax functionality you can extend the `Drupal\FunctionalJavascriptTests\WebDriverTestBase` base class. 
Tests extending this should go in a `/tests/src/FunctionalJavascript/` directory.

## Assertions

Assertions are the outcomes you are looking for. These could be simply that a page returns a 200 or 403 response. You can also assert that specific text appears on a page, a link exists, etc.

The full list of web assertions you can make can be reviewed here: `web/core/tests/Drupal/Tests/WebAssert.php`

## Create a base test class

If you find yourself repeating code in a variety of tests, create a base test class that your other tests extend.

For example, refactoring the Topic Type test from the "Creating a First Test" page we can move the setup to a base class.

We might create something like: `/myproject/web/modules/custom/topics/tests/src/Functional/TopicTestBase.php`

```php
<?php

namespace Drupal\Tests\topics\Functional;

use Drupal\topics\TopicTestTrait;
use Drupal\Tests\BrowserTestBase;

/**
 * Tests topic access handling.
 */
abstract class TopicTestBase extends BrowserTestBase {

  use TopicTestTrait;

  /**
   * Modules to enable.
   *
   * @var array
   */
  protected static $modules = [
    'topics',
    'text',
    'node',
  ];

  /**
   * Testing topic type entity.
   *
   * @var \Drupal\topics\Entity\TopicType
   */
  protected $type;


  /**
   * Testing topic entity.
   *
   * @var \Drupal\topics\Entity\Topic
   */
  protected $topic;

  /**
   * Testing admin user.
   *
   * @var \Drupal\user\Entity\User
   */
  protected $adminUser;

  /**
   * {@inheritdoc}
   */
  protected $defaultTheme = 'stark';

  /**
   * {@inheritdoc}
   */
  protected function setUp(): void {
    parent::setUp();

    $this->adminUser = $this->drupalCreateUser([
      'administer topic types',
      'administer topics',
      'access topic overview',
      'create topic',
      'edit topic',
      'delete topic',
      'view topic',
    ], NULL, TRUE);
  }

}
```

Now we can just extend the base class:

```php
<?php

namespace Drupal\Tests\topics\Functional;

use Drupal\topics\Entity\TopicType;

/**
 * Tests the topic type UI.
 *
 * @group topics
 */
class TopicTypeTest extends TopicTestBase {

  /**
   * {@inheritdoc}
   */
  protected function setUp(): void {
    parent::setUp();
    $this->drupalLogin($this->adminUser);
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

## Test Traits

Another common pattern is to create traits with functionality you might want to use across a variety of tests.
For example, if we want to test that our Topic Type list displays topic types, then we need to create some. 
We'll also need to be able to create them to test various other features like the topic type form, deleting a topic type, etc.

So we might create a trait that encapsulates the functionality of creating Topics and Topic Types. Then any test class that needs that capability, can just use that trait. 

For example, in the code above, the TopicTestTrait was included with in the base test class with:  
```php
use TopicTestTrait;
```

And the trait might contain some code like this:  
```php
<?php

namespace Drupal\topics;

use Drupal\topics\Entity\TopicType;
use Drupal\topics\Entity\Topic;

/**
 * Provides methods to create topic and topic_types.
 *
 * This trait is meant to be used only by test classes.
 */
trait TopicTestTrait {

  /**
   * Creates a topic type for tests.
   *
   * @param string $id
   *   The topic type machine name.
   * @param string $label
   *   The name of the topic type.
   * @param string $description
   *   The description of the topic type.
   *
   * @return \Drupal\topics\Entity\TopicType
   *   Returns a profile type entity.
   */
  protected function createTopicType($id = NULL, $label = NULL, $description = NULL) {
    $id = !empty($id) ? $id : $this->randomMachineName();
    $label = !empty($label) ? $label : $this->randomMachineName();

    $type = TopicType::create([
      'id' => $id,
      'label' => $label,
      '$description ' => $label,
    ]);
    $type->save();

    return $type;
  }

  /**
   * Create a topic.
   *
   * @param \Drupal\topics\Entity\TopicType $topic_type
   *   A topic type for the created topic entity.
   * @param string $id
   *   An optional id to manually set.
   * @param string $label
   *   An optional label to set manually for the topic.
   *
   * @return \Drupal\topics\TopicInterface
   *   A topic.
   */
  protected function createTopic(TopicType $topic_type, $id = NULL, $label = NULL) {

    $id = !empty($id) ? $id : $this->randomMachineName();
    $label = !empty($label) ? $label : $this->randomMachineName();

    $topic = Topic::create([
      'bundle' => $topic_type->id(),
      'uid' => 1,
      'title' => $label,
    ]);
    $topic->save();

    return $topic;
  }

}
```

Now we can extend the TopicTypeList() test to test not just the empty state, but also that new topic types are displayed as well: 
```php
 /**
  * Tests topic type list page.
  */
  public function testTopicTypeList() {
    $this->drupalGet('/admin/structure/topic_types');
    $this->assertSession()->statusCodeEquals(200);
    $this->assertSession()->pageTextContains('No topic types available.');

    $this->type = $this->createTopicType('test_topic_type', 'Test Topic Type', 'This is a test topic type');
    $this->drupalGet('/admin/structure/topic_types');
    $this->assertSession()->pageTextContains('Test Topic Type');
  }

```

### Some additional tests to make on the topic types:

Here we make sure we can delete a topic type without error.

```php
  /**
   * Tests the topic type edit page.
   */
  public function testTopicTypeDelete() {
    $this->topic_type = $this->createTopicType('test_topic_type', 'Test Topic Type', 'This is a test topic type');
    $this->drupalGet($this->topic_type->toUrl('delete-form'));

    $this->submitForm([], 'Delete');
    $topic_type_exists = (bool) TopicType::load($this->topic_type->id());
    $this->assertEmpty($topic_type_exists);

    $this->assertSession()->pageTextContains('The topic type Test Topic Type has been deleted.');
  }
  ```

Some things being done in this test:

Instead of hard coding the route path for the drupalGet function, we get it from the topic_type entity by referencing the entity link 'delete-form':  
```php
$this->drupalGet($this->topic_type->toUrl('delete-form'));
```
We then submit the delete form using `submitForm()`.
The first parameter, where we are passing an empty array, allows you to set an array of values to be submitted. These would be the form fields for example. The second parameter is the button we want to emulate clicking. In this case 'Delete'.

We then assert that the topic type with that id no longer exists.

We also assert that the deleted message is displayed.

Some improvements to the above would be to use variables for the topic type names.

Another piece of functionality in the Topics system is that you can enable different entity types to be added to a topic. Here's an example of a test for that page:
```php
/**
 * Tests the topic type - assignable resources page.
 */
public function testTopicTypeAssignableResourcesPage() {
    $this->drupalCreateContentType(array('type' => 'article', 'name' => 'Article'));

    $this->type = $this->createTopicType('test_topic_type', 'Test Topic Type', 'This is a test topic type');
    $this->drupalGet('/admin/structure/topic_types/test_topic_type/topic_content_types');
    $this->assertSession()->statusCodeEquals(200);
    $this->assertSession()->pageTextContains('Assignable Resource Types');
    $this->assertSession()->pageTextContains('Article');
    $this->assertSession()->linkExists('Enable');

    $this->clickLink('Enable');
    $this->assertSession()->statusCodeEquals(200);
    $this->assertSession()->pageTextContains('Enable topic resource');

    $page = $this->getSession()->getPage();
    $page->fillField('max_entities', '6');
    $page->pressButton('Save');

    $this->drupalGet('/admin/structure/topic_types/test_topic_type/topic_content_types');
    $this->assertSession()->pageTextContains('Assignable Resource Types');
    $this->assertSession()->linkExists('Disable');
}
```
First we create an 'article' content type (node type) so that we can enable it for a topic type. Remember, no data exists except for what we create, so default content types like article wouldn't exist out of the box.

If an entity type hasn't been enabled for a topic type yet, it should have an 'enable' link. If it is enabled, it should have a 'disabled' link. So we check for the 'Enable' link with: 
```php
$this->assertSession()->linkExists('Enable');
```
We then click it with: 
```php
$this->clickLink('Enable');
```
Clicking that link should take us to the 'Enable topic resource' form. We confirm are redirected there. 

We then fill out the form fields and press the "Save" button. 
```php
// Get the Document Element so we can access specific fields.
$page = $this->getSession()->getPage();
// Fill the max_entities form field with the value '6'.
$page->fillField('max_entities', '6');
// Click the Save button.
$page->pressButton('Save');
```

We should now be redirected back to the assignable resources page, and the article entity type should have a 'disable' link since we just enabled it.
