# Polymer Flow

Set of behaviors to enable uni-directional data flow in Polymer application.

## History & Overview

When you start working on a new Polymer application, it's easy to start and build the first few elements, and make them communicate via events and data binding, so everything looks nice and rosy. However, as the number and complexity of elements grows, it becomes increasingly difficult to manage relationships between them, trace where/when the data changes happened, and debug the problems. So this project started as an attempt by our team at Google to find a good way to architect large Polymer application. 

Inspired by React's community Flux (and, later, Redux) architecture, we implemented a unidirectional data flow pattern (data down, events up) for Polymer. We found that when using Polymer Flow application code becomes more streamlined (e.g. it is clear what the responsibilities of each element are) and much easier to manage; the code has fewer bugs, and debugging is a lot more efficient. Adding new functionality no longer exponentially increases complexity.

This project was also inspired by Backbone Marionette. Backbone.js back in the days of its glory was a great library that provided a nice set of building blocks for building JavaScript applications. However, it left much of the application design, architecture and scalability to the developer, including memory management, view management, and more. Marionette brought an application architecture to Backbone, along with built in view management and memory management. It was designed to be a lightweight and flexible library of tools that sits on top of Backbone, providing the framework for building a scalable application.

We feel that Polymer, and web components in general, is a great concept that takes interoperability and encapsulation in Web development to the next level. But it lacked the patterns for building large and complex applications, and this is the void we expect Polymer Flow to fill. It is still in beta, so breaking changes may be happening before the first release. However, we believe that abstractions implemented in the library can be useful for Polymer community, so we encourage people to try, fork, ask questions, send  comments, and submit pull requests.

## Applicability

This library implements the architectural pattern called 'unidirectional data flow'. It works best if application logic involves complicated data management, when multiple elements need to have access to or modify the same data. Even though the pattern can be implemented just using built-in Polymer concepts, such as custom events and data binding, the Polymer Flow library provides a useful set of tools and abstractions, and helps to structure application code.

## Implementation

Polymer Flow is implemented as a set of behaviors that developers assign to their elements. It is assumed that each application has a singleton application element that maintains state of entire application. Each element that needs access to the data is bound, directly or indirectly, to sub-tree of application state tree. Two way data binding is never used to send data up, from child to parent, so only parent elements send data to children using one way data binding. Child elements, in turn, send the events (emit actions) responding to user actions, indicating that the data may need to be modified. Special non-visual elements called action dispatchers mutate the data, then all elements listening to the data changes render new data. 

## Action Dispatcher

Use PolymerFlow.ActionDispatcher for non-visual elements that process actions emitted by visual
elements. Action dispatchers usually placed at the application level. Each action dispatcher
element gets a chance to process the action in the order the elements are present in the
DOM tree. It is important that action dispatcher elements get two-way data binding to
application state as follows:

   <action-dispatcher state="{{state}}"></action-dispatcher>

Action dispatcher elements can include nested action dispatchers, so you can have a
hierarchical organization of action dispatchers.

### Example:

#### HTML:
```html
<dom-module id="parent-dispatcher">
<template>
  <child-dispatcher-a state="{{state}}"></child-dispatcher-a>
  <child-dispatcher-b state="{{state}}"></child-dispatcher-b>
</template>
</dom-module>
```    

#### JavaScript:

```javascript
Polymer({
  is: 'parent-dispatcher',
  
  behaviors: [
   PolymerFlow.ActionDispatcher
  ],
  
  MY_ACTION(detail) {
   // do MY_ACTION processing here
   // return false if you want to prevent other action dispatchers from
   // further processing of this action
  };
});
```

## Action Emitter

Whenever element needs to emit an action, this behavior should be used. Action object must always include type property.

## Application State

Assign this behavior to your main application element. It provides global
state and functionality to maintain individual elements states. This behavior
is responsible for notifying all state-aware elements about their state
changes (provided those elements have `statePath` property defined).
Only one element in the application is supposed to have this behavior.

### Example:

#### HTML:
```html
<template>
  <!-- action dispatchers in the order of action processing -->
  <action-dispatcher-a state="{{state}}"></action-dispatcher-a>
  <action-dispatcher-b state="{{state}}"></action-dispatcher-b>
  
  <!-- state-aware elements -->
  <some-element state-path="state.someElement"></some-element>
</template>
```
#### JavaScript:

```javascript
Polymer({
  is: 'my-app',

  behaviors: [
    PolymerFlow.ApplicationState
  ],

  attached() {
    this.state = {
      someElement: {}
    }
  }
});
```

In the example above, `<some-element>` will receive notification of any changes to the state,
as if it was declared as follows:

```html
<some-element state="[[state]]"></some-element>
```

Also, if `<some-element>` has `propertyA`, on element attach this property will be assigned
the value of `state.someElement.propertyA`, and receive all notification of the property change
whenever the corresponding data in state tree changes. This essentially translates to following
declaration:

```html
<some-element state="[[state]]"
              propertyA="[[state.someElement.propertyA]]">
</some-element>
```

Note that data binding is one-way in both cases. Although state-aware elements can modify their
own state, it is considered their private state and no other elements will be notified of those
changes.

## List View

This behavior used by elements that need to render multiple models backed
by 'list' array. You may want to use ModelView to render individual
models in the list. The behavior supports element selection by setting predefined
$selected property on list elements.

### Example:

#### HTML:

```html
<ul>
  <template id="list-template" is="dom-repeat" items="[[list]]">
    <li id="[[item.id]]">
      <paper-checkbox checked="{{item.$selected}}">
      <model-view state-path="[[statePath]].list.#[[index]]"></model-view>
    </li>
  </template>
</ul>
Selected: [[selectedCount]] items
<paper-button on-tap="onDeleteTap">Delete</paper-button>
```

#### JavaScript:

```javascript
Polymer({

  is: "list-element",

  behaviors: [
    PolymerFlow.ListView,
    PolymerFlow.StateAware
  ],

  onDeleteTap() {
    this.deleteSelected();
  }

});
```

In the example above list view element is also state-aware, meaning it has its own place
in the application state tree. Assuming it has been declared as follows:

```html
<list-element state-path="state.listElement"></list-element>
```

it will be rendering `state.listElement.list` and observing changes to it. Each `model-view`
within dom-repeat template will have `state-path` property  set to
`state.listElement.list.#<index>`  where `index` is the element's index in the array.
