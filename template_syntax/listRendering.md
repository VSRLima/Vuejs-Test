# List Rendering

## v-for
The ```v-for``` directive requires a special syntax in the form of ```item of items```, where ```items``` is the source array and ```item```is an alias for the array element being iterated on:

``` 
//js
const items = ref([{ message: 'Foo' }, { message: 'Bar' }])

//template
<li v-for="item in items">
  {{ item.message }}
</li>
```

```v-for``` also supports an optional second alias for the index of current item:

``` 
//js
const parentMessage = ref('Parent')
const items = ref([{ message: 'Foo' }, { message: 'Bar' }])

//template
<li v-for="(item, index) in items">
  {{ parentMessage }} - {{ index }} - {{ item.message }}
</li>
```

Notice that ```v-for``` matches the function signature of the ```forEach``` callback. In fact, you can use destructuring on the ```v-for``` item alias similar to destructuring function arguments:

```
<li v-for="{ message } in items">
  {{ message }}
</li>

<!-- with index alias -->
<li v-for="({ message }, index) in items">
  {{ message }} {{ index }}
</li> 
```

For nested ```v-for```. scoping also works similar to nested functions. Each ```v-for``` scope has access to parent scope:

``` 
<li v-for="item in items">
  <span v-for="childItem in item.children">
    {{ item.message }} {{ childItem }}
  </span>
</li>
```

You can also use ```of``` as the delimeter instead on ```in```, so that it's closer to JavaScript's syntax for iterators:

```<div v-for="item of items"></div>```

## ```v-for``` with an Object
You can also iterate objects using ```v-for```. The iteration order will be based on the result of calling ```Object.keys()``` on the object:

``` 
//js
const myObject = reactive({
  title: 'How to do lists in Vue',
  author: 'Jane Doe',
  publishedAt: '2016-04-10'
})

//template
<ul>
  <li v-for="value in myObject">
    {{ value }}
  </li>
</ul>
```

You can also provide a second alias for the property's name (a.k.a key):

``` 
<li v-for="(value, key) in myObject">
  {{ key }}: {{ value }}
</li>
```

And another for the index:

``` 
<li v-for="(value, key, index) in myObject">
  {{ index }}. {{ key }}: {{ value }}
</li>
```

## ```v-for``` on ```<template>```
Similar to template ```v-if```, you can also use a ```<template>``` tag with ```v-for``` to render a block of multiple elements. For example:

``` 
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

## ```v-for``` with ```v-if```

#### Note
It's not recommended to use ```v-if``` and ```v-for``` on the same element due to implicit precendence.

When they exist on the same node, ```v-if``` has a higher priority than ```v-for```. That means the ```v-if``` condition will not have access to variables from the scope of the ```v-for```:

``` 
<!--
This will throw an error because property "todo"
is not defined on instance.
-->
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo.name }}
</li>
```

This can be fixed by moving ```v-for``` to a wrappign ```<template>``` tag (which is also more explict):

``` 
<template v-for="todo in todos">
  <li v-if="!todo.isComplete">
    {{ todo.name }}
  </li>
</template>
```

## Maintaing State with ```key```
When Vue is updationg a list of elements rendered with ```v-for```, by default it uses an "in-place patch" strategy. If the order of the data items has changed, instead of moving the DOM elements to match the order of the items, Vue will patch each element in-place and make sure it reflects what should be rendered at the particular index.

This default mode is efficient, but only suitable when your list render output doesn't rely on child component state of temporary DOM state (e.g form input values).

To give Vue a hint so that can track each node's identity, and thus reuse and reorder existing elements, you need to provide a unique ```key``` attribute for each item:

``` 
<div v-for="item in items" :key="item.id">
  <!-- content -->
</div>
```

When using ```<template v-for>```, the ```key``` should be placed on the ```<template>``` container:

```
<template v-for="todo in todos" :key="todo.name">
  <li>{{ todo.name }}</li>
</template>
```

#### Note
```key``` here isa special attribute being bound with ```v-bind```. It should not be confused with the property key variable when using ```v-for``` with an object.

The ```key``` binding expects primitive values - i.e. string and numbers.  Do not use objects as ```v-for``` keys.

## ```v-for``` with a Component
You can directly use ```v-for``` on a component, like any normal element (don't forget to provide a ```key```):

``` 
<MyComponent v-for="item in items" :key="item.id" />
```

However, this won't automatically pass any data to the component, because components have isolated scopes of their own. In order to pass the iterated data into the component, we should also use props:

``` 
<MyComponent
  v-for="(item, index) in items"
  :item="item"
  :index="index"
  :key="item.id"
/>
```

The reason for not automatically injecting ```item``` into the component is because that makes the component tightly coupled to how ```v-for``` works. Being explicit about where its data comes from makes the component reusable in other situations.

## Array Change Detection

### Mutation Methods
Vue is able to detect when a reactive array's mutation methods are called and trigger necessary updates. These mutations methods are:

- ```push()```
- ```pop()```
- ```shift()```
- ```unshift()```
- ```splice()```
- ```sort()```
- ```reverse()```

### Replacing an Array
When working with non-mutating methods, like ```filter()```, ```concat()``` and ```slice()```, we should replace the old array with the new one:

``` 
// `items` is a ref with array value
items.value = items.value.filter((item) => item.message.match(/Foo/))
```

In this case the Vue doesn't throw away the existing DOM and re-render the entire list. So Vue implements some smart heuristics to maximize DOM element reuse, so replacing an array with another array containing overlapping objects is a very efficient operation.

## Displaying Filtered/Sorted Results
When you don't want to display filtered or sorted arrays without actually mutate or reseting the original data, you can create a computed property that returns the filtered or sorted array. For example:

``` 
//js
data() {
  return {
    numbers: [1, 2, 3, 4, 5]
  }
},
computed: {
  evenNumbers() {
    return this.numbers.filter(n => n % 2 === 0)
  }
}

//template
<li v-for="n in evenNumbers">{{ n }}</li>
```

In situations where computed properties are noe feasible (e.g. inside nested ```v-for``` loops), you can use a method:

``` 
//js
data() {
  return {
    sets: [[ 1, 2, 3, 4, 5 ], [6, 7, 8, 9, 10]]
  }
},
methods: {
  even(numbers) {
    return numbers.filter(number => number % 2 === 0)
  }
}

//template
<ul v-for="numbers in sets">
  <li v-for="n in even(numbers)">{{ n }}</li>
</ul>
```

Be careful with ```reverse()``` and ```sort()``` in a computed property! These two methods will mutate the original array, which should be avoided in computed getters. Create a copy of the original array before calling these methods:

``` 
- return numbers.reverse()
+ return [...numbers].reverse()
```
