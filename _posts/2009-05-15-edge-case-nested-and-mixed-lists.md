---
layout: post
title: "Edge Case: Nested and Mixed Lists"
categories:
  - Edge Case
tags:
  - content
  - css
  - edge case
  - lists
  - markup
---

Nested and mixed lists are an interesting beast. It's a corner case to make sure that

<<<<<<< HEAD
- Lists within lists do not break the ordered list numbering order
- Your list styles go deep enough.
=======
* Lists within lists do not break the ordered list numbering order
* Your list styles go deep enough.
>>>>>>> 076c16e... changed theme

### Ordered -- Unordered -- Ordered

1. ordered item
<<<<<<< HEAD
2. ordered item

- **unordered**
- **unordered**
  1. ordered item
  2. ordered item

=======
2. ordered item 
  * **unordered**
  * **unordered** 
    1. ordered item
    2. ordered item
>>>>>>> 076c16e... changed theme
3. ordered item
4. ordered item

### Ordered -- Unordered -- Unordered

1. ordered item
<<<<<<< HEAD
2. ordered item

- **unordered**
- **unordered**
  - unordered item
  - unordered item

=======
2. ordered item 
  * **unordered**
  * **unordered** 
    * unordered item
    * unordered item
>>>>>>> 076c16e... changed theme
3. ordered item
4. ordered item

### Unordered -- Ordered -- Unordered

<<<<<<< HEAD
- unordered item
- unordered item
  1. ordered
  2. ordered
  - unordered item
  - unordered item
- unordered item
- unordered item

### Unordered -- Unordered -- Ordered

- unordered item
- unordered item
  - unordered
  - unordered
    1. **ordered item**
    2. **ordered item**
- unordered item
- unordered item
=======
* unordered item
* unordered item 
  1. ordered
  2. ordered 
    * unordered item
    * unordered item
* unordered item
* unordered item

### Unordered -- Unordered -- Ordered

* unordered item
* unordered item 
  * unordered
  * unordered 
    1. **ordered item**
    2. **ordered item**
* unordered item
* unordered item
>>>>>>> 076c16e... changed theme
