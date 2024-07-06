# Vue Router

## An Example

To introduce some of the main ideas, we're going to consider this example:

Let's start by looking at the root component, ```App.vue```.

``` 
<template>
  <h1>Hello App!</h1>
  <p>
    <strong>Current route path:</strong> {{ $route.fullPath }}
  </p>
  <nav>
    <RouterLink to="/">Go to Home</RouterLink>
    <RouterLink to="/about">Go to About</RouterLink>
  </nav>
  <main>
    <RouterView />
  </main>
</template>
```

Instead of using regular ```<a>``` tags, we use the custom component ```RouterLink``` to create links. This allows Vue Router to change the URL without reloading the page, handle URL generation, encoding, and various other features.

The ```RouterView``` component tells Vue Router where to render the current route component. That's the component that corresponds to the current URL path. It doesn't have to be in ```App.vue```, you can put it anywhere to adapt it to your layout, but it does need to be included somewhere, otherwise Vue Router won't render anything.

The example above also uses ```{{ $route.fullPath }}```. You can use ```$route``` in your component templates to access an object that represents the current route.

## Creating the router instance

The router instance is created by calling the function ```createRouter()```:

``` 
import { createMemoryHistory, createRouter } from 'vue-router'

import HomeView from './HomeView.vue'
import AboutView from './AboutView.vue'

const routes = [
  { path: '/', component: HomeView },
  { path: '/about', component: AboutView },
]

const router = createRouter({
  history: createMemoryHistory(),
  routes,
})
```

The ```routes``` options defines the routes themselves, mapping URL paths to components. The component specified by the ```component``` option is the one that will be rendered by the ```<RouterView>``` in our earlier ```App.vue```. These routes components are sometimes referred to as views, though they are just normal Vue components.

The ```history``` optiuon controls how routes are mapped onto URLs and vice versa. For the Playground example we're using ```createMemoryHistory()```, which ignores the browser URL entirely and uses its own internal URL instead. That works well for the Playground, but it's unlikely to be what you'd want in a real application. Typically, you'd want to use ```createWebHistory()``` instead, or perhaps ```createWebHashHistory()```.

## Registering the router plugin

Once we've created our router instance, we need to register it as a plugin by calling ```use()``` on our application:

``` 
createApp(App)
  .use(router)
  .mount('#app')
```

Or, equivalently:

``` 
const app = createApp(App)
app.use(router)
app.mount('#app')
```

Like with most Vue plugins, the call to ```use()``` needs to happen before the call to ```mount()```.

If you're curious about what this plugin does, some of its responsabilities include:

1. Globally registering the ```RouterView``` and ```RouterLink``` components.

2. Adding the global ```$router``` and ```$route``` properties.

3. Enabling the ```useRouter()``` and ```useRoute()``` composables.

4. Triggering the router to resolve the initial route.

## Accessing the router and current route

If you're exporting the router instance from an ES module, you could import the router instance directly where you need it. In some cases this is the best approach, but we have other options if we're inside a component.

In component templates, the router instance is exposed as ```$router```.

If we're using the Options API, we can access these same two properties as ```this.$router``` and ```this.$route``` in our JavaScript code. The ```HomeView.vue``` component in the Playground example accesses the router that way:

``` 
export default {
  methods: {
    goToAbout() {
      this.$router.push('/about')
    },
  },
}
```

This method is calling ```push()```, which is used for programmatic navigation.

With the Composition API, we don't have access to the component instance via ```this```, so Vue Router includes some composable that we can use instead. ```AboutView.vue``` in the Playground example is using that approach:

``` 
<script setup>
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const router = useRouter()
const route = useRoute()

const search = computed({
  get() {
    return route.query.search ?? ''
  },
  set(search) {
    router.replace({ query: { search } })
  }
})
</script>
```

