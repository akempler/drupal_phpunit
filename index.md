---
layout: default
title: Intro
nav_order: 1
---

# Drupal Unit Testing

An introduction and set of resources for unit testing in Drupal.

## PHPUnit
Drupal leverages the opensource [PHPUnit testing framework](https://phpunit.de/).

Drupal has three main kinds of tests that you can create:
### Unit
Use these to test individual functions that are fairly self contained. 
- Unit tests do NOT load the database.
- Unit tests do NOT load a full Drupal site. 

If the functionality you are testing interacts with Drupal in a meaningful way such as 
leveraging Drupal's classes or other functionality, you will likely be better off with a Kernel or Functional test.
### Kernel
Kernel tests offer a middlegound between Unit and Functional tests. <br/>
They bootstrap the kernel and a minimal number of extensions.
### Functional
Functional tests use a fully bootstrapped version of Drupal. 
They can simulate stepping through a website the same way a user would. 
As such they can accomplish a lot, but are also the slowest to run. 
A single test can take a few minutes to run, depending on the complexity and number of steps in the test.

## Test data

Unit tests in Drupal do **NOT** use your project’s data. 
All data is created/defined specifically for the tests. 

A new database is created for each test. 
This helps make sure your tests are repeatable as they don't rely on changing data.

You have a few options for where the test data/tables are created.  
* You can setup phpunit to use your existing drupal database to store the test data
* You can define a separate database for tests.
* You can use the built-in sqlite.

## Installing PHPUnit

```
composer require --dev phpunit/phpunit
composer require --dev phpspec/prophecy-phpunit:^2
composer require drupal/entity_browser --dev
```
Test your installation with: 
```
vendor/bin/phpunit --version
```

## Running Tests

Drupal uses 2 different test runners: *run-tests.sh* and *phpunit*

Run all tests in a group: 
```
phpunit --group topics
```

Run all tests in a module:
```
phpunit web/modules/custom/topics
```

Run a specific test:
```
phpunit web/modules/custom/topics/tests/src/Functional/TopicTest.php
```

Full list of phpunit runner command line options:
https://phpunit.readthedocs.io/en/9.5/textui.html


### Lando Integration



----
This is a work in progress. Feel free to create a pull request to add or edit any of the information.