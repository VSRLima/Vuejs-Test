# Single-File Components

## Why SFC

While SFCs require a build step, there are numerous benefits in return:

- Author modularized components using familiar HTML, CSS and JavaScript syntax.
- Colocation of inherently coupled concerns
- Pre-compiled templates without runtime compilation cost
- Component-scoped CSS
- More ergonomic syntax when working with Composition API
- More compile-time optimizations by cross-analyzing template and script
- IDE support with auto-completion and type-checking for template expressions
- Out of the box Hot-Module Replacement (HMR) support

SFC is a defining feature of Vue as a framework, and is the recommended approach for using Vue in the following scenarios:

- Single-Page Applications (SPA)
- Static Site Generation (SSG)
- Any non-trivial frontend where a build step can be justified for better development experience (DX)

## How it works

Vue SFC is a framework-specific file format and must be pre-compiled by ```@vue/compiler-sfc``` into standard JavaScript and CSS. A compiled SFC is a standard JavaScript (ES) module - which means with proper build setup you can import an SFC like a module:

``` 
import MyComponent from './MyComponent.vue'

export default {
  components: {
    MyComponent
  }
}
```

```<style>``` tags inside SFCs are typically injected as native ```<style>``` tags during development to support hot updates. For production they can be extracted and merged into a single CSS file.

In actual projects, we typically integrate SFC compiler with a build tool like Vite and Vue-CLI (which is based on webpack), and Vue provides official scaffolding tools to get you started with SFC as fast as possible.