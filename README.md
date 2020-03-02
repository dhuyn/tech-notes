# tech-notes

## Redux vs Context API
https://daveceddia.com/context-api-vs-redux/

Context API released March 29, 2018 as part of React 16.3

### Prop Drilling
https://twitter.com/dan_abramov/status/1021849622002782208?s=20
> The picture makes it look like prop drilling is discouraged. In fact it’s the opposite: usually you *want* prop drilling, and we don't suggest context (or Redux) unless the complexity is worth it.

Common React pattern involving passing props from parent through levels of children. Intermediary children do not require the props.

PROS:
- Good pattern to start with for simple components.
- Recommended by Dan Abramov. This is a desired pattern unless additional complexity is worth it.

CONS:
- Deep drilling one prop becomes annoying.
- Deep drilling multiple props becomes even more annoying.
- Creates coupling for components that would otherwise be decoupled. Difficult to reuse or refactor.


### Children Pattern
https://twitter.com/dan_abramov/status/1021850499618955272

https://reactjs.org/docs/context.html#before-you-use-context

```jsx
// Body needs a sidebar and content, but written this way,
// they can be ANYTHING
const Body = ({ sidebar, content }) => (
  <div className="body">
    <Sidebar>{sidebar}</Sidebar>
    {content}
  </div>
);
```
```jsx
  // Render the App component
  render() {
    const { user } = this.state;

    return (
      <div className="app">
        <Nav>
          <UserAvatar user={user} size="small" />
        </Nav>
        <Body
          sidebar={<UserStats user={user} />}
          content={<Content />}
        />
      </div>
    );
  }
```

>Note how by making <Nav> and <Body> accept any elements as children, I removed the need to pass the "user" prop down through them.

(See React docs on [Containment](https://reactjs.org/docs/composition-vs-inheritance.html#containment))
>This inversion of control can make your code cleaner in many cases by reducing the amount of props you need to pass through your application and giving more control to the root components. However, this isn’t the right choice in every case: moving more complexity higher in the tree makes those higher-level components more complicated and forces the lower-level components to be more flexible than you may want.


### Redux
https://daveceddia.com/redux-tutorial/

#### Redux without React
```jsx
import { createStore } from "redux";
```  
`createStore()`  
Redux has one store. Initialize and return it with the `createStore(reducer)` function.

`reducer()`  
`createStore()` must take in a reducer. Reducer is a user-defined function that takes in a `state` and `action` and returns a new state.

`action`  
`action`s are JS objects that have a `type` to describe the action.

`dispatch`  
`store.dispatch(action)` is used to execute an action on a state. It does this by directly calling the `reducer()` function.

#### Redux with React
```jsx
import { connect, Provider } from "react-redux";
```  
`connect()`  
Connect a React Component to Redux state with this HOC by calling `export default connect(mapStateToProps)(Component)`. It pulls out the entire Redux state and passes that through to `mapStateToProps`. Then it returns a function that takes in a React Component which is now wrapped with the modified Redux state.

`mapStateToProps()`  
This is a user-defined function that turns the global Redux state into props that the wrapped function requires. Obviously, you should only get props that are truly needed from the Redux state.

`dispatch()`  
`connect()` not only passes Redux state to the wrapped component, it also passes the store's `dispatch()` function. In the wrapped component, import the defined `action`s and call `dispatch()` on them in order to update the Redux state.

```jsx
import React from "react";
import ReactDOM from "react-dom";

// We need createStore, connect, and Provider:
import { createStore } from "redux";
import { connect, Provider } from "react-redux";

// Create a reducer with an empty initial state
const initialState = {};
function reducer(state = initialState, action) {
  switch (action.type) {
    // Respond to the SET_USER action and update
    // the state accordingly
    case "SET_USER":
      return {
        ...state,
        user: action.user
      };
    default:
      return state;
  }
}

// Create the store with the reducer
const store = createStore(reducer);

// Dispatch an action to set the user
// (since initial state is empty)
store.dispatch({
  type: "SET_USER",
  user: {
    avatar: "https://www.gravatar.com/avatar/5c3dd2d257ff0e14dbd2583485dbd44b",
    name: "Dave",
    followers: 1234,
    following: 123
  }
});

// This mapStateToProps function extracts a single
// key from state (user) and passes it as the `user` prop
const mapStateToProps = state => ({
  user: state.user
});

// connect() UserAvatar so it receives the `user` directly,
// without having to receive it from a component above

// could also split this up into 2 variables:
//   const UserAvatarAtom = ({ user, size }) => ( ... )
//   const UserAvatar = connect(mapStateToProps)(UserAvatarAtom);
const UserAvatar = connect(mapStateToProps)(({ user, size }) => (
  <img
    className={`user-avatar ${size || ""}`}
    alt="user avatar"
    src={user.avatar}
  />
));

// connect() UserStats so it receives the `user` directly,
// without having to receive it from a component above
// (both use the same mapStateToProps function)
const UserStats = connect(mapStateToProps)(({ user }) => (
  <div className="user-stats">
    <div>
      <UserAvatar />
      {user.name}
    </div>
    <div className="stats">
      <div>{user.followers} Followers</div>
      <div>Following {user.following}</div>
    </div>
  </div>
));

// Nav doesn't need to know about `user` anymore
const Nav = () => (
  <div className="nav">
    <UserAvatar size="small" />
  </div>
);

const Content = () => (
  <div className="content">main content here</div>
);

// Sidebar doesn't need to know about `user` anymore
const Sidebar = () => (
  <div className="sidebar">
    <UserStats />
  </div>
);

// Body doesn't need to know about `user` anymore
const Body = () => (
  <div className="body">
    <Sidebar />
    <Content />
  </div>
);

// App doesn't hold state anymore, so it can be
// a stateless function
const App = () => (
  <div className="app">
    <Nav />
    <Body />
  </div>
);

// Wrap the whole app in Provider so that connect()
// has access to the store
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.querySelector("#root")
);
```

#### Optional Redux with React
`action creator function`  
Action creator is a function that returns an `action` object. It is suggested that this can be cleaner in larger code bases, but I think the real use is to be able to use `mapDispatchToProps`. Instead of using `this.props.dispatch({type: ACTION})`, use `this.props.dispatch(action())`

`mapDispatchToProps`  
A user-defined object whose keys and values are the same name as the user-defined `action creator function`s. By calling `export default connect(mapStateToProps, mapDispatchToProps)(Component)`, the function props are passed to the wrapped component. This allows you to avoid using `this.props.dispatch()` and instead use `this.props.action()`.

#### Redux Middleware
```jsx
import thunk from 'redux-thunk';
import { createStore, applyMiddleware } from 'redux';

function reducer(state, action) {
  // ...
}

const store = createStore(
  reducer,
  applyMiddleware(thunk)
);
```
`thunks`  
Uncommon name for the function that is returned from a function. Used in an `action creator` to return a function that executes an API call or other statements instead of the `action` object.

Allows an `action creator` to return a function instead of an action. You'll want to do this in order to delay the action or to return it conditionally.

