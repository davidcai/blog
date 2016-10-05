---
title: "Bridge Redux and Angular 1.5 components"
date: "2016-10-02"
description: "Bring Redux state and Angular 1.5 component bindings together by the Presentational and Container component pattern."
categories:
  - "angular"
  - "redux"
draft: true
hero: "/img/dynamic-templates.png"
heroalt: "Dynamic templates"
---

When I was developing the [canvas component](https://pacific.medallia.com/display/OCEAN/Feature%3A+Builders+-+Canvas+1.1), I had some struggles with Redux states and Angular component bindings. The canvas component can be implemented in one of the two ways -- pure Angular component, or component with Redux state. Neither is necessarily wrong, however, both approaches have limitations.

### Pure Angular component:

```html
<oa-canvas roles="$ctrl.roles" role-id="$ctrl.roles[0].id" is-published="false">
</oa-canvas>
```

Basically, all configurations and inputs for this component are passed through the binding attributes, e.g. role-id. The component itself has no knowlege of the application states, nor the presence of Redux. The component's data model is driven by its ancestor's scope properties, e.g. `$ctrl.roles`, and `$ctrl.roles[0].id`. This is a typical Angular component.


### Component with Redux state

```html
<oa-canvas></oa-canvas>
```

One obvious difference of this approach than the pure Angular component is that all binding attributes such as `roles` and `role-id` are gone. The component now doesn't need its ancestor components to supply the configuration and inputs. Instead, the configuration and inputs (let's call them data model for the component) come from the Redux store, and the component's controller manages how to bridge its data model with Redux state. In component's controller, you will find the code that selects and maps Redux states (application state) to controller's `this`.

```javascript
export default class OaCanvasController {
  constructor($ngRedux) {
    // ...
  }

  // Select and map application state from Redux store to `this`.
  mapStateToThis(state) {
    return {
      roles: state.dashboard.roles,
      roleId: state.canvas.roleId,
      isPublished: state.canvas.isPublished
    };
  }

  // ...
}
```


### Problem 1

In the case of Canvas, the component needs to display a preview of a working model. Whenever the working model is changed and saved, the canvas will need to update the preview. The events that trigger the update can come from many places which dont't necessary to be canvas' parent or ancestor components. For example, a control panel which modifies the working model can be a sibling component to the canvas. 

The first approach -- pure Angular component's data model is supplied by its ancestor components:

```html
<oa-canvas roles="$ctrl.roles" role-id="$ctrl.roles[0].id" is-published="false">
</oa-canvas>
```

This approach requires an ancestor's controller to manage the data model. When the number of places that can trigger model changes increases, you might find yourself pushing up the code that manages the data model to higher-level controllers, so the model changes can be triggered from anywhere but to a single source. 

![Data model bindings](/img/data-model-bindings.png)

Vetran Angular developer would use a service to centralize the updates of data model, and inject such service to wherever needed. Or, implement a subscription-and-publication pattern that allows events to be thrown anywhere, and interesting parties such as Canvas can subscribe to an event manager and capture the event. This gives us the most flexibilty in terms of code coupling. In case components need to be re-organized in HTML hierarchical structures, we don't have to worry about rewiring the data model from the component's parent controller to the component's binding attributes. Because the data model doesn't reply on parents's controllers. Instead, a single source at application level will provide the data model, and any change will be communicated through events. In Redux, this pattern is incarnated by Store, Actions and Reducers.


### Problem 2

This leads us to the second approach -- component with Redux state:

```html
<oa-canvas></oa-canvas>
```

For the second approach -- components with Redux, the component's data model comes from Redux store. The `mapStateToThis` method in the controller aforementioned selects and maps state from Redux store to the component's controller.

```javascript
// canvas-controller.js

// Select and map Redux state to `this`.
mapStateToThis(state) {
  return {
    roles: state.dashboard.roles,
    roleId: state.canvas.roleId,
    isPublished: state.canvas.isPublished
  };
}
```

Since the mapping is done inside the component, the component assumes the knowledge of application state and how to interpret the state to fit into its own need. This creates a tight coupling between the application and the component. The tight coupling reduces the re-usability of the component in different applications, since other application might have a different state scheme, e.g. `state.dashboard.roles` doesn't exist, instead, `roles` can be found at `state.report.roles` in a report application.

Another obvious disadvantage is that the component can't work in an application without Redux. 


### Solution

The proposed solution is to adopt the [Presentational and Container component pattern](http://redux.js.org/docs/basics/UsageWithReact.html#presentational-and-container-components). The goal is to have a clear separation of responsibilities:

* Presentational component -- pure UI functionalities.
* Contaienr component -- bridge Redux state to presentational component.

Our Angular component `<oa-canvas>` in the first approach can be a presentational component, which only honors the inputs passed through the binding attributes. The binding attributes is the component's only contract of communications with the outside world. It has no idea of the application logic nor Redux.

Then, a `<oa-canvas-container>` container component wraps the presentational component -- `<oa-canvas>`. The container component connects to Redux store and uses the `mapStateToThis` method to map Redux states to component's controller. This is all the container component does. It is a thin layer over the actual presentational component:

```javascript
// canvas-container-controller.js

const _$ngRedux = Symbol('_$ngRedux');
const _disconnect = Symbol('_disconnect');

angular.component('oaCanvasContainer', {
  template: `
    <oa-canvas
      roles="$ctrl.roles"
      role-id="$ctrl.roleId"
      is-published="$ctrl.isPublished">
    </oa-canvas>
  `,
  controller: class OaCanvasContainerController {
    constructor($ngRedux) {
      this[_$ngRedux] = $ngRedux;
    }

    mapStateToThis(state) {
      return {
        roles: state.dashboard.roles,
        roleId: state.canvas.roleId,
        isPublished: state.canvas.isPublished
      };
    }

    $onInit() {
      this[_disconnect] = this[_$ngRedux].connect(this.mapStateToThis, {})(this);
    }

    $onDestroy() {
      this[_disconnect]();
    }
  }
});
```

With this pattern, we let the container component `oa-canvas-container` take care of the state managemnt, and the presentational component `oa-cavnas` focus on UI functionalities. We are even able to re-use `oa-canvas` in applications that might not use Redux at all.

How does this solution help us handle the events scenario that I mentioned early? How can we trigger Canvas' preview updates from almost anywhere in the application?

For instance, in Redux code, we will have an action creator that creates actions to change publish state `isPublished`. In a sense, an action is like event with a type and payload object that contains the change.

```javascript
// canvas-actions.js

export const CANVAS_PUBLISH_STATE_CHANGE = 'CANVAS_PUBLISH_STATE_CHANGE';

export function fireChangeCanvasPublishState(isPublished) {
  return {
    type: CANVAS_PUBLISH_STATE_CHANGE,
    payload: {
      isPublished
    }
  };
}
```

This action creator can be bound to any component's controller via the ng-redux library, and thus can be called anywhere in the application. The action creator fires the action. Reducers will capture the action, and return new states with the changed `isPublished`. The `mapStateToThis` method defined in container component `oa-canvas-container` will then be called to update the component's data model using the new state. This is all regular Redux usage. 


### From Presentational component to Container

What if the `oa-canvas` component allows users to select publish state from published to unpublished, and we want such change to be propagated out from `oa-canvas` as changed state?

First, we need to have a callback to be passed to `oa-canvas`, so `oa-canvas` will notify us when the publish state is changed. This is done via pure Angular binding:

```html
<oa-canvas
  ...
  on-publish-state-change="$ctrl.doSomething($event)">
</oa-canvas>
```

Then, in the container component `oa-canvas-container`, we define the callback, e.g. `onPublishStateChange`. When the callback is invoked, we call the `fireChangeCanvasPublishState` action creator (see above) to fire an action that is to modify the `isPublished` state.

```javascript
// canvas-container-controller.js

// Import action creator
import { fireChangeCanvasPublishState } from './canvas-actions';

angular.component('oaCanvasContainer', {
  template: `
    <oa-canvas
      roles="$ctrl.roles"
      role-id="$ctrl.roleId"
      is-published="$ctrl.isPublished"
      on-publish-state-change="$ctrl.onPublishStateChange($event)">
    </oa-canvas>
  `,
  controller: class OaCanvasContainerController {
    ...

    $onInit() {
      this[_disconnect] = this[_$ngRedux].connect(this.mapStateToThis, {
        // Bind action creator to controller
        fireChangeCanvasPublishState
      })(this);
    }

    onPublishStateChange($event) {
      this.fireChangeCanvasPublishState($event.isPublished);
    }

    ...
  }
});
```


### Final notes

Not all components need to be composed of both presentational and container components. I'd suggest starting from presentational components first. Later, if you find one of the following is true, you might want to create a container component to wrap the presentational component: 

* The component's internal state can be driven by many parts of the application.
* The component's internal state is useful to be saved as appliction state. 


