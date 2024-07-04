# Watchers

We can use watchers to apply modifications when some computed props changes, as like a "side effect". We can do this using ``` watch``` option:

```
//js
export default {
  data() {
    return {
      question: '',
      answer: 'Questions usually contain a question mark. ;-)',
      loading: false
    }
  },
  watch: {
    // whenever question changes, this function will run
    question(newQuestion, oldQuestion) {
      if (newQuestion.includes('?')) {
        this.getAnswer()
      }
    }
  },
  methods: {
    async getAnswer() {
      this.loading = true
      this.answer = 'Thinking...'
      try {
        const res = await fetch('https://yesno.wtf/api')
        this.answer = (await res.json()).answer
      } catch (error) {
        this.answer = 'Error! Could not reach the API. ' + error
      } finally {
        this.loading = false
      }
    }
  }
}

//template
<p>
  Ask a yes/no question:
  <input v-model="question" :disabled="loading" />
</p>
<p>{{ answer }}</p>
```

The ```watch``` option also supports a dot-delimited path as key: 

```
export default {
  watch: {
    // Note: only simple paths. Expressions are not supported.
    'some.nested.key'(newValue) {
      // ...
    }
  }
} 
```

## Deep Watchers

```watch``` is shallow by default: the callback will only trigger when the watched property has been assigned a new value - it won't trigger in nested properties changes. If you want a call back to fire in all nested mutations, you need to use a deep watcher:

```
export default {
  watch: {
    someObject: {
      handler(newValue, oldValue) {
        // Note: `newValue` will be equal to `oldValue` here
        // on nested mutations as long as the object itself
        // hasn't been replaced.
      },
      deep: true
    }
  }
} 
```

### Tip

Deep watchers requires traversing all nested properties in the watched object, and can be expensive using in large data structures. 

## Eager Watchers

```watch``` is lazy by default: the callback won't be called until the watched source has changed. But in some cases we may want the same callback logic to be run eagerly - for example, we may want to fetch some initial data, and then re-fetch the data whenever relevant state changes.

We can force a watcher's callback to be executed immediately by declaring it using an object with a ```handler``` function and the ```immediate: true``` option:

```
export default {
  // ...
  watch: {
    question: {
      handler(newQuestion) {
        // this will be run immediately on component creation.
      },
      // force eager callback execution
      immediate: true
    }
  }
  // ...
} 
```

The initial execution of the handler function will happen just before the ```created``` hook. Vue will have already processed the ```data```, ```computed``` and ```methods``` options, so those properties will be available on the first invocation.

## Once Watchers

If you want the callback to trigger only once when the source changes, use the ```once: true``` option.

```
export default {
  watch: {
    source: {
      handler(newValue, oldValue) {
        // when `source` changes, triggers only once
      },
      once: true
    }
  }
} 
```

## Callback Flusing Timing

By default, a watcher's callback is called after parent component updates (if any), and before the owner component's DOM updates. This means if you attempt to access the owner component's own DOM inside a watcher callback, the DOM will be in a pre-update state.

### Post Watchers

If you want to access the owner component's DOM in watcher callback after the Vue updates it, you'll need to specify ```flush: 'post'``` option: 

```
export default {
  // ...
  watch: {
    key: {
      handler() {},
      flush: 'post'
    }
  }
} 
```

### Sync Watchers

It's also possible to create a watcher that fires synchronously, before Vue-managed updates:

``` 
export default {
  // ...
  watch: {
    key: {
      handler() {},
      flush: 'sync'
    }
  }
}
```

#### TIP

Sync watchers don't have batching and triggers every time a reactive mutation is detected. It's ok to use them to watch simple boolean values, but avoid using them on data sources that might be synchronously mutated many times e.g arrays.

## ```this.$watch()```

It's also possible to imperatively create watchers using the ```$watch()``` instance method:

``` 
export default {
  created() {
    this.$watch('question', (newQuestion) => {
      // ..
    })
  }
}
```

This is useful when you need to conditionally set up a watcher, or only watch something in response to user interaction. It also allows you to stop the watcher early.

## Stopping a Watcher

Watchers declared using the ```watch``` option or the ```$watch()``` instance method are automatically stopped when the owner component is unmounted.

In the rare cases where you need to stop a watcher before the owner component unmounts, the ```$watch()``` API returns a function for that: 

```
const unwatch = this.$watch('foo', callback)

// ...when the watcher is no longer needed:
unwatch() 
```