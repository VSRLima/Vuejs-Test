# Caveats

Ref unwrapping will only occur if it’s a top-level property in the template render context.

In the example below, count and object are top-level props, but object.id isn’t:

```
    const count = ref(0)
    const object = { id: ref(1) }
```

Therefore, this expression works as expected:

```
    {{ count + 1 }}
```

While this one doesn’t:

```
    {{ object.id + 1 }}
```

It’s results will be `[object Object]1` because the `object.id` isn't unwrapped when evaluating the expression and remains a ref object. To fix this, we can destructuring it in a Top-level, it'll be like:

```
    const { id } = object  //js

    {{ id + 1 }} //template
```

Now it'll render the value `2`.

Another thing to note is that ref does unwrapped if it is the final evaluated value of a text interpolation, so the following will render:

```
    {{ object.id}}
```

This is just a convinience feature of a text interpolation equivalent to this:

```
    {{ object.id.value }}
```
