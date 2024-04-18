# All about vue.js

- Ways of using Vue

  - Standalone Script

Vue can be used as a standalone script file if you have a backend framework that’s already rendering most of the HTML, or a frontend logic that isn’t complex enough to justify a build step.
Vue also provides an alternative distribution for this case that is a petite-vue, basically is a optimize, light-weight version of vue that enhance the existing HTML.

- Embedded Web Components

There’s a version of vue to build standard web components that can be embedded in any HTML page, regardless of how they are rendered.

- Single-Page Application (SPA)

Basically Vue is based on SPA (Single-Page-Application) and it means that it offers a rich interactivity, deep session depth and non-trivial stateful logic on the frontend. With this, vue provides core libraries and comprehensive tooling support with amazing developer experience for building modern SPAs, including:

- Client-side router
- Blazing fast build tool chain
- IDE support
- Browser devtools
- TypeScript Integrations
- Testing utilities

- FullStack/SSR

Pure client-side SPAs are problematic when the app is sensitive to SEO and time-to-content. So vue provides first-class APIs to “render” a Vue app into HTML strings on the server, allowing end users to see content immediately while the JavaScript is being downloaded. This is called Server-Side-Rendering (SSR) and it greatly improves Core Web Vital metrics such as Largest Contentful Paint (LCP) that is a metric responsible for measuring perceived load speed, it marks the point in the page load timeline when the page’s main content has likely loaded. So a fast LCP helps reassure the user that the page is useful.

- JAMStack/SSG

To improve site performance we sometimes require that the some contents be entirely pre-rendered into HTML statically. With this, we don’t need to dynamically render pages on each request as we do in Server-Side-Rendering, so this is called Static-Site Generation (SSG), also know as JAMStack.

There are two vertents of SSG: single-page and multi-page. Both of them pre-renders the site into a static HTML, but the difference is:

1. After the initial page load, a single-page SSG “hydrates” the page into a SPA. This requires more upfront JS payload and hydration cost, but subsequent navigation will be faster, since it only need to partially updates the page content instead of reloading the entire page.

2. A multi-page SSG loads a new page on every navigation. The upside is that it can ship a minimal JS or no JS at all if the page requires no interaction! Some multi-page SSG frameworks such as Astro also support “partial hydration” - which allows you to use Vue components to create interactive “islands” inside static HTML.

Single-page SSGs are better suited if you expect non-trivial interactivity, deep session lengths, or persisted elements / state across navigations. Otherwise, multi-page SSG would be the better choice.

- Differents ways of downloading Vue
  Besides the normal way, using the node to install vue in our project, there are using CDN. The first way of doing it is the most basic one

```
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```

And using it in the global build:

```
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

    <div id="app"> {{message}} </div>

    <script>
        const { createApp, ref } = Vue;

        createApp({
            setup () {
                const message = ref('Hello World');
                return {
                    message
                }
            }
        }).mount('#app');
    </script>
```

Another way of doing it with CDN is using ES Module Build. Most modern browsers now support ES modules natively, so we can use Vue from a CDN via native ES modules like this:

```
    <div id="app"> {{message}} </div>

    <script type="module">
        import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'

        createApp({
            setup () {
                const message = ref('Hello World');
                return {
                    message
                }
            }
        }).mount('#app');
    </script>
```

Notice that we’re using `<script type=”module”>`, and the imported CDN URL is pointing to the ES modules build of Vue instead.

We can teach the browser where to locate the vue import by using Import Maps:

```
    <script type="importmap">
        {
            "imports": {
                "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
            }
        }
    </script>

    <script type="module">
        import { createApp, ref } from 'vue'

        createApp({
            setup () {
                const message = ref('Hello World');
                return {
                    message
                }
            }
        }).mount('#app');
    </script>
```

As we dive deeper into the construction of site, we may need to split our code into separate JavaScript file so that they’re easier to manage. For example:

```
    <!-- index.html -->
    <div id="app"></div>

    <script type="module">
        import { createApp } from 'vue';
        import MyComponent from './my-component.js'

        createApp(MyComponenent).mount('#app')
    </script>

    // my-component.js

    import { ref } from 'vue'

    export default {
        setup() {
            const count = ref(0)
            return { count }
        },

        template: `<div>count is {{ count }}</div>`
    }
```
