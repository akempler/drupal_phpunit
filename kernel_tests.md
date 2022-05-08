---
layout: default
title: Kernel Tests
nav_order: 4
---

# Kernel Tests

Since Drupal is not fully bootstrapped for kernel tests, 
you need to specify which schemas are needed for your tests. 

For example, if you are working with nodes, you might need the node data schema:  
```
$this->installSchema('node', ['node_data']);
```
`installSchema()` expects a module name, and the table you need managed by that module.

You could also just require all the schemas for that module if you are unsure with: 
```
$this->installEntitySchema('node');
```
