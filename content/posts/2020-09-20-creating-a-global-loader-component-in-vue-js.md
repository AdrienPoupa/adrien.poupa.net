---
title: 'Creating a Global Loader Component in Vue.js'
date: '2020-09-20T17:23:54-04:00'
author: Adrien Poupa
url: /creating-a-global-loader-component-in-vue-js/
categories:
    - Vue.js
---

In any frontend application, invariably, one has to deal with showing the user an HTTP request is in progress. This is usually done by displaying a spinner while the request is being executed.

As I got bored dealing with Axios callbacks and independent `isLoading` data property in my Vue.js components, I realized I needed a global solution that would prevent me from cluttering my components with those redundant data properties.

But I also wanted my solution to work with simultaneous requests. And this is where the tricky part comes in, because on most posts I’ve seen, the author assumes you only ever have one API call that toggles a single loading switch. However, in real life, you have have 2, 3 or more API calls to build a page.

You may want to hold off displaying the page until all of the data is there, or maybe you need 2 out of the 3 requests before showing the page. My solution allows you to do both. It is inspired from [this post](https://medium.com/@lukasbaceviius/loading-indicator-with-every-async-request-vue-edition-26731b2b2864).

Long story short, I created a new Vuex module to store the loading state, that’s determined by the number of requests currently being made. This number is increased or decreased using Axios interceptors, meaning they will be triggered whenever you do a call to `axios[get|post|put|patch|delete|etc]`. I also added an option that allows you to disable the loader if need be.

Let’s start by reviewing the Vuex module, that I’ll call loader.js:

```js
export const loader = {
    namespaced: true,
    state: {
        loading: false,
        requestsPending: 0,
    },
    actions: {
        show({ commit }) {
            commit("show");
        },
        hide({ commit }) {
            commit("hide");
        },
        pending({ commit }) {
            commit("pending");
        },
        done({ commit }) {
            commit("done");
        }
    },
    mutations: {
        show(state) {
            state.loading = true;
        },
        hide(state) {
            state.loading = false;
        },
        pending(state) {
            if (state.requestsPending === 0) {
                this.commit("loader/show");
            }

            state.requestsPending++;
        },
        done(state) {
            if (state.requestsPending >= 1) {
                state.requestsPending--;
            }

            if (state.requestsPending <= 0) {
                this.commit("loader/hide");
            }
        }
    }
};

```

In a Vue CLI application, this would go under src/store/modules/loader.js. Then I load it in Vuex with the following store/index.js file:

```js
import Vue from 'vue';
import Vuex from 'vuex';
import { loader } from './modules/loader';

Vue.use(Vuex);

export default new Vuex.Store({
    modules: {
        loader,
    },
})
```

Now, we need to setup the Axios interceptors to trigger the state changes automatically.

First, after the Axios import in your main.js file, enable the showLoader option:

```js
axios.defaults.showLoader = true;
```

Please note [there was a bug](https://github.com/axios/axios/pull/2006) in the 0.19.0 release of Axios causing custom options being removed, so either use &gt;= 0.19.1 or 0.18.

In the created function of the `new Vue` instantiation, setup the interceptors as follow:

```js
created() {
    axios.interceptors.request.use(
        config => {
            if (config.showLoader) {
                store.dispatch('loader/pending');
            }
            return config;
        },
        error => {
            if (error.config.showLoader) {
                store.dispatch('loader/done');
            }
            return Promise.reject(error);
        }
    );
    axios.interceptors.response.use(
        response => {
            if (response.config.showLoader) {
                store.dispatch('loader/done');
            }

            return response;
        },
        error => {
            let response = error.response;

            if (response.config.showLoader) {
                store.dispatch('loader/done');
            }

            return Promise.reject(error);
        }
    )
}

```

This is basically calling the store module we just created to increase the number of requests in progress just before the HTTP call is made, and reducing it when it finishes or an error is encountered.

What comes next depends on your application. Maybe you want to show an overlay when requests are made. maybe you want to remove all the content when a query is made…

Whatever you want to do, you can know if there is a request pending by adding a mapState to your computed block:

```js
computed: {
    ...mapState('loader', ['loading'])
},
```

Then, in your layout template or wherever, you can do as follows:

```js
<loader v-if="loading" />
<slot v-else></slot>
```

My Loader component simply shows a Vuetify `v-progress-circular` component, but this is of course entirely up to you.

Finally, if for some reason you want to do disable the loader for a specific request, you can do it by passing the showLoader option to Axios like so:

```js
axios.get('api/your-endpoint', { showLoader: false })
```

I hope you found this useful!