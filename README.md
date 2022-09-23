# vuex

> A Vue.js project

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).

##Implementing modules
Add a module by adding an object within the modules property on the store.

To keep the blog seperate, create a blog.js file to export a default object.
Add a state object to keep the blog posts.
Cut and paste the blog posts from the ListPosts component:

export default {
    state: {
        posts: [
            { id: 1, title: 'Vue.js is Awesome' },
            { id: 2, title: 'Vuex is Pretty Nice Too' },
            { id: 3, title: 'New Version of Vue.js Released' }
        ]
    }
};

Now register it in the store.
In main.js import it: 

import BlogModule from './blog/store/blog';

Then add a modules object to the store.
Pass in a name and the configuration object:

modules: {
    blog: BlogModule
}

Then to use it in the ListPosts component add a computed property to the ListPosts component.
Add a posts property then return the posts.
The posts are stored in the blog module we need to access it through that:

computed: {
    posts() {
        return this.$store.state.blog.posts;
    }
}

Add an empty object as the user object for now.
Import it into main.js and add it to the store modules:

modules: {
    blog: BlogModule,
    user: UserModule
}

#Accessing root state from getters
Move the user related stuff over to the module.
Cut and paste the isLoggedIn getter to the user.js file.
Modify the args to have state, getters, and rootState.
The state object is the local state for the user module.
The rootState is the state of the root store defined in the main.js file.
Return the rootState isLoggedIn method:

getters: {
    isLoggedIn(state, getters, rootState, rootGetters) {
        return rootState.isLoggedIn;
    }
}

##Accessing root state from actions
In the User module add a login and logout action.
Add an actions object.
Add a login action and pass in the state, commit, and rootState properties.
Invoke an alert to show the state of the rootState isLoggedIn property.
Add a logout action:

actions: {
    login({ state, commit, rootState }) {
        alert("isLoggedIn is: " + rootState.isLoggedIn);
    },
    logout({ state, commit, rootState }) {
  
    }
}

Then use the mapActions helper to tell the App component to use these actions.
Import the mapActions helper in the App component: 

import { mapActions } from 'vuex';

Then use the helper with the spread operator:

methods: {
  ...mapActions([
    'login',
    'logout'
  ])
}

Now an alert is displayed for the isLoggedIn property when the Log In button is clicked.
This shows we are accessing a rootState property within a modules action.

##Adding mutations to modules
In the user module add a login and logout mutation.
So we need to set the state which is in the store.
We cannot access the rootState within mutations.
So we need to add the state to the module.

In the main.js file cut the isLoggedIn state out and paste it into the user module:

state: {
    isLoggedIn: false
}

So then we can easily set the state:

mutations: {
    login(state) {
        state.isLoggedIn = true;
    },
    logout(state) {
        state.isLoggedIn = false;
    }
}

Now update the getter to use the local state:

getters: {
    isLoggedIn(state, getters, rootState ) {
        return state.isLoggedIn;
    }
}

Commit the mutations within the actions:

actions: {
    login({ state, commit, rootState }) {
        commit('login)
    },
    logout({ state, commit, rootState }) {
        commit('logout');
    }
}

Now we can login and logout again.

##Introduction to namespaces
Getters, actions, and mutations are added to the global namespace by default.
They help to avoid conflict between different modules that may have the same getters.

A module without namespace may access a getter with:

this.$store.getters.getPosts

A module with namespace:

this.$store.getters['user/isLoggedIn']

##Adding namespaces
Add a namespace property to the module object and set it to true.
So in both of the modules:

namespaced: true,

The name of each namespace will be blog and user.
We need to update the app now to access these names.

##Accessing namespaced types with helpers
Update the mapActions helper in the App component:

...mapActions([
    'user/login',
    'user/logout'
])

This is not ideal, the best way is to pass in the name of the namespace to be prefixed: 

methods: {
    ...mapActions('user', [
        'login',
        'logout'
    ])
}

Let's specify an object so the keys will be the method names within the module and the values will be the action names:

methods: {
    ...mapActions('user', {
        login: 'login',
        logout: 'logout'
    })
}

Then we need to do the same for the getter.
This time we will add the namespace within the key value pair:

...mapGetters({
    isLoggedIn: 'user/isLoggedIn'
})

We don't need to do this for the blog module as the state module refers to the local state and is not affected by namespaces.

##Accessing root getters from namespaced modules
When a namespace is added to a module, the getter argument is then a localised object - only including getters witin the module itself.
Any getters added to the store or other components are not part of the object.
The same is for dispatching actions and commiting mutations.

Let' simulate a user being banned from the app.
In main.js add a new state object with a isBanned property set to true:

state: {
    isBanned: false
}

Then add a isBanned getter:

getters: {
    isBanned(state) {
        return state.isBanned;
    }
}

Now we have a getter on the store and the user module.
To prove that the getters passed to the module getter are local getters, log the getters in user.js:

getters: {
    isLoggedIn(state, getters, rootState) {
        console.log(getters);
    }
}

We see there is no isBanned property available on the object because it only contains getters for the module itself.
To access the rootGetters add a 4th param called rootGetters:

getters: {
    isLoggedIn(state, getters, rootState, rootGetters) {
        return state.isLoggedIn;
    }
}

We won't apply it here instead we'll use it in the actions object.
So check if isBanned is not true and commit the login mutation.
Otherwise show an alert:

actions: {
    login({ state, commit, rootState, rootGetters }) {
        if (!rootGetters.isBanned) {
            commit('login', null, { root: true });
        } else {
            alert("Get outta here!");
        }
    }
}

We should see the alert as isBanned is set to true.

##Dispatching root actions and committing root mutations
Create a new mutation called login in main.js:

mutations: {
    login(state) {
        console.log("login (root)");
    }
}

Set the isBanned property to false.
Then in user.js log a string in the login mutation to see if it is invoked:

login(state) {
    console.log("login (user)");
    state.isLoggedIn = true;
}

Now in the login action commit the mutation to the store aka root mutation.
Pass an arg to the commit method.
1st arg is the name of the mutation and the 2nd arg is the payload.
1st arg we can pass an object with a root key with the value of true.
This means the root mutation will be invoked instead of the local one:

login({ state, commit, rootState, rootGetters }) {
    if (!rootGetters.isBanned) {
        commit('login', null, { root: true });
    } else {
        alert("Get outta here!");
    }
}

For dispatching actions it's the same.
When invokng a dispatch method pass an object as the 3rd arg with the root key and a value of true.