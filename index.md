An introduction and set of resources to unit testing in Drupal.

## PHPUnit
Drupal leverages the opensource [PHPUnit testing framework](https://phpunit.de/).

Drupal has three kinds of tests that you can create:
### Unit
Use these to test individual functions that are fairly self contained. 
- Unit tests do NOT load the database.
- Unit tests do NOT load a full Drupal site.
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

## Installing PHPUnit and Running Tests

### Lando Integration