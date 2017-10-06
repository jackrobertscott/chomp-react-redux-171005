# Chomp

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Emoji_u1f40a.svg/600px-Emoji_u1f40a.svg.png" width="200px" align="right" />

### Convention over configuration approach to React and Redux

React is an unopinionated framework. This means that when it comes to building apps, a lot of time is spent deciding on the best way to approach developing specific features e.g. should component files be named `HotdogList.js`, `hotdogList.js`, `hotdog.list.js` etc. This results in hours of frustration, sloppy code, a whole bunch of wasted time and *many* sleepless nights.

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
    +-- containers
        +-- DrinkForm.js
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

This is mostly inspired by the [Ducks](https://github.com/erikras/ducks-modular-redux) proposal.

As applications grow, it becomes increasingly difficult to ensure good code quality and maintenance. As such, we recommend using helper libraries such as [redux-actions](https://github.com/reduxactions/redux-actions) to encourage better code quality and reduce unneeded flexibility. It is also imporant to keep the layout of the file structured and easy to read so that making changes is simple.

Shoulds:

1. Do name the file `<feature>.reducer.js` e.g. `hotdog.reducer.js` or `drink.reducer.js`
2. Do use the property `problem` instead of `error` for setting and error in the state
3. Do use descriptive constants e.g. `HOTDOGS_LOADING`
4. Do use the [redux-actions](https://github.com/reduxactions/redux-actions)
5. Do make sure your reducer functions and action creators are **pure**
6. Do use [redux-thunk](https://github.com/gaearon/redux-thunk) to make functions with multiple dispatch calls
7. Do prepend thunk functions with `attempt` e.g. `attemptCreateHotdog()`

Should **nots**:

1. Don't be sloppy, keep the types (constants, actions, etc.) seperate so other developers can easily read
2. Don't use undescriptive constants e.g. `LOAD` or `CREATE` as we might want to export them to other reducers
3. Don't use libraries like redux-promise as they do *not* create pure actions

`src/hotdog/hotdog.reducer.js`

```js
import { createAction, handleActions } from 'redux-actions';
import { reset as resetForm } from 'redux-form';
import { apiGetHotdogs, apiCreateHotdog } from './hotdog.service';

/**
 * Initial state
 */
const initialState = {
  hotdogs: [],
  current: null,
  problem: null, // use "problem" instead of "error" as error causes issues when passed as prop
  loading: false,
  success: false,
};

/**
 * Constants
 */
export const HOTDOG_RESET = 'HOTDOG_RESET';
export const HOTDOG_LOADING = 'HOTDOG_LOADING';
export const HOTDOG_ERRORED = 'HOTDOG_ERRORED';
export const HOTDOG_SET = 'HOTDOG_SET';
export const HOTDOG_CURRENT = 'HOTDOG_CURRENT';

/**
 * Actions
 * 
 * These describe what happened.
 */
export const resetHotdog = createAction(HOTDOG_RESET);
export const loadingHotdog = createAction(HOTDOG_LOADING);
export const erroredHotdog = createAction(HOTDOG_ERRORED);
export const setHotdog = createAction(HOTDOG_SET);
export const currentHotdog = createAction(HOTDOG_CURRENT);

/**
 * Thunks
 * 
 * The return value of the inner function should be a promise. The dispatch function
 * returns the value of the function from within it. This allows us to chain dispatch functions.
 */
export const attemptGetHotdogs = () => async (dispatch, getState) => {
  dispatch(loadingHotdog());
  try {
    const { token } = getState().player.auth;
    const hotdogs = await apiGetHotdogs(token);
    dispatch(setHotdog(hotdogs));
  } catch (error) {
    dispatch(erroredHotdog(error));
  }
  dispatch(loadingHotdog(false));
};
export const attemptCreateHotdog = () => async (dispatch, getState) => {
  dispatch(loadingHotdog());
  try {
    const state = getState();
    const { token } = state.player.auth;
    const formName = 'hotdogSimple';
    const body = { ...state.form[formName].values };
    const hotdog = await apiCreateHotdog(token, body);
    dispatch(currentHotdog(hotdog));
    dispatch(resetForm(formName));
  } catch (error) {
    dispatch(erroredHotdog(error));
  }
  dispatch(loadingHotdog(false));
};

/**
 * Reducer
 * 
 * All reducer functions should be pure. They describe how the state is mutated.
 */
export default handleActions({

  [HOTDOG_RESET]: () => ({
    ...initialState,
  }),

  [HOTDOG_LOADING]: (state, { payload = true }) => ({
    ...state,
    loading: payload,
  }),

  [HOTDOG_ERRORED]: (state, { payload = null }) => ({
    ...state,
    problem: payload,
  }),

  [HOTDOG_SET]: (state, { payload = [] }) => ({
    ...state,
    hotdogs: payload,
  }),

  [HOTDOG_CURRENT]: (state, { payload = null }) => ({
    ...state,
    current: payload,
  }),

}, initialState);
```

## Services

Services are standard implementations of functions that request data from the server and return the data in a promise.

Shoulds:

1. Do name the file `<feature>.service.js` e.g. `hotdog.service.js` or `drink.service.js`
2. Do convert the service values from JSON or XML to JavaScript objects (in the `handleResponse` method)
3. Do use a *config* value for setting the endpoint in the fetch calls

Should **nots**:

1. Don't put application logic inside these files, keep them encapsulated so we can swap them in and out as we want

`src/hotdog/hotdog.service.js`

```js
import config from '../config';

async function handleResponse(res) {
  if (res.status === 204) return {};
  const data = await res.json();
  if (res.ok) return data;
  throw data.error;
};

export const apiGetHotdogs = (token) => fetch(`${config.endpoint}/hotdogs`, {
  method: 'GET',
  headers: {
    'Authorization': token,
  },
}).then(handleResponse); // this extracts the json content from the response

export const apiCreateHotdog = (token, hotdog) => fetch(`${config.endpoint}/hotdogs`, {
  method: 'POST',
  headers: {
    'Authorization': token,
  },
  body: JSON.stringify(hotdog),
}).then(handleResponse); // this extracts the json content from the response
```

## Containers vs Components

Components should be split into smart `containers` which handle data and dumb `components` which handle presentation. There is a good post on it [here](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). The *main* difference between containers and components is that containers change or request the redux state, where as components are encapsulated and pure.

> I call components encapsulated React components that are driven solely by props and don't talk to Redux. Same as “dumb components”. They should stay the same regardless of your router, data fetching library, etc.
>
> I call containers React components that are aware of Redux, Router, etc. They are more coupled to the app. Same as “smart components”.
> ~ [gaearon](https://github.com/gaearon)

## Routing

Routing components are the entry point to your feature and should **only** handle routing and layout. Any action dispatches should be handled in a sub-container. As route components effect and are aware of the application's infrustructure and directly relate other containers to the view, we consider these components also as *containers*.

Shoulds:

1. Do only handle routing and layout
2. Do put it in the containers folder
3. Do name the file `<Feature>[<Type>]Routes.js` e.g. `HotdogRoutes.js` or `DrinkSidebarRoutes.js`

Should **nots**:

1. Don't handle dispatching of actions in these components
2. Don't style the component (style the sub-components)

`src/hotdog/containers/HotdogRoutes.js`

```jsx
import React from 'react';
import PropTypes from 'prop-types';
import { Switch, Route, Redirect } from 'react-router-dom';
import HotdogList from './HotdogList';
import HotdogCreate from './HotdogCreate';

function HotdogRoutes({ match }) {
  return (
    <Switch>
      <Route path={ match.url } exact component={ HotdogList } />
      <Route path={ `${match.url}/create` } component={ HotdogCreate } />
      <Redirect to={ match.url } />
    </Switch>
  );
}

HotdogRoutes.propTypes = {
  match: PropTypes.shape({
    url: PropTypes.string.isRequired,
  }).isRequired,
};

export default HotdogRoutes;
```

## Forms

Forms should consist of at least 2 component layers:

1. The *logic* layer 
2. The *interface* layer

The logic layer determines how the values in the form are handled, loaded, and changed. It directly dispatches actions to affect and load from the redux state. The logic layer also loads in the correct interface layer to present the form.

`src/hotdog/containers/HotdogCreate.js`

```jsx
import React, { Component } from 'react';
import PropTypes from 'prop-types';
import { connect } from 'react-redux';
import { attemptCreateHotdog } from '../hotdog.reducer';
import CoolModalThing from '../components/CoolModalThing';
import SimpleForm from '../components/SimpleForm';

class HotdogCreate extends Component {
  
  handleCreateHotdog(event) {
    event.preventDefault();
    this.props.attemptCreateHotdog();
  }
  
  render() {
    const { problem, loading, success } = this.props;
    return (
      <CoolModalThing
        showUnicorn={ success }
      >
        <SimpleForm 
          handleSubmit={ event => this.handleCreateHotdog(event) } // use arrow function to bind "this" to component
          problem={ problem }
          loading={ loading }
        />
      </CoolModalThing>
    );
  }
  
}

HotdogCreate.propTypes = {
  handleSubmit: PropTypes.func.isRequired,
  success: PropTypes.bool.isRequired,
  loading: PropTypes.bool.isRequired,
  problem: PropTypes.shape({
    message: PropTypes.string,
  }),
};

HotdogCreate.defaultProps = {
  problem: null,
};

const mapStateToProps = ({ hotdog: { problem, loading, success } }) => ({ problem, loading, success });
const mapDispatchToProps = { attemptCreateHotdog };
export default connect(mapStateToProps, mapDispatchToProps)(HotdogCreate);
```

The interface layer controls the form inputs and validations of those inputs. They do not *pull* the state. When the form is submitted, it will call a submit handler function passed to it by the logical layer. These interface layer form components can be reused by multiple logical components e.g. `HotdogCreate` and `HotdogEdit`.

`src/hotdog/containers/HotdogSimpleForm.js`

```jsx
import React from 'react';
import PropTypes from 'prop-types';
import { reduxForm } from 'redux-form';
import * as Formed from '../../shared/components/Formed'; // styled form components

const HotdogSimpleForm = ({ handleSubmit, loading, problem }) => (
  <form onSubmit={ handleSubmit }>
    <Formed.Control>
      <Formed.Label htmlFor="name">Name</Formed.Label>
      <Formed.Input name="name" type="text" component="input" />
    </Formed.Control>
    { problem && <Formed.ErrorAlert>{ problem.message }</Formed.ErrorAlert> }
    <Formed.Submit type="submit" disabled={ loading }>Submit</Formed.Submit>
  </form>
);

HotdogSimpleForm.propTypes = {
  handleSubmit: PropTypes.func.isRequired,
  loading: PropTypes.bool.isRequired,
  problem: PropTypes.shape({
    message: PropTypes.string,
  }),
};

HotdogSimpleForm.defaultProps = {
  problem: null,
};

export default reduxForm({ form: 'hotdogSimple' })(HotdogSimpleForm);
```

It is a good convention to keep forms related to the state of the app so the rest of the app can correctly access the data if they need to. To do this, we use the [redux-form](https://redux-form.com/) helper library. As per the above example, the higher order component of `reduxForm` is pushing data to the redux state. The application is aware of the form and therefore the interface layer component is considered as a *container* as well.

---

**Taking the bite out of React and Redux :crocodile:**
