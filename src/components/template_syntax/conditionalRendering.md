# Conditional Rendering

## ```v-else-if```
The ```v-else-if``` serves as na "else if block". It can also be chained multiple times:

``` 
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

Similar to ```v-else```, a ```v-else-if``` element must immediatly follow a ```v-if``` or a ```v-else-if``` element.

## ```v-if``` on ```<template>```
We can use ```v-if``` on a ```<template>``` element to serves as an invisible wrapper to multiple elements. For example:

``` 
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

```v-else``` and ```v-else-if``` can also be used on ```<template>```.

## v-show
Another option for conditionally displaying an element is the ```v-show``` directive. The usage is largely the same:

``` 
<h1 v-show="ok">Hello!</h1>
```

The difference is that an element with ```v-show``` will always be rendered and remain in the DOM; ```v-show``` only toggles the ```display``` CSS property of the element.

```v-show``` doesn't support the ```<template>``` element, nor does it work with ```v-else```.

## ```v-if``` vs. ```v-show```

```v-if``` is the "real" conditional rendering because it ensures that event listeners and child components inside the conditional block are properly destroyed and re-created during toggles.

```v-if``` is also lazy; if the conditional is false on initial render, it won't do anything - the conditional block won't be rendered unitil the condition becomes true for the first time.

In comparison, ```v-show``` is much simpler - the element is always rendered regardless of initial condition, with CSS-based toggling.

Generally speaking, ```v-if``` has higher toggle cost while ```v-show``` has higher initial render costs. So prefer ```v-show``` if you need to toggle something very often, and prefer ```v-if``` if the condition is unlikely to change at runtime.
