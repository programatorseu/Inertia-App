<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>


iInertia.js

Client side routing library - connect server-side framework with client-side framework

- we can use authorization, controller even though we use single page application
- we do not need to use repo with API 



- we do not provide traditional post - request to server 

  > inertia intercept request and perform ajax request 

- routing -> loading client-side view and we pass filter and other args 



## 1.1 Install and configure inertia 

```js
composer require inertiajs/inertia-laravel
```

add root template -> to load site assets 

->  set up middleware  (to provide proper response)

Once generated, register the `HandleInertiaRequests` middleware in your `App\Http\Kernel`



**setup client side **

-> install vue3  -inertia

```bash
npm install @inertiajs/inertia @inertiajs/inertia-vue3
```

-> install vue next  & paackage  for **single file components**

```bash
 npm install vue@next
npm install -D @vue/compiler-sfc
```

res/app.js paste code from documentation 

1 interesting thing - how to track page 

```js
createInertiaApp({
  resolve: name => require(`./Pages/${name}`),
```



create res/js/Pages directory

-> add to webpack mix to turn on VUE + turn on on versioning support (add hash to compiled files )

-> install remaing depnedency 

```bash
npm install
npx mix # compile down according to our 
```

### 1.2 Pages

res/js/pages/Welcome.vue (template and script - simple vue file)

web.php -> add link with `inertia` function 

- compile and view in browser using server
- `return Inertia::render('Welcome');`

```js
    return Inertia::render('Welcome', [
        'name' => 'Piotrek'
    ]);
```



then to use - we need to accept props and output in client Side 

```bash
npx mix watch
```



### 1.3 Links

to mimic serve response wait : 

```php
Route::get('/users', function () {
    sleep(2);
```

Link component from vue !  ->> we do not perfom full page request !! 

- export component and wrap navigation items with `Link`

```js
<template>
<h1>Home</h1>
<nav>
    <ul>
        <li><Link href="/">Home</Link></li>
        <li><Link href="/users">Users</Link></li>
        <li><Link href="/settings">Settings</Link></li>
    </ul>
</nav>
</template>

<script>
import { Link } from "@inertiajs/inertia-vue3";
export default {
    components: {Link},
};
</script>
```

create ad Shared/Nav.vue

then to use it :

```js
<Nav />
</template>
<script>
import Nav from "../Shared/Nav";
export default {
    components: {Nav},
};
```

### 1.4 Progress indicators

> progress libraary 
>
> - pull in to app.js file  
> - at the bottom of app.js init 

```js
InertiaProgress.init();
```

### 1.5 Perform non-get requests

- patch or post 

```php
            <li><Link href="/logout" method="post">Log out</Link></li>
```

inertia use axios library / laravel pass csrf automatically 

when we click "ctr + click" to open new tab - it will be post request because it is post request 

we want to make it as button : 

```php
            <li><Link href="/logout" method="post" as="button">Log out</Link></li>
```

### 1.6 Preserve the Scroll position

```js
        <Link
            href="/users"
            class="text-blue-500"
            preserve-scroll>
            Refresh
        </Link>
    </div>
```

### 1.7 Active Links

```php
        <Link 
            href="/" 
            class="text-blue-500 hover:underline"
            :class="{'font-bold underline' : $page.component === 'Home'}">
            Home
        </Link>
```

or we can extract to seperate component 

NavLink :

-> template contains <Link> element

-> script : import Link from inertia / export k

```js
            <NavLink href="/" :active="$page.component === 'Home'">
                Home
            </NavLink>
```

### 1.8 Layout files 

we do not want to pull in manually nav many times 

- create Layout file shared/Layout.vue
     - with header and Nav 
     - with Section slot 
- import in home and other files 

```js
  <script>
  import Layout from "../Shared/Layout";
  export default {
    components: { Layout },
  };
  </script>
```

### 1.9 Shared Data 

`HandleInertiaRequests`  `@share` method 

```js
        return array_merge(parent::share($request), [
            'auth' => [
                'user' => [
                    'username' => 'Piotrek S'
                ]
            ]
        ]);
```

use computed property  in export default

```js
    computed: {
        username() {
        return this.$page.props.auth.user.username;
     }
    }
}
```

other way is manual pass

```php
Route::get('/', function () {
    return Inertia::render('Home', [
        'username' => 'Piotr'
    ]);
});

```

inside Home.vue



```php
<Layout :username="username">
    <script>
    	components: {Layout},
		props: {username: String}
    </script>
```

### 1.10 Global Component Registration 

to use Link - we need to import and then export as component 

app.js -> register link component 



now there is composition API so we can use `script setup`

```js
    <script setup>
    import Nav from "../Shared/Nav";
    import {Link} from "@inertiajs/inertia-vue3";
    defineProps({
        time: String
    })
    </script>
```

### 1.11 Persistent Layout

layout is child of page -> every time is deleted 

`layout: Layout` -> declare what is layouit for our component

```js
  import Layout from "../Shared/Layout";
  export default {
    layout: Layout
  };
```

 

### 1.12 Default Layout 

- we will not need to import layout in every component 

  app,js -> change `resolve`

```js
import Layout from "./Shared/Layout";
createInertiaApp({
  resolve: name => {
    let page = require(`./Pages/${name}`).default;
    // check if there is already Layout setup : 
    page.layout ??= Layout;
    return page;
  },
```

### 1.13 Code Splitting and Dynamic Imports

- webpack.mix we activate vue / versioning our assets 
   - add extract() --> automatically extracts dependencies from `node_modules` into own life 	
   - vendor JS file will be cached without forcing user to pay penalty of re-downloading all of vendor JS code 
- update app.blade.php to pull in `manifest` and vendor js files 

we want to load dynamically when user load page so inside `app.js` call `import` 

avoid users from downloading bunch of code that they will not need it 

```js
createInertiaApp({
  resolve: async name => {
    let page = (await import(`./Pages/${name}`)).default;
```

### 1.14 Dynamic Title and Meta Tags

Inertia Head component 

```js
<template>
    <Head>
        <title>My App -- Home</title>
    </Head>
      <h1 class="text-3xl">
        Home
      </h1>
  </template>
<script setup>
import {Head} from "@inertiajs/inertia-vue3";
</script>
  
```

we can import in layout

```js
import {Head} from "@inertiajs/inertia-vue3";
  
  export default {
    components: { Nav, Head },
```

or we can do it in App.js file 

```js
 setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .component("Link", Link)
      .component("Head", Head)
      .mount(el);
  },
  title: title => `My App - ${title}`
```

### 1.15 SPA Security Concern

-> accept list of users in `script`

```js
...
</template>
    <script setup>
        defineProps({users:Array})
    </script>
```



-> `li v-for`  - remember about setup key

-> migrate & seed users (default )

-> route file get Users from 

when we output - Vue Object get thousands data of information passed that should not be visible 

​	-> more data we do not need / entire structure of user table is passed to our Client  with that : 

```php
        'users' => User::all()
```

​	-> instead passing all - use map to return subset ! 

```php
        'users' => User::all()->map(fn($user) => [
           'name' => $user->name 
        ])
```

### 1.16 Pagination 

we need to call `paginate` on Users 

-> returns data & information about Paginator & `Links` property that give information about what page we are in 



-> accept User as object not Array then to get

```js
           <tr v-for="user in users.data" :key="user.id">
```

in the bottom get link to pages 

```js
 <Link 
        v-for="link in users.links"
        :href="link.url"
        v-html="link.label"
    />
```

Dynamic Component with `Component` -> Shared/Pagination.vue

```js
<Component
      :is="link.url ? 'Link' : 'span'"
```

to use it :

```js
  <Pagination :links="users.links" class="mt-6" />
</template>
    <script setup>
    import Pagination from "../Shared/Pagination";
        defineProps({users:Object})
    </script>
```

throught -> works like map but it is only applied to current bunch of dataa

```php
       'users' => User::paginate(10)->through(fn($user) => [
            'id' => $user->id,
            'name' => $user->name
        ])
```

### 1.17 Filtering / State and Query strings 

- having input  for search 
  - `v-model` and `ref` - initialize with empty string to -- it will make it reactive 

 - watch for changes ->> then make request - import Inertia to run `get` reqeuest  
   - remember about `preseverState` to not lose it what was typed in 

Laravel layer - call `Users::query()` it will allow us to run different methods 

- withQueryString  as part of pagination 



-> update input placeholder - pass request 

then on client-side

```js
  let props = defineProps({
            users:Object,
            filters: Object
        });
        
        let search =  ref(props.filters.search);
```

clicking back -> every character :) 

- replace option 

```js
 Inertia.get('/users', {search: value}, {
                preserveState:true,
                replace: true
            });
```

## 2. Forms

- new endpoint for users/create:

script-> pull reactive and wrap all fields 

```js
    import { reactive } from 'vue';
    let form = reactive({
        name: '',
        email: '',
        password: '',
    });
```

remember about adding `v-model='form.name'`

and ssumibt 

```js
   let submit = () => {
        Inertia.post('/users', form);
    };

```

-> laravel post - validate simple request

- add mutator in Model to crypt password



**error displaying**

```js
<div v-if="errors.password" v-text="errors.password" class="text-red-500 text-xs mt-1"></div>
...
  <script setup>
    import { reactive } from 'vue';
    import { Inertia } from '@inertiajs/inertia';
    defineProps({
        errors: Object
    });
```



### 2.1 Inertia's Form Helper

throttle problem - while loading we send to many request 

-> we need to disable imediately button 

- import `ref`

​	inside script add : 

```js
    let processing = ref(false);
...
 let submit = () => {
        processing.value = true
```



and add to button 

```js
:disabled="processing">Submit</button>
```

we want to turn on and turn off conditionally 

-> `submit` -> pass option for inertia : 



```js
    let submit = () => {
        Inertia.post('/users', form, {
            onStart: () => {processing.value = true},
            onFinish: () => {processing.value = false}
        });
    };
```

Inertia automate : 

-> useForm  import and grab entire form with `useForm`

-> change error display ` <div v-if="form.errors.email" v-text="form.errors.email" `



### 2.2 Better Performance with Throttle and Debounce

-> using search input and typing words consisted of 10 characters - we make 10 requests 

-> use lodash

```js
        import throttle from "lodash/throttle";
```

-> update watch - throttle once per 500 ms   

* `debounce` will do 1 after 500 ms - after we stop typing 

```js
        watch(search, throttle(function (value) {
            Inertia.get('/users', { search: value }, { 
                preserveState: true, 
                replace: true });
        }, 300));
```

### 3. Authentication with Inertia
-> set at the top  middleware and group every route 

-> create controller and proper method + vue files

-> change `app.js` to provide layout without nav 

-> Login.vue - `inertia form` as helper and we will follow the similar 

-> authentication QuickStart ! - **optional**

`redirect()->intendended()`  rememer in sesion where we where heading to go

- update HandleInertiaRequest