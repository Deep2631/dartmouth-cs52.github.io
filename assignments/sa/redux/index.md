---
layout: page
title: ""
published: true
---

[![](img/redux.png)](http://redux.js.org/)


## Overview

React + Routing + [Redux](http://redux.js.org/)

For this workshop we're just going to add redux to our starter pack!

Redux is a fancy state management system that we'll be using to build our blog platform.  It will allow us to simplify our code and have a global state in our app with fewer callbacks!


## Let's Start

Clone your react+router starter or the [js-react-router-starter](https://github.com/dartmouth-cs52/js-react-starter), you should know the drill by now.  Redux does add some complexity to your project which is not always needed. So I would recommend either creating a branch for your starter code for redux. `git checkbout -b with_redux`. Or create a new repository. Your choice, but I do strongly recommend getting all your repos organized and named correctly. 😎


### Github Clone and Switch Reminder

If you need a reminder for how to clone and change repos here's the easy way.

```bash
# create new repo on github.com and use url for remote add line
git clone git@github.com:dartmouth-cs52/js-react-starter.git    yournewprojectname
cd yournewprojectname
git remote rename origin starter
git remote add origin git@github.com:yourusername/yournewprojectname.git
```

There are fancier ways to do this if you want to retain the full repository tags and all branches, but in most cases you do not, you just want the starter code.


## Redux



🚀 Install redux

```bash
npm install --save redux react-redux redux-thunk
```

These are the packages we will be using. Redux is a general state management paradigm that does not necessarily need to be tied to React. But together they are an unstoppable force for clean webdevelopment — so we will use another module `react-redux` that connects the two.   We won't use `redux-thunk` today but will soon — it allows us to work better with asynchronous server calls.


## Chrome React and Redux Devtools

These Chrome extensions will greatly assist us in debugging our React+Redux apps.

### React Dev Tools:

If you don't have React Devtools installed yet, please do so now [React Devtools in Chrome Store](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)

More docs [here](https://github.com/facebook/react-devtools).


### Redux Dev Tools

And [redux-devtools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd) will give us time travel! No joke.

More docs [here](https://github.com/zalmoxisus/redux-devtools-extension).




## What is Redux

> Redux is a predictable state container for JavaScript apps.

Redux has 3 main concepts that we will tackle. Each concept is related to a particular structure within Redux.


- **Store**:  the entire state of your app is stored in an object tree inside a single store.
- **Actions**:  the only way to change the state tree 🌳is to emit an action — an object that describes that happened.
- **Reducers**:  specify the state transformation based on the actions triggered.




### Actions

Remember how our state is immutable (read-only)?   Actions are the way to trigger new states.  In plain React we would run `setState` based on events passed up through callbacks. With Redux we can still use local component state for certain things, but whenever we need a far reaching state change we will use actions instead.

An action is an object with at least the property `type: string`.  The convention is to make these all upper case. They can also have payloads of other associated data.  

For instance an action might be:

```javascript
{
  type: VIDEO_SELECTED,
  video: videoID
}
```

Actions ensure that no views or callbacks will write to the state. An action does not alter state itself,  it expresses an intent for the state to change. All actions are processed by associated reducers in a centralized manner.  Remember, actions are just plain old objects, they can be logged and stored, and replayed even!



### Reducers


When an action is triggered, reducers kick in to do the necessary state changes.  Reducers are functions that take in previous state + the action triggered, and return the next state.  This is similar to what we did with `setState` where we would take existing state and set it to new state.  Your app may have multiple reducers each responsible for a particular part of the state tree.  

An example reducer that responds to the above VIDEO_SELECTED action might look something like this:

```javascript
export default function(state = null, action) {
    switch (action.type) {
        case 'VIDEO_SELECTED':
            return { selectedVideo: action.video };
        default:
            return state;
    }
}
```

Note the switch statement.  Actions are global and every reducer gets called with every action. It is up to the reducer to act on the action or not.

### Store

Finally there is the Redux Store.  A redux store is a way to store the global state for your app.  So far for the React apps that we have built we have had local component state if a few of our smart components, and often have had a top level state inside of our App component.  Having your app state (minus in a single state tree has lots of positive benefits.  The store is initialized in your top level `index.js` file.



## Code

Let's wire this all together, it may seem a bit tricky at first but we'll work though it!


### File Structure

🚀 Create a file structure to store our actions and reducers.

```bash
mkdir src/actions src/reducers src/components
```


We're going to create an extensible action and reducer structure so there might be a little bit extra boilerplate, but not to worry it'll all make sense.


### Actions

Best way to trigger actions in a connected component is via triggering an ActionCreator.  Action creators are simply functions that return an action object. Remember an action object is an arbitrary object that that has at a minimum a type property `{ type: 'SOME_ACTION' }`.

🚀 Let's create an `actions/index.js` file and give it some very basic actions.

```javascript

// keys for actiontypes
export const ActionTypes = {
  INCREMENT: 'INCREMENT',
  DECREMENT: 'DECREMENT',
};


export function increment() {
  return {
    type: ActionTypes.INCREMENT,
    payload: null,
  };
}

export function decrement() {
  return {
    type: ActionTypes.DECREMENT,
    payload: null,
  };
}
```

1. we have `ActionType` an object with keys. This is simply so we have constants so we don't have to type in string identifiers.
2. `increment` just returns an INCREMENT action
3. `decrement` just returns a DECREMENT action


### Reducers

🚀 Let's create a `reducers/index.js` file.   

With ES6 if you have a directory with multiple files and an `index.js` file you can import it by the directory name:  `import ./reducers`.

We are going to split up our reducers into a top level root reducer and smaller reducers that each act on a part of the state tree. The root reducer will use `combineReducers` to combine multiple reducers together into our redux state.


```javascript
import { combineReducers } from 'redux';

import CountReducer from './count-reducer';

const rootReducer = combineReducers({
  count: CountReducer,
});

export default rootReducer;
```

Here, we have created our `rootReducer` which combines all other potential reduces into one main state object with several top level keys.


🚀 Now create `reducers/count-reducer.js`

```javascript
import { ActionTypes } from '../actions';

const CountReducer = (state = 0, action) => {
  switch (action.type) {
    case ActionTypes.INCREMENT:
      return state + 1;
    case ActionTypes.DECREMENT:
      return state - 1;
    default:
      return state;
  }
};

export default CountReducer;
```

Our `CountReducer` gets called with actions.  When it is called with the 2 actions it knows something about it will return new state.  What will our state object look like overall? Simply:

```javascript
{
  count: 0
}
```


### Initialize Store

The Redux store is the main class responsible for bringing it all together.  The store is initialized with the reducers and knows when and how to call them. The store contains our state data and contains the dispatch functionality that we need to trigger actions.   It is what holds it all together.


Just a tiny bit more boilerplate, I promise.

🚀 Add the following to your root/main `index.js` file.

```javascript
// at the top
import { Provider } from 'react-redux';
import { createStore, applyMiddleware, compose } from 'redux';
import reducers from './reducers';


// this creates the store with the reducers, and does some other stuff to initialize devtools
const store = createStore(reducers, {}, compose(
  applyMiddleware(),
  window.devToolsExtension ? window.devToolsExtension() : f => f
));


//replace your ReactDOM render with the following
// note this uses the Router stuff from last week
ReactDOM.render(
  <Provider store={store}>
    <Router history={browserHistory} routes={routes} />
  </Provider>
  , document.getElementById('main'));

```



## Connected components

Now we have the basics done, we should create some connected components.  A connected component is one that is literally connected to the Redux store.  The way we do this is by wrapping a component in a higher-order function that returns a new component that has access to Redux.

🚀 First, lets create a separate directory for connected components.  Let's call our connected components `containers` as this is what the Redux creator [wants us to do](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.yobej4pio).


Now we should make some components that make use of our Redux setup!  These will be toy examples, and don't particularly need Redux.  But they'll demonstrate the various wirings needed for setting up a much more complicated app.


### Counter

🚀 Let's create a component for displaying our state. Create a new file `containers/counter.js`

```javascript
import React from 'react';
import { connect } from 'react-redux';

// this can be dumb or smart component - connect works with either
const Counter = (props) => {
  return (
    <div>
      Current Count: {props.count}
    </div>
  );
};

// connects particular parts of redux state to this components props
const mapStateToProps = (state) => (
  {
    count: state.count,
  }
);

// react-redux glue -- outputs Container that know state in props
export default connect(mapStateToProps, null)(Counter);
```


Some things to note about the this component:

1. `mapStateToProps`.  This is a simple function that takes the full global state as an argument and maps some part or parts of it this components props.  The mapping is a simple object with key: value pairs. As is everything!
1. `connect`.  This is the magic that connects the component to Redux.  It takes 2 arguments: mapStateToProps and mapDispatchToProps and returns a new version of the component.
1. `props.count`.  Check it, passed into props from Redux.


🚀 Add `<Counter />` to the render method of one of the components that are currently displaying like *Welcome*


You should now see it rendering with a value of 0.  This value is coming from the default value in our reducer!


### Controls

🚀 Let's create one more component. Oof!  This time it'll just be a couple of buttons to call some actions.  Make one last file: `containers/controls.js`.

```javascript
import React from 'react';
import { connect } from 'react-redux';

import { increment, decrement } from '../actions';

const Controls = (props) => {
  return (
    <div>
      <button onClick={props.increment} className="btn btn-info">+</button>
      <button onClick={props.decrement} className="btn btn-warning">-</button>
    </div>
  );
};


// react-redux glue -- outputs Container that knows how to call actions
export default connect(null, { increment, decrement })(Controls);
```


Some things to note about the this component:

1. `import { increment, decrement } from '../actions';`:  we're importing the 2 ActionCreators we created earlier.
1. `connect`: note the second argument to connect, this can either be a function mapDispatchToProps, or a simple dictionary mapping.  This makes sure that connected versions of the increment and decrement actioncreators are passed into props for us.
1. `props.increment/decrement`:  Here's the confusing part. You must call the connected version of the actionCreator methods in props rather than calling them directly. If you just call the methods directly nothing will happen (as Redux doesn't know about them. However the versions of the versions that are passed into `props` will get passed down into the reducer.



🚀 Add `<Controls />` to the render method of one of the components that are currently displaying like *Welcome*



Ok, so that may seem like a lot of code for a toy examples, but note that these two components don't need to have a common parent, and you don't to pass around functions in callbacks.  In a simple example like this it might not make much of a difference,  we could have put the counter into the local component state of the parent. However if you had multiple levels of components the callback tracking would get difficult.


We'll do much more React+Redux so if this is confusing don't worry!  


## Debugging


Last thing!  Try opening up the Redux Chrome DevTools and play with the timeline to move the state forward and back.



## Resources

* [http://redux.js.org/index.html](http://redux.js.org/index.html)
* [https://css-tricks.com/learning-react-redux/](https://css-tricks.com/learning-react-redux/)
