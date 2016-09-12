# Polymer Flow

Set of behaviors to enable uni-directional data flow in Polymer application.

## History & Overview

When you start working on a new Polymer application, it's easy to start and build the first few elements, and make them communicate via events and data binding, so everything looks nice and rosy. However, as the number and complexity of elements grows, it becomes increasingly difficult to manage relationships between them, trace where/when the data changes happened, and debug the problems. So this project started as an attempt by our team at Google to find a good way to architect large Polymer application. 

Inspired by React's community Flux (and, later, Redux) architecture, we implemented a unidirectional data flow pattern (data down, events up) for Polymer. We found that when using Polymer Flow application code becomes more streamlined (e.g. it is clear what the responsibilities of each element are) and much easier to manage; the code has fewer bugs, and debugging is a lot more efficient. Adding new functionality no longer exponentially increases complexity.

This project was also inspired by Backbone Marionette. Backbone.js back in the days of its glory was a great library that provided a nice set of building blocks for building JavaScript applications. However, it left much of the application design, architecture and scalability to the developer, including memory management, view management, and more. Marionette brought an application architecture to Backbone, along with built in view management and memory management. It was designed to be a lightweight and flexible library of tools that sits on top of Backbone, providing the framework for building a scalable application.

We feel that Polymer, and web components in general, is a great concept that takes interoperability and encapsulation in Web development to the next level. But it lacked the patterns for building large and complex applications, and this is the void we expect Polymer Flow to fill. It is still in beta, so breaking changes may be happening before the first release. However, we believe that abstractions implemented in the library can be useful for Polymer community, so we encourage people to try, fork, ask questions, send  comments, and submit pull requests.

## Applicability

This library implements the architectural pattern called 'unidirectional data flow'. It works best if application logic involves complicated data management, when multiple elements need to have access to or modify the same data. Even though the pattern can be implemented just using built-in Polymer concepts, such as custom events and data binding, the Polymer Flow library provides a useful set of tools and abstractions, and helps to structure application code.

## Documentation

Polymer Flow is implemented as a set of behaviors that developers assign to their elements. It is assumed that each application has a singleton application element that maintains state of entire application. Each element that needs access to the data is bound, directly or indirectly, to sub-tree of application state tree. Two way data binding is never used to send data up, from child to parent, so only parent elements send data to children using one way data binding. Child elements, in turn, send the events (emit actions) responding to user actions, indicating that the data may need to be modified. Special non-visual elements called action dispatchers mutate the data, then all elements listening to the data changes render new data. 
