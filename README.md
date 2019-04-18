# NgrxForm [![npm version](https://badge.fury.io/js/%40yoozly%2Fngrx-form.svg)](https://badge.fury.io/js/%40yoozly%2Fngrx-form)

A lib for Angular that binds reactive-forms and @ngrx/store together. Every change in a form is reflected in the store and vice versa. The store object for each form is automatically generated by the library.

The lib is designed to work in a feature modules, but it can be implemented in the root module as well. A feature module can handle multiple forms.

## Installation

Npm : `npm i @yoozly/ngrx-form -s`

Yarn : `yarn add @yoozly/ngrx-form`

## Setup

### Root Module

In the root module of your application, add `ngrxForm` in the array of metareducers

```javascript
import { ngrxForm } from '@yoozly/ngrx-form';

@NgModule({
    StoreModule.forRoot(reducers, {
        metaReducers: [ngrxForm]
    })
})
export class RootModule {}
```

### Feature Module

First, we need to import the following dependendies in the feature module :

- `NgrxFormModule`
- `NgrxFormReducer`, a utility that creates the reducers for us
- `FEATURE_REDUCER_TOKEN` to inject the feature name

```javascript
import {
  NgrxFormModule,
  NgrxFormReducer,
  FEATURE_REDUCER_TOKEN
} from '@yoozly/ngrx-form';
```

Then, use the same feature name (`StoreModule` and `NgrwFormModule`) and use the `FEATURE_REDUCER_TOKEN` injection token.

```javascript
@NgModule({
  imports: [
    ...
    StoreModule.forFeature('someFeatureName', FEATURE_REDUCER_TOKEN),
    NgrxFormModule.forFeature('someFeatureName')
    ...
  ]
})
```

Finally, use the providers array to configure the reducers that will be created. You need to pass an array of names to the `NgrxFormReducer` factory function.

```javascript
@NgModule({
  ...
  providers: [
    {
      provide: FEATURE_REDUCER_TOKEN,
      useFactory: function(): ActionReducerMap<any> {
        return {
          ...reducers,
          ...NgrxFormReducer(['formNameA','formNameB','formNameC'])
        };
      }
    }
  ]
})
```

Here is the full configuration :

```javascript
import {
  NgrxFormModule,
  NgrxFormReducer,
  FEATURE_REDUCER_TOKEN
} from '@yoozly/ngrx-form';
@NgModule({
  StoreModule.forFeature('someFeatureName', FEATURE_REDUCER_TOKEN),
  NgrxFormModule.forFeature('someFeatureName')
  providers: [
    {
      provide: FEATURE_REDUCER_TOKEN,
      useFactory: function(): ActionReducerMap<any> {
        return {
          ...reducers,
          ...NgrxFormReducer(['formNameA','formNameB','formNameC'])
        };
      }
    }
  ]
})
```

### Usage

Use the `NgrxFormConnect` directive to bind your form group to the store.

```html
<form [formGroup]="myFormGroup" NgrxFormConnect="formNameA">
  ...
</form>
```

The form name must be the same that the one provided in `NgrxFormReducer` array.

```javascript
return {
  ...reducers,
  ...NgrxFormReducer(['formNameA', 'formNameB'])
};
```

```html
<form [formGroup]="myFormGroupA" NgrxFormConnect="formNameA">
  ...
</form>
<form [formGroup]="myFormGroupA" NgrxFormConnect="formNameB">
  ...
</form>
```

## API

### Models

`NgrxFormState<T>`: the interface for each generated form. An optional type can be passed with the form content (ie form fields).

```javascript
export interface NgrxFormState<T> {
  value: T;
  errors?: { [fieldName: string]: string };
  pristine?: Boolean;
  valid?: Boolean;
}
```

`UpdateFormPayload`: the interface for the UpdateForm action payload.

```javascript
export interface UpdateFormPayload<T> {
  feature: string;
  path: string;
  form: NgrxFormState<T>;
}
```

## Action

`Updateform` : the action used to update the store.
The payload must contains :

- the feature name
- the path of the form state (featurename.pathname --> ie the form name)
- the form datas

```javascript
this.store.dispatch(
  new Updateform({
  feature: 'theFeatureName',
  path: 'theFormName',
  form: {
    value: { /* the form datas */},
    errors: {},
    pristine: false,
    valid: false
  })
)
```
