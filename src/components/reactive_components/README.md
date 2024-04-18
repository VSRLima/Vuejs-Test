# Reactivity Fundamentals

- Declaring Reactive State

`ref ()` is the recommended way of declaring reactive state and it could be like it:

```
    import { ref } from 'vue'

    const count = ref(0)
```

So basically the ref takes an argument and returns a wrapped within a ref object with a prop `.value` :

```
    const count = ref(0)

    console.log(count) // { value: 0 }
    console.log(count.value) // 0

    count.value++
    console.log(count.value) // 1
```

Note: There’s a way in TS to properly type the [refs](https://vuejs.org/guide/typescript/composition-api#typing-ref)

In case you want to access refs in a component’ template, declare a return them from a component’s `setup()`:

```
    import { ref } from 'vue'

    export default {
    // `setup` is a special hook dedicated for the Composition API.
    setup() {
            const count = ref(0)

            // expose the ref to the template
            return {
            count
            }
        }
    }

    <div>{{ count }}</div>
```

Notice that in this case you don’t need to append `.value` when using the ref in the template. For convenience, refs are automatically unwrapped when used inside templates (with a few caveats)

# Caveats

Ref unwrapping will only occur if it’s a top-level property in the template render context.

In the example below, count and object are top-level props, but object.id isn’t:

Therefore, this expression works as expected:
While this one doesn’t:
It’s results will be [object Object]1 bec
