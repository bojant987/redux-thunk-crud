# redux-thunk-crud

redux-thunk-crud is a library that seeks to eliminate the boilerplate of writing highly repetitive reducers and action creators for simple RESTful API CRUD actions, using redux-thunk to handle asynchronous API calls. It's made of:

CrudActionCreators - a class encapsulating CRUD action creators for a single endpoint

createCrudReducers - a higher-order function that creates a reducer for a given CrudActionCreators instance

fetchAdapter - CrudActionCreators takes an adapter which provides the ability to make API calls. This adapter implements API calls using Fetch API and is included in order to provide a default and make the library testable without adding a dependency

[![npm version](https://badge.fury.io/js/redux-thunk-crud.svg)](http://badge.fury.io/js/redux-thunk-crud)

## Installation

```sh
npm install --save redux-thunk-crud
```

You will also need Redux, which is a peer dependency, so if you don't already have it:

```sh
npm install --save redux
```

## Basic usage

The parts of the library are available as named exports:

```js
import {createCrudReducers, CrudActionCreators} from 'redux-thunk-crud';
```

fetchAdapter is also available as a named export but in most cases shouldn't be used directly.

The first step is to instantiate an object containing action creators for an endpoint, since this also generates action types:

```js
const someThingsActionCreators = new CrudActionCreators('https://jsonplaceholder.typicode.com/posts', 'SOME_THING');
```

SOME_THING is the suffix for action types which will be generated, for example REQUEST_SOME_THINGS, DELETE_SOME_THINGS, etc. A full set of action types is available in someThingsActionCreators.actionTypes

Next, we create the reducer:

```js
const someThingsReducer = createCrudReducers(someThingsActionCreators);
```

Which can simply be fed to combineReducers or directly to createStore just like any other reducer.

## Action creators

The action creators constructor has the following signature:
```js
new CrudActionCreator(url, actionTypesSuffix, settings = {})
```

The parameters are:

**url** - url for the endpoint

**actionTypesSuffix** - a suffix to append to the generic preffies in order to generate action types, should match the name of the resource, in singular, for easier debugging

**settings** - other optional settings, described in the following table:

| Property         | Type    | Default | Description |
| ------------ | ------- | ------- | ----------- |
| **adapter** | function | fetchAdapter | A thunk action creator used to make an API call according to the given parameters. Custom adapters may be supplied and are documented below. |
| **primaryKey** | string | 'id' | Property of the resource used as primary key. |
| **actionTypesSuffixPlural** | string | undefined | Action types suffix to use when plural is needed. If undefined will be generated by appending 'S' to the singular suffix provided as constructor argument. |
| **json** | boolean | false | Whether to encode request body as json. This is passed on to the adapter which should encode the data and set appropriate headers. |
| **headers** | array of strings | [] | Headers to add to the request. Content type headers for json requests should be set automatically by the adapter. |
| **getters** | object | {} | Object that should contain functions used to get data included into dispatched actions from the data received from the API for GET requests. Possible keys are getList and getOne. Both will be passed the response received from the API. |
| **responseType** | json | undefined | Expected type of response from the API. Passed on to the adapter. The default is undefined to let the adapter use its own default.|

Normally only the following thunk action creators should be used directly:

| Method         | Type    | Description | Arguments |
| ------------ | ------- | ----------- | ----------- |
| **fetchList(params)** | R | Makes a GET request to fetch the list of resources. | **params **- object - optional search parameters |
| **fetchOne(id)** | R | Makes a GET request to fetch a single resource. | **id** - any primitive - primary key of the resource to fetch, appended to the endpoint url |
| **save(data,id,method)** | C/U | Makes a request to create or update a resource. If no method is specified will make POST to create and PUT to update. | **data** - object - object representing the resource to create or new values for the update <br> **id** - any primitive - optional primary key of the resource to update, appended to the url if provided <br> **method** - string - optinal methed to override the default behavior, if not specified POST will be assumed if id is provided and PUT if id is undefined. |
| **delete(id)** | D | Makes a DELETE request to delete a resource. | **id** - any primitive - primary key of the resource to delete, appended to the endpoint url |

## Adapters

The library abstracts away API calls by using an adapter function provided to the constructor as argument. The default adapter implements API calls using Fetch API in order to provide a default that works for testing without adding a higher-level library as dependency. In a real scenario, you will probably want to write your own adapter which implements the calls using your favorite XMLHttpRequest library. An adapter is a thunk creator which should take the same config object as the included fetchAdapter and return a thunk, a function that takes redux's dispatch() function as an argument and returns a promise. Adapter config object has the following properties:

| Property | Type | Description |
| ------------ | ------- | ----------- |
| **requstAction** | function | Action creator dispatched just before the API call is made. |
| **successAction** | function | Action creator dispatched when the API call returns a non-error response. |
| **failureAction** | function | Action creator dispatched when the API call returns an error response. |
| **method** | string | HTTP method for the API call. |
| **url** | string | URL of the endpoint to make the request to. |
| **params** | object | Params for the API call, sent either as query string or request body. |
| **json** | boolean | Whether to encode params as json and add content type header if request body is sent. |
| **headers** | array | Additional headers to add to the request. |
| **responseType** | string | Type of data expected as response. |

Any of the config properties may be omitted and handling undefined properties is left to the implementation of the adapter. It's usually a good idea to not dispatch actions when an action creator is not defined for example.


## Enhancing and Customizing

If you need more than the stanadrd CRUD action, writing additional action creators as separate functions is the most straightforward way to implement them. If those actions are repetitive, CrudActionCreators can be extended to add new action creators or override the existing ones.

The reducer can be enhanced to handle additional action types by passing it through a higher order reducer.

## Contributing

We are happy to take contributions in form of issues and PRs, but please keep in mind that this is supposed to be a minimal CRUD library. There's a lot of API weirdness in the world but there's no intent to support every little quirk every API developer thought was a good idea to implement. To handle those in your own project, please refer to the Enhancing and Customizing section above. For anything else, please feel free to open an issue or PR and we will be happy to address it as soon as possible.

When making a PR, please make sure you are running prettier and tests as a pre-commit hook and not forcing commits without those. Also please add tests for any bigger change that is not covered by existing tests.