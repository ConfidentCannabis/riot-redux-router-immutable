riot-redux-router-immutable
-----------------

Dead simple integration between the riot js router and redux with ImmutableJS

Combine Riot's router, redux, and immutable so you can easily dispatch route
actions to simultaneously update your store and the browser url. Does very
little else - this readme is longer than all the code combined.

Features:
- `route` action to dispatch in order to change the route
- middleware to handle riot's router
- optional reducer to update the store with the current url


## Install:

`npm install riot-redux-router-immutable`


## Usage:

To start using the riot router with redux, you must add the riotRouterMiddleware
to your store (and the router reducer or your own to track the current url).

The middleware listens for ROUTER_GO_ACTIONs and uses their payload to
trigger the riot router's `route` function instead of continuing on to the
reducers. When the route changes, the middleware dispatches a
ROUTER_ROUTE_CHANGED action, which is handled by the reducer to update the
current url in the store.


## Example

Example initialization - add the router.middleware to redux
```
var riot = require('riot')
var redux = require('redux')
var router = require('riot-redux-router-immutable')

var rootReducer = require('./app/reducer')

var createStoreWithMiddleware = redux.compose(
  redux.applyMiddleware(router.middleware),
  // your other redux middleware here...
)(redux.createStore)

var reduxStore = createStoreWithMiddleware(rootReducer)
```


Example top level reducer -- `router.current_url` will be set on route change
```
var redux = require('redux')
var router = require('riot-redux-router-immutable')


module.exports = redux.combineReducers({
  router: router.reducer,
  // your other reducers here
})
```


Example usage -- 'custom-tag.tag'
```
<custom-tag>

  <a onclick={selectPage} url='page1'>Page 1</a>
  <a onclick={selectPage} url='page2'>Page 2</a>

  <div class={hidden: state.router.current_url !== 'page1'}>Page 1 FTW</div>
  <div class={hidden: state.router.current_url !== 'page2'}>Page 2 is Great</div>

  <div>Current Page: {state.router.current_url}</div>

  <style scoped>
    .hidden { display: none; }
  </style scoped>

  <script>
    var actions = require('riot-redux-router').actions
    var store = this.opts.store

    store.subscribe(function() {
      // NOTICE the toJS() - turn the immutable map into POJO
      this.state = store.getState().toJS()
      this.update()
    }.bind(this))

    selectPage(element) {
      store.dispatch(
        actions.route(
          element.target.getAttribute('url')
        )
      )
    }
  </script>
</custom-tag>
```