## Vue js Authentication Guide
### Installation

```bash
npm init vue@latest
```
This command will install and execute `create-vue`, the official Vue project scaffolding tool. You will be presented with prompts for several optional features such as TypeScript and testing support:

```bash
✔ Project name: … <your-project-name>
✔ Add TypeScript? … No / Yes
✔ Add JSX Support? … No / Yes
✔ Add Vue Router for Single Page Application development? … No / Yes
✔ Add Pinia for state management? … No / Yes
✔ Add Vitest for Unit testing? … No / Yes
✔ Add Cypress for both Unit and End-to-End testing? … No / Yes
✔ Add ESLint for code quality? … No / Yes
✔ Add Prettier for code formatting? … No / Yes
Scaffolding project in ./<your-project-name>...
Done.
```

-If you are unsure about an option, simply choose No by hitting enter for now. Once the project is created, follow the instructions to install `dependencies` and start the `dev server`:
```bash
> cd <your-project-name>
> npm install
> npm run dev ## run dev server
```
- Install more dependencies — vuex and axios:
```bash
npm install vuex axios
```

# Setup
## 1. Vuex Configuration With Axios 

`Axios` is a JavaScript library that is used to send requests from the browser to APIs

`Vuex` is a `state management pattern + library` for Vue.js applications. It serves as a centralized store for all the components in an application, with rules ensuring that the state can only be mutated in a predictable fashion.

> Vuex is a store used in a Vue application that allows us to save data that will be available to every component and provide ways to change such data. We’ll use `Axios` in `Vuex` to send our requests and make changes to our state (data). Axios will be used in Vuex actions to send `GET` and `POST`, response gotten will be used in sending information to the `mutations` and which updates our store data.

> To deal with Vuex resetting after refreshing we will be working with `vuex-persistedstate, a library that saves our Vuex data between page reloads.`
```bash
npm install vuex-persistedstate
```
- Now let’s create a new folder store in `src`, for configuring the `Vuex store`. In the `store` folder, create a new folder; modules and a file `index.js`. 

> Note: It’s important to note that you only need to do this if the folder does not get created for you automatically.

```js
// src/store/index.js

import Vuex from 'vuex';
import Vue from 'vue';
import createPersistedState from "vuex-persistedstate";
import auth from './modules/auth';

// Load Vuex
Vue.use(Vuex);
// Create store
export default new Vuex.Store({
  modules: {
    auth
  },
  plugins: [createPersistedState()]
});
```
Here we are making use of `Vuex` and importing an auth `module` from the modules folder into our store.

### 1.1. MODULES
Modules are different segments of our store that handles similar tasks together, including:

    - state
    - actions
    - mutations
    - getters

Before we proceed, let’s edit our `main.js` file

```js
// src/main.js
import Vue from 'vue'
import App from './App.vue'
import router from './router';
import store from './store';
import axios from 'axios';

axios.defaults.withCredentials = true
axios.defaults.baseURL = 'http://localhost:8000/api'; // change this to your api url

Vue.config.productionTip = false
new Vue({
  store,
  router,
  render: h => h(App)
}).$mount('#app')
```
We imported the store object from the `./store` folder as well as the `Axios` package.

> Note
- In the `snippet above`, we do that using `axios.defaults.withCredentials = true`, this is needed because by default cookies are not passed by Axios.

- `axios.defaults.withCredentials = true` is an instruction to Axios to send all requests with credentials such as; authorization headers, TLS client certificates, or cookies (as in our case).

- We set our `axios.defaults.baseURL` for our Axios request to our API. This way, whenever we’re sending via Axios, it makes use of this base URL. With that, we can add just our endpoints like `/register` and `/login` to our actions without stating the `full URL each time`.

Now inside the modules folder in `store` create a file called `auth.js` and add the following code:

```js
//store/modules/auth.js

import axios from 'axios';
const state = {

};
const getters = {

};
const actions = {

};
const mutations = {

};
export default {
  state,
  getters,
  actions,
  mutations
};
```
Here we have our `auth module` with the `state, getters, actions, and mutations` properties.

#### 1.1.1 State
- In our state dict, we are going to define our data, and their default values:
```js
const state = {
  user: null,
  posts: null,
};
```
- We are setting the default value of state, which is an object that contains `user` and `posts` with their initial values as null.

#### 1.1.2 Getters
- Getters are functionalities to get the state. It can be used in multiple components to get the current state. The `isAuthenticatated` function checks if the state.user is defined or null and returns true or false respectively. `StatePosts` and `StateUser` return state.posts and state.user respectively value.
```js
const getters = {
    isAuthenticated: state => !!state.user,    
    StatePosts: state => state.posts,
    StateUser: state => state.user,
};
```
#### 1.1.3 Mutations
- Mutations are functionalities to change the state. It can be used in multiple components to change the current state. The `setUser` function sets the state.user to the payload value. `setPosts` and `setUser` set state.posts and state.user respectively value.
```js
const mutations = {
  setUser(state, username) {
    state.user = username;
  },

  setPosts(state, posts) {
    state.posts = posts;
  },
  logout(state, user) {
    state.user = user;
  },
};
```
- Each `mutation` takes in the state and a value from the action committing it, aside `logout`. The value gotten is used to change certain parts or all or like in `logout` set all variables back to null.

#### 1.1.4 Actions 
- Actions are functions that are used to commit a mutation to change the state or can be used to dispatch i.e `calls another action.` It can be called in different components or views and then commits mutations of our state;

##### 1.1.4.1. Register Action
- Our `Register action` takes in form data, sends the data to our `/register` endpoint, and assigns the response to a variable response. Next, we will be dispatching our form username and password to our login action. This way, we log in the user after they sign up, so they are redirected to the `/posts` page.
```js
async Register({dispatch}, form) {
  await axios.post('register', form)
  let UserForm = new FormData()
  UserForm.append('username', form.username)
  UserForm.append('password', form.password)
  await dispatch('LogIn', UserForm)
},
```
##### 1.1.4.2. Login Action

- Here is where the main authentication happens. When a user fills in their username and password, it is passed to a `User` which is a `FormData` object, the `LogIn` function takes the `User` object and makes a POST request to the `/login` endpoint to log in the user.

The Login function finally commits the username to the `setUser` mutation.
```js
async LogIn({commit}, User) {
  await axios.post('login', User)
  await commit('setUser', User.get('username'))
},
```
##### 1.1.4.3. Create Post Action

- Our `CreatePost` action is a function, that takes in the post and sends it to our `/post` endpoint, and then dispatches the `GetPosts` action. This enables the user to see their posts after creation.
```js
async CreatePost({dispatch}, post) {
  await axios.post('post', post)
  await dispatch('GetPosts')
},
```
##### 1.1.4.4. Get Posts Action

- Our `GetPosts` action sends a GET request to our `/posts` endpoint to fetch the `posts` in our API and commits `setPosts` mutation.
```js
async GetPosts({ commit }){
  let response = await axios.get('posts')
  commit('setPosts', response.data)
},
```
##### 1.1.4.5. Log Out Action
- Our `LogOut` action removes our user from the `browser cache`. It does this by committing a logout
```js
async LogOut({commit}){
  let user = null
  commit('logout', user)
}
```
> Note the whole code for the `auth.js` file is as follows:
```js
//store/modules/auth.js
import axios from "axios";

const state = {
  user: null,
  posts: null,
};

const getters = {
  isAuthenticated: (state) => !!state.user,
  StatePosts: (state) => state.posts,
  StateUser: (state) => state.user,
};

const actions = {
  async Register({dispatch}, form) {
    await axios.post('register', form)
    let UserForm = new FormData()
    UserForm.append('username', form.username)
    UserForm.append('password', form.password)
    await dispatch('LogIn', UserForm)
  },

  async LogIn({commit}, user) {
    await axios.post("login", user);
    await commit("setUser", user.get("username"));
  },

  async CreatePost({ dispatch }, post) {
    await axios.post("post", post);
    return await dispatch("GetPosts");
  },

  async GetPosts({ commit }) {
    let response = await axios.get("posts");
    commit("setPosts", response.data);
  },

  async LogOut({ commit }) {
    let user = null;
    commit("logout", user);
  },
};

const mutations = {
  setUser(state, username) {
    state.user = username;
  },

  setPosts(state, posts) {
    state.posts = posts;
  },
  logout(state, user) {
    state.user = user;
  },
};

export default {
  state,
  getters,
  actions,
  mutations,
};
```
### 1.2. Setting Up Components
#### 1.2.1. `NavBar.vue` AND `App.vue` COMPONENTS 
- In your `src/components` folder, delete the HelloWorld.vue and a new file called `NavBar.vue`.

- This is the component for our navigation bar, it links to different pages of our component been routed here. Each router link points to a `route/page` on our app.

- The `v-if="isLoggedIn"` is a condition to display the Logout link if a user is logged in and hide the `Register` and `Login` routes. We have a logout method which can only be accessible to `signed-in users`, this will get called when the Logout link is clicked. It will dispatch the `LogOut` action and then direct the user to the login page.

```js
//src/components/NavBar.vue
<template>
  <div id="nav">
    <router-link to="/">Home</router-link> |
    <router-link to="/posts">Posts</router-link> |
    <span v-if="isLoggedIn">
      <a @click="logout">Logout</a>
    </span>
    <span v-else>
      <router-link to="/register">Register</router-link> |
      <router-link to="/login">Login</router-link>
    </span>
  </div>
</template>
<script>
export default {
  name: 'NavBar',
  computed : {
      isLoggedIn : function(){ return this.$store.getters.isAuthenticated}
    },
    methods: {
      async logout (){
        await this.$store.dispatch('LogOut')
        this.$router.push('/login')
      }
    },
}
</script>
<style>
#nav {
  padding: 30px;
}
#nav a {
  font-weight: bold;
  color: #2c3e50;
}
a:hover {
  cursor: pointer;
}
#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
```
- Now edit your `App.vue` component to look like this:
```js
//src/App.vue
<template>
  <div id="app">
    <NavBar />
    <router-view/>
  </div>
</template>
<script>
// @ is an alias to /src
import NavBar from '@/components/NavBar.vue'
export default {
  components: {
    NavBar
  }
}
</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
```
#### 1.2.2. Views Components
- Views components are different pages on the app that will be defined under a route and can be accessed from the navigation bar. To get started Go to the `views` folder, delete the `About.vue` component, and add the following components:
    - Home.vue
    - Register.vue
    - Login.vue
    - Posts.vue
##### 1.2.2.1. Home.vue
- Rewrite the Home.vue to look like this:
```js
// src/views/Home.vue
<template>
<div class="home">
<p>Heyyyyyy welcome to our blog, check out our posts</p>
</div>
</template>
<script>

export default {
name: 'Home',
components: {
}
}
</script>
```
- This will display a welcome text to the users when they visit the homepage.

##### 1.2.2.2. Register.vue
- This is the Page we want our users to be able to sign up on our application. When the users fill the form, their information is been sent to the API and added to the database then logged in.

- Looking at the API, the `/register` endpoint requires a username, full_name and password of our user. Now let’s create a page and a form to get those information:
```js
// src/views/Register.vue
<template>
  <div class="register">
      <div>
          <form @submit.prevent="submit">
            <div>
              <label for="username">Username:</label>
              <input type="text" name="username" v-model="form.username">
            </div>
            <div>
              <label for="full_name">Full Name:</label>
              <input type="text" name="full_name" v-model="form.full_name">
            </div>
            <div>
              <label for="password">Password:</label>
              <input type="password" name="password" v-model="form.password">
            </div>
            <button type="submit"> Submit</button>
          </form>
      </div>
      <p v-if="showError" id="error">Username already exists</p>
  </div>
</template>
```
- In the Register component, we’ll need to call the `Register` action which will receive the form data.
```js
<script>
import { mapActions } from "vuex";
export default {
  name: "Register",
  components: {},
  data() {
    return {
      form: {
        username: "",
        full_name: "",
        password: "",
      },
      showError: false
    };
  },
  methods: {
    ...mapActions(["Register"]),
    async submit() {
      try {
        await this.Register(this.form);
        this.$router.push("/posts");
        this.showError = false
      } catch (error) {
        this.showError = true
      }
    },
  },
};
</script>
```
- We start by importing `mapActions` from Vuex, what this does is to import actions from our store to the component. This allows us to call the action from the component.

- `data()` contains the local state value that will be used in this component, we have a form object that contains `username`, `full_name` and `password`, with their initial values set to an empty string. We also have `showError` which is a `boolean`, to be used to either show an error or not.

- In the methods we import the Register action using the `Mapactions` into the component, so the `Register` action can be called with `this.Register`.

- We have a submit method this calls the `Register` action which we have access to using `this.Register`, sending it `this.form`. If no error is encountered we make use of `this.$router` to send the user to the `login page`. Else we set showError to true.

- Having done that, we can include some styling.
```js
<style scoped>
* {
  box-sizing: border-box;
}
label {
  padding: 12px 12px 12px 0;
  display: inline-block;
}
button[type=submit] {
  background-color: #4CAF50;
  color: white;
  padding: 12px 20px;
  cursor: pointer;
  border-radius:30px;
}
button[type=submit]:hover {
  background-color: #45a049;
}
input {
  margin: 5px;
  box-shadow:0 0 15px 4px rgba(0,0,0,0.06);
  padding:10px;
  border-radius:30px;
}
#error {
  color: red;
}
</style>
```
##### 1.2.2.3. Login.vue
- Our `LogIn page` is where registered users, will enter their `username` and `password` to get authenticated by the API and logged into our site.
```js
// src/views/Login.vue
<template>
  <div class="login">
    <div>
      <form @submit.prevent="submit">
        <div>
          <label for="username">Username:</label>
          <input type="text" name="username" v-model="form.username" />
        </div>
        <div>
          <label for="password">Password:</label>
          <input type="password" name="password" v-model="form.password" />
        </div>
        <button type="submit">Submit</button>
      </form>
      <p v-if="showError" id="error">Username or Password is incorrect</p>
    </div>
  </div>
</template>
```
- Now we’ll have to pass our form data to the action that sends the request and then push them to the secure page Posts
```js
<script>
import { mapActions } from "vuex";
export default {
  name: "Login",
  components: {},
  data() {
    return {
      form: {
        username: "",
        password: "",
      },
      showError: false
    };
  },
  methods: {
    ...mapActions(["LogIn"]),
    async submit() {
      const User = new FormData();
      User.append("username", this.form.username);
      User.append("password", this.form.password);
      try {
          await this.LogIn(User);
          this.$router.push("/posts");
          this.showError = false
      } catch (error) {
        this.showError = true
      }
    },
  },
};
</script>
```
- We import `Mapactions` and use it in importing the `LogIn` action into the component, which will be used in our submit function.

- After the `Login` action, the user is redirected to the `/posts` page. In case of an error, the error is caught and `ShowError` is set to true.

- Now, some styling:
```js
<style scoped>
* {
  box-sizing: border-box;
}
label {
  padding: 12px 12px 12px 0;
  display: inline-block;
}
button[type=submit] {
  background-color: #4CAF50;
  color: white;
  padding: 12px 20px;
  cursor: pointer;
  border-radius:30px;
}
button[type=submit]:hover {
  background-color: #45a049;
}
input {
  margin: 5px;
  box-shadow:0 0 15px 4px rgba(0,0,0,0.06);
  padding:10px;
  border-radius:30px;
}
#error {
  color: red;
}
</style>
```
##### 1.2.2.4. Posts.vue
- Our Posts page is the secured page that is only available to authenticated users. On this page, they get access to posts in the API’s database. This allows the users to have access to posts and also enables them to create posts to the API.
```js
// src/views/Posts.vue
<template>
  <div class="posts">
      <div v-if="User">
        <p>Hi {{User}}</p>
      </div>
      <div>
          <form @submit.prevent="submit">
            <div>
              <label for="title">Title:</label>
              <input type="text" name="title" v-model="form.title">
            </div>
            <div>
              <textarea name="write_up" v-model="form.write_up" placeholder="Write up..."></textarea>
            </div>
            <button type="submit"> Submit</button>
          </form>
      </div>
      <div class="posts" v-if="Posts">
        <ul>
          <li v-for="post in Posts" :key="post.id">
            <div id="post-div">
              <p>{{post.title}}</p>
              <p>{{post.write_up}}</p>
              <p>Written By: {{post.author.username}}</p>
            </div>
          </li>
        </ul>
      </div>
      <div v-else>
        Oh no!!! We have no posts
      </div>
  </div>
</template>
```
- In the above code, we have a form for the user to be able to create new posts. Submitting the form should cause the post to be sent to the API — we’ll add the method that does that shortly. We also have a section that displays posts obtained from the API (in case the user has any). If the user does not have any posts, we simply display a message that there are no posts.

- The `StateUser` and `StatePosts` getters are mapped i.e imported using `mapGetters` into `Posts.vue` and then they can be called in the template.
```js
<script>
import { mapGetters, mapActions } from "vuex";
export default {
  name: 'Posts',
  components: {
    
  },
  data() {
    return {
      form: {
        title: '',
        write_up: '',
      }
    };
  },
  created: function () {
    // a function to call getposts action
    this.GetPosts()
  },
  computed: {
    ...mapGetters({Posts: "StatePosts", User: "StateUser"}),
  },
  methods: {
    ...mapActions(["CreatePost", "GetPosts"]),
    async submit() {
      try {
        await this.CreatePost(this.form);
      } catch (error) {
        throw "Sorry you can't make a post now!"
      }
    },  
  }
};
</script>
```
- We have an initial state for form, which is an object which has title and write_up as its keys and the values are set to an empty string. These values will change to whatever the user enters into the form in the template section of our component.

- When the user submits the post, we call the `this.CreatePost` which receives the form object.

- As you can see in the created lifecycle, we have `this.GetPosts` to fetch posts when the component is created.

- Some styling,
```js
<style scoped>
* {
  box-sizing: border-box;
}
label {
  padding: 12px 12px 12px 0;
  display: inline-block;
}
button[type=submit] {
  background-color: #4CAF50;
  color: white;
  padding: 12px 20px;
  cursor: pointer;
  border-radius:30px;
  margin: 10px;
}
button[type=submit]:hover {
  background-color: #45a049;
}
input {
  width:60%;
  margin: 15px;
  border: 0;
  box-shadow:0 0 15px 4px rgba(0,0,0,0.06);
  padding:10px;
  border-radius:30px;
}
textarea {
  width:75%;
  resize: vertical;
  padding:15px;
  border-radius:15px;
  border:0;
  box-shadow:0 0 15px 4px rgba(0,0,0,0.06);
  height:150px;
  margin: 15px;
}
ul {
  list-style: none;
}
#post-div {
  border: 3px solid #000;
  width: 500px;
  margin: auto;
  margin-bottom: 5px;;
}
</style>
```
## 2. Defining Routes
- In our `router/index.js` file, import our views and define routes for each of them
```js
// src/router/index.js

import Vue from 'vue'
import VueRouter from 'vue-router'
import store from '../store';
import Home from '../views/Home.vue'
import Register from '../views/Register'
import Login from '../views/Login'
import Posts from '../views/Posts'

Vue.use(VueRouter)
const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/register',
    name: "Register",
    component: Register,
    meta: { guest: true },
  },
  {
    path: '/login',
    name: "Login",
    component: Login,
    meta: { guest: true },
  },
  {
    path: '/posts',
    name: Posts,
    component: Posts,
    meta: {requiresAuth: true},
  }
]
const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})

export default router
```
- We have defined routes for our `Home`, `Register`, `Login` and `Posts` pages. We have also added a meta property to the `Register` and `Login` routes. This `meta property is used to define the routes that are only accessible to guests`.

## 3. Handling Users
### 3.1 Unauthorized Users
- If you noticed in defining our posts routes we added a meta key to indicate that the user must be authenticated, now we need to have a `router.BeforeEach` navigation guard that checks if a route has the `meta: {requiresAuth: true}` key. If a route has the meta key, it checks the store for a token; if present, it redirects them to the login route.
```js
const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})
router.beforeEach((to, from, next) => {
  if(to.matched.some(record => record.meta.requiresAuth)) {
    if (store.getters.isAuthenticated) {
      next()
      return
    }
    next('/login')
  } else {
    next()
  }
});

export default router;
```
### 3.2 Authorized Users
- We also have a meta on the `/register` and `/login` routes. The `meta: {guest: true}` stops users that are logged in from accessing the routes with the guest meta.
```js
router.beforeEach((to, from, next) => {
  if (to.matched.some((record) => record.meta.guest)) {
    if (store.getters.isAuthenticated) {
      next("/posts");
      return;
    }
    next();
  } else {
    next();
  }
});

export default router;
```
> Note: In the end, your file should be like this:
```js
// src/router/index.js

import Vue from "vue";
import VueRouter from "vue-router";
import store from "../store";
import Home from "../views/Home.vue";
import Register from "../views/Register";
import Login from "../views/Login";
import Posts from "../views/Posts";

Vue.use(VueRouter);

const routes = [
  {
    path: "/",
    name: "Home",
    component: Home,
  },
  {
    path: "/register",
    name: "Register",
    component: Register,
    meta: { guest: true },
  },
  {
    path: "/login",
    name: "Login",
    component: Login,
    meta: { guest: true },
  },
  {
    path: "/posts",
    name: "Posts",
    component: Posts,
    meta: { requiresAuth: true },
  },
];

const router = new VueRouter({
  mode: "history",
  base: process.env.BASE_URL,
  routes,
});

router.beforeEach((to, from, next) => {
  if (to.matched.some((record) => record.meta.requiresAuth)) {
    if (store.getters.isAuthenticated) {
      next();
      return;
    }
    next("/login");
  } else {
    next();
  }
});

router.beforeEach((to, from, next) => {
  if (to.matched.some((record) => record.meta.guest)) {
    if (store.getters.isAuthenticated) {
      next("/posts");
      return;
    }
    next();
  } else {
    next();
  }
});

export default router;
```

## 4. Handling Expired Token (Forbidden Requests)

- Our API is set to expire tokens after 30 minutes, now if we try accessing the `posts page `after 30 minutes, we get a 401 error, which means we have to log in again, so we will set an `interceptor` that reads if we get a 401 error then it redirects us back to the login page.

- Add the snippet below after the Axios default URL declaration in the `main.js` file.
```js
axios.interceptors.response.use(undefined, function (error) {
  if (error) {
    const originalRequest = error.config;
    if (error.response.status === 401 && !originalRequest._retry) {
  
        originalRequest._retry = true;
        store.dispatch('LogOut')
        return router.push('/login')
    }
  }
})
```

> RESOURCES :- For mpre information on the above topics, you can check out the following resources:
- [“Handling Cookies With Axios,” Aditya Srivastava, Medium](https://medium.com/@adityasrivast/handling-cookies-with-axios-872790241a9b)
- [“Creating An Authentication Navigation Guard In Vue,” Laurie Barth, Ten Mile Square Blog](https://tenmilesquare.com/resources/software-development/creating-an-authentication-navigation-guard-in-vue/)
- [“Getting Started With Vuex,” Official Guide](https://vuex.vuejs.org/guide/)
- [“Vue.js JWT Authentication With Vuex And Vue Router,” BezKoder](https://www.bezkoder.com/jwt-vue-vuex-authentication/)