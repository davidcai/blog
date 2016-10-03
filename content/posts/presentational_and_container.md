---
title: "Bridge Redux and Angular component"
date: "2016-10-02"
description: "Bring Redux state and Angular component bindings together by the Presentational and Container component pattern."
categories:
  - "angular"
  - "redux"
draft: true
hero: "/img/dynamic-templates.png"
heroalt: "Dynamic templates"
---

When I was developing the [canvas component](https://pacific.medallia.com/display/OCEAN/Feature%3A+Builders+-+Canvas+1.1), I had some struggles with Redux states and Angular component bindings. The canvas component can be implemented in two ways -- pure Angular component, or component with Redux state. 

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

One obvious difference is that all binding attributes such as `roles` and `role-id` are gone. The component now doesn't need its ancestor components to supply the configuration and inputs. Instead, the configuration and inputs come from the Redux store, and the component's controller manages how to bridge its data modal with Redux state. In component's controller, you will find the code that selects and maps Redux states (application state) to controller's `this`.

```javascript
export default class OaCanvasController {
  constructor($ngRedux) {
    // ...
  }

  // Select and map Redux state to `this`.
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


### Problems

In the case of Canvas, the component needs to display a preview of a working modal. Whenever the working modal is changed and saved, the canvas will need to update the preview. The events that can trigger this preview update can come from many places which dont't necessary to be canvas component's parent or ancestor components. For example, a control panel which modifies the working modal can be a sibling component to the canvas. 

The first approach -- pure Angular component inherits its data modal from ancestor components:

```html
<oa-canvas roles="$ctrl.roles" role-id="$ctrl.roles[0].id" is-published="false">
</oa-canvas>
```

This approach requires an ancestor's controller to supply and manage the data modal, e.g. `roles`. When the number of places that can trigger modal changes increases, you might find yourself pushing up the code that manages the data modal to higher-level controllers, so the modal changes can be triggered from anywhere but to a single source. Vetran Angular developer would use a service to centralize the methods of data modal's setters and getters, and inject such service to wherever needed. Or, an event subscription-and-publication pattern. In Redux, this is implemented in Actions and Reducers. This leads us to the second appraoch -- component with Redux state:

```html
<oa-canvas></oa-canvas>
```

For the 2nd approach -- components with Redux, the component's data modal comes from Redux store. The `mapStateToThis` method in the controller aforementioned selects and maps Redux state to the component's scope `this`.

```javascript
// Select and map Redux state to `this`.
mapStateToThis(state) {
  return {
    roles: state.dashboard.roles,
    roleId: state.canvas.roleId,
    isPublished: state.canvas.isPublished
  };
}
```

Since the mapping is done inside the component, the component assumes the knowledge of application state and how to interpret the state to fit into its own need. This creates a tight coupling between the application and the component. The tight coupling  reduces the re-usability of the component in different applications, since other application might have a different state scheme, e.g. `state.dashboard.roles` doesn't exist, instead, `roles` can be found in `state.report.roles` in a report application.

Another obvious disadvantage is that the component can't work in a context without Redux. 


### Solution

The proposed solution is to adopt the [Presentational and Container component pattern](http://redux.js.org/docs/basics/UsageWithReact.html#presentational-and-container-components). The goal is to create a clear separation of responsibility -- Presentational component: pure UI functionalities, and Contaienr component: bridge Redux state to presentational component.

Our Angular component `<oa-canvas>` in the first approach can be the presentational component, which only honors the inputs passed through the binding attributes. The binding attributes is the component's only contract of communications with the outside world. It has no idea of the application logic nor Redux.

Then, I created a `<oa-canvas-container>` component that wraps the presentational component -- `<oa-canvas>`. It connects to Redux store and uses the `mapStateToThis` method to map Redux states to component's `this`. Basically, this is all the container component does. It is a thin layer wrapping the actual presentational component:

```javascript
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

With this pattern, we are able to re-use the presentational component `oa-canvas` in different applications that might not use Redux at all. And the container component `oa-canvas-container` is solely responsible for state management. 

How does this solution handle the ?

In Redux code, I have an action creator that creates actions to change publish state `isPublished`. 

```javascript
export const CANVAS_PUBLISH_STATE_CHANGE = 'CANVAS_PUBLISH_STATE_CHANGE';

export function fireChangeCanvasRoleId(isPublished) {
  return {
    type: CANVAS_PUBLISH_STATE_CHANGE,
    payload: {
      isPublished
    }
  };
}
```

The action creator can be called anywhere in the application. A reducer will capture the actions created by the above action creator, and return new states with the changed `isPublished`. The `mapStateToThis` method defined in container component `oa-canvas-container` then will be called to 



