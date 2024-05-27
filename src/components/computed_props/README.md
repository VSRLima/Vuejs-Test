# Basic Example

Computed is indicated to use to replace complex logic that could be used in-templates ref, for example if we have an nested object like this:
```
const author = reactive({
    name: 'John Doe',
    books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
    ]
})
```

And we want to check if the author has published books, we could do something like:
```
<p>Has published books:</p>
<span>{{ author.books.length > 0 ? 'Yes' : 'No' }}</span>
```

At this point, the template is getting a bit cluttered. We have to took it for a second before realizing that it performs a calculation depending on ```author.books```. More importantly, we probably don't want to repeat ourselves if we need to include this calculation in the template more than once.

That's why for complex logic that includes reactive data is recommend that we use computed property. Here's a little example refactoring the code above:
```
<script setup>
import { reactive, computed } from 'vue'

const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})

// a computed ref
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})
</script>

<template>
  <p>Has published books:</p>
  <span>{{ publishedBooksMessage }}</span>
</template>
```

Here we declared a computed property called ```publishedBooksMessage``` that takes the logic and it's easier to be used more that once. Also, it's important to mention that the computed property is auto-unwrapped, this mean that we can refer to its value without using ```.value``` in template expression. Futhermore, a computed property is automatically tracks its reactive dependencies. Vue is aware that the prop ```publishedBooksMessage``` depends on ```author.books```, so it will update any bindings that depend on ```publishedBooksMessage``` when ```author.books``` changes.

# Computed Caching X Methods

You may have noticed that if we define a method like this:

```
//in template
<p>{{ calculateBooksMessage() }}</p>

// in component
function calculateBooksMessage() {
  return author.books.length > 0 ? 'Yes' : 'No'
}
```

We'll achieve the same result. However, the difference is that computed properties are cached based on their reactivities dependencies. This means that as long as ```author.books``` has not changed, multiple access to ```publishedBooksMessage``` will immediately return the previously computed result without having to run the getter function again. 

This also means that the following computed property will never updates, since ```Data.now()``` isn't a reactive dependency:
```
const now = computed(() => Date.now());
```

In comparison, a method invocation will always run a getter function whenever a re-render happens.

Why do we need caching? Imagine that we have a huge computed property ```list```, which requires looping through a huge array and do a lot of computations. Then we have others properties that depends on this ```list```. Without caching, we would be executing ```list```´s getter many more times than necessary! 

# Writable Computed

Computed properties are by default only getter functions. If you attempt to assign a new value to a computed property, you will receive a runtime warning. In the rare cases where you need a "writable" computed property, you can create it by providing getter and setter functions
```
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  // getter
  get() {
    return firstName.value + ' ' + lastName.value
  },
  // setter
  set(newValue) {
    // Note: we are using destructuring assignment syntax here.
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
</script>
```

Now when you run ```fullName.value = 'John Doe'```, the setter will be invoked and ```firstName``` and ```lastName``` will be updated accordingly.

# Best Practices

## Getters should be side-effect free

Its important to remember that computed getter functions should only perform pure computation and be free of side effects. For example, don't mutate other state, make async requests, or mutate DOM inside a computed getter! Think of computed as declarative describing how to derive a value based on other values - its only responsibility should be computing and return that value.

## Avoid mutating computed value

The returned value from a computed property is derived state. Think of it as a temporary snapshot - every time the source changes, a new snapshot is created. It doesn't make sense to mutate a snapshot, so a computed return value should be treated as a read-only and never be mutated. Instead, update the source state it depends on to trigger a new computation.