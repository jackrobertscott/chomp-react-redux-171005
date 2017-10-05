# Chomp

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Emoji_u1f40a.svg/600px-Emoji_u1f40a.svg.png" width="200px" align="right" />

### Take the bite out of React and Redux

React is an unopinionated framework. This means that when it comes to building apps, a lot of time is spent deciding on the best way to approach developing specific features e.g. should component files be named `HotdogList.js`, `hotdogList.js`, `hotdog.list.js` etc. This results in hours of frustration, sloppy code, and a whole bunch of wasted time and *many* sleepless nights.

Ruby on Rails adopted the doctrine of **convention over configuration**. This basically means that they have strict guidelines on how features may be developed. They show exactly how variables need to be defined, how validations should be made, what the folder structure must be, and everything inbetween. As such, because developers don't need to think about these trivial development decisions, they have been able to:

1. Speed up development time
2. Reduced barriers to entry
3. Free up more time up for playing RuneScape

I'm not saying we should adopt the same conventions as Rails but we should definitely adopt the doctrine. Therefore, without futher adue, here is Chomp; the convention over configuration guidelines for React and Redux applications.

## File Structure

Organise your files by feature. This makes it a lot easier to:

1. Find the files you need
2. Name your files
3. Manage folder sizes on large scale applications
4. Share specific modules between multiple applications

The folder structure rules are as such:

1. Root folders should be the singular name of the feature
2. All files that do not fit in a specific feature folder go in the `shared` folder
3. Helper files should be formatted as `<feature>.<type>.js` e.g. `hotdog.reducer.js`
4. There should be a `components` and `containers` set of folders in each feature
5. Container (smart) file names should be formatted as `<Feature><Action>.js`
6. Component (dumb) file names should be formatted as `<Description><Type>.js`

```
src
+-- hotdog
    +-- components // do not include feature name in component name as should not use outside feature
        +-- GoodButton.js
    +-- containers // format of file names should be <Feature><Action>
        +-- HotdogList.js
        +-- HotdogUpdate.js
    +-- hotdog.reducer.js
    +-- hotdog.service.js
+-- drink
    +-- components
        +-- ListItem.js
        +-- CupWrap.js
        +-- EditForm.js
    +-- containers
        +-- DrinkList.js
        +-- DrinkCreate.js
        +-- DrinkUpdate.js
+-- shared // where all files that don't belong to a specific feature should go
    +-- components
        +-- CommonButton.js
    +-- containers
        +-- App.js
    +-- utils.helper.js
```

## Reducers

This is mostly inpsired by the [Ducks](https://github.com/erikras/ducks-modular-redux) proposal.

As applications grow, it becomes increasingly difficult to ensure good code quality and maintenance. As such, we recommend using helper libraries such as [redux-actions](https://github.com/reduxactions/redux-actions) to encourage better code quality and reduce unneeded flexibility. It is also imporant to keep the layout of the file structured and easy to read so that making changes is simple.

```js
import { createAction, handleActions } from 'redux-actions';
import { apiGetHotdogs, apiCreateHotdog } from './hotdog.service';

/**
 * Initial state
 */
const initialState = {
  hotdogs: [],
  current: null,
  loading: false,
  success: false,
  problem: null, // use "problem" instead of "error" as error causes issues when passed as prop
};

/**
 * Constants
 */
export const HOTDOGS_LOADING = 'HOTDOGS_LOADING';
export const HOTDOGS_GET = 'HOTDOGS_GET';
export const HOTDOG_CREATE = 'HOTDOG_CREATE';

/**
 * Actions
 * 
 * These describe what happened.
 */
export const loadingHotdogs = createAction(HOTDOGS_LOADING);
export const getHotdogs = createAction(HOTDOGS_GET, apiGetHotdogs);
export const createHotdog = createAction(HOTDOG_CREATE, apiCreateHotdog);

/**
 * Thunks
 * 
 * The return value of the inner function should be a promise. The dispatch function
 * returns the value of the function from within it. This allows us to chain dispatch functions.
 */
export const attemptGetHotdogs = () => (dispatch, getState) => {
  dispatch(loadingHotdogs());
  const { token } = getState().user.auth;
  return dispatch(getHotdogs(token));
};
export const attemptCreateHotdog = () => (dispatch, getState) => {
  dispatch(loadingHotdogs());
  const state = getState();
  const { token } = state.user.auth;
  const hotdog = { ...state.form.hotdog.values }; // get values from 'redux-form' form
  return dispatch(createHotdog(token, hotdog));
};

/**
 * Reducer
 * 
 * All reducer functions should be pure. They describe how the state is mutated.
 */
export default handleActions({

  [HOTDOGS_LOADING]: (state) => ({
    ...state,
    loading: true,
    problem: null,
    success: false,
  }),

  [HOTDOGS_GET]: (state, { payload, error }) => ({
    ...state,
    loading: false,
    hotdogs: error ? [] : payload,
    problem: error ? payload : null,
  }),

  [HOTDOG_CREATE]: (state, { payload, error }) => ({
    ...state,
    loading: false,
    problem: error ? payload : null,
    success: !error,
  }),

}, initialState);
```

In the above example, we are also using the [redux-thunk](https://github.com/gaearon/redux-thunk) and [redux-promise](https://github.com/acdlite/redux-promise) middleware. This allows us to handle the execution of multiple action and access service function with ease.

## Services

Services are standard implementations functions that request data from the server and return the data in a promise.

```js
import config from '../config';
import { handleResponse } from '../shared/util.helper'; // this extracts the json content from the response

export const apiGetHotdogs = (token) => fetch(`${config.endpoint}/hotdogs`, {
  method: 'GET',
  headers: {
    'Authorization': token,
  },
}).then(handleResponse);

export const apiCreateHotdog = (token, hotdog) => fetch(`${config.endpoint}/hotdogs`, {
  method: 'POST',
  headers: {
    'Authorization': token,
  },
  body: JSON.stringify(hotdog),
}).then(handleResponse);
```

## Containers vs Components

## Routing

## Forms
