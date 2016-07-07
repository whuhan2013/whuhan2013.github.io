---
layout: post
title: React Flux Learn
date: 2016-7-7
categories: blog
tags: [React Native]
description: React Flux Learn
---

Flux is the application architecture that Facebook uses for building client-side web applications. It complements React's composable view components by utilizing a unidirectional data flow. It's more of a pattern rather than a formal framework, and you can start using Flux immediately without a lot of new code.       


Flux applications have three major parts: the dispatcher, the stores, and the views (React components). These should not be confused with Model-View-Controller. Controllers do exist in a Flux application, but they are controller-views — views often found at the top of the hierarchy that retrieve data from the stores and pass this data down to their children. Additionally, action creators — dispatcher helper methods — are used to support a semantic API that describes all changes that are possible in the application. It can be useful to think of them as a fourth part of the Flux update cycle.

**Structure and Data Flow **                 
Data in a Flux application flows in a single direction:    
![](http://facebook.github.io/flux/img/flux-simple-f8-diagram-1300w.png)

A unidirectional data flow is central to the Flux pattern, and the above diagram should be the primary mental model for the Flux programmer. The dispatcher, stores and views are independent nodes with distinct inputs and outputs. The actions are simple objects containing the new data and an identifying type property.

The views may cause a new action to be propagated through the system in response to user interactions:

![](http://facebook.github.io/flux/img/flux-simple-f8-diagram-with-client-action-1300w.png)

All data flows through the dispatcher as a central hub. Actions are provided to the dispatcher in an action creator method, and most often originate from user interactions with the views. The dispatcher then invokes the callbacks that the stores have registered with it, dispatching actions to all stores. Within their registered callbacks, stores respond to whichever actions are relevant to the state they maintain. The stores then emit a change event to alert the controller-views that a change to the data layer has occurred. Controller-views listen for these events and retrieve data from the stores in an event handler. The controller-views call their own setState() method, causing a re-rendering of themselves and all of their descendants in the component tree.

![](http://facebook.github.io/flux/img/flux-simple-f8-diagram-explained-1300w.png)

### Example  

**Dispatcher**                
The dispatcher is the central hub that manages all data flow in a Flux application. It is essentially a registry of callbacks into the stores and has no real intelligence of its own — it is a simple mechanism for distributing the actions to the stores. Each store registers itself and provides a callback. When an action creator provides the dispatcher with a new action, all stores in the application receive the action via the callbacks in the registry.    

```
 /* AppDispatcher
 *
 * A singleton that operates as the central hub for application updates.
 */

var Dispatcher = require('flux').Dispatcher;

module.exports = new Dispatcher();
```   

**stores**             
Stores contain the application state and logic. Their role is somewhat similar to a model in a traditional MVC, but they manage the state of many objects — they do not represent a single record of data like ORM models do. Nor are they the same as Backbone's collections. More than simply managing a collection of ORM-style objects, stores manage the application state for a particular domain within the application.

```
// Register callback to handle all updates
AppDispatcher.register(function(action) {
  var text;

  switch(action.actionType) {
    case TodoConstants.TODO_CREATE:
      text = action.text.trim();
      if (text !== '') {
        create(text);
        TodoStore.emitChange();
      }
      break;

    case TodoConstants.TODO_TOGGLE_COMPLETE_ALL:
      if (TodoStore.areAllComplete()) {
        updateAll({complete: false});
      } else {
        updateAll({complete: true});
      }
      TodoStore.emitChange();
      break;

    case TodoConstants.TODO_UNDO_COMPLETE:
      update(action.id, {complete: false});
      TodoStore.emitChange();
      break;

    case TodoConstants.TODO_COMPLETE:
      update(action.id, {complete: true});
      TodoStore.emitChange();
      break;

    case TodoConstants.TODO_UPDATE_TEXT:
      text = action.text.trim();
      if (text !== '') {
        update(action.id, {text: text});
        TodoStore.emitChange();
      }
      break;

    case TodoConstants.TODO_DESTROY:
      destroy(action.id);
      TodoStore.emitChange();
      break;

    case TodoConstants.TODO_DESTROY_COMPLETED:
      destroyCompleted();
      TodoStore.emitChange();
      break;

    default:
      // no op
  }
});

module.exports = TodoStore;
```

**Views and Controller-Views**            
React provides the kind of composable and freely re-renderable views we need for the view layer. Close to the top of the nested view hierarchy, a special kind of view listens for events that are broadcast by the stores that it depends on. We call this a controller-view, as it provides the glue code to get the data from the stores and to pass this data down the chain of its descendants. We might have one of these controller-views governing any significant section of the page.


```
var React = require('react');
var TodoActions = require('../actions/TodoActions');
var TodoTextInput = require('./TodoTextInput.react');

var Header = React.createClass({

  /**
   * @return {object}
   */
  render: function() {
    return (
      <header id="header">
        <h1>todos</h1>
        <TodoTextInput
          id="new-todo"
          placeholder="What needs to be done?"
          onSave={this._onSave}
        />
      </header>
    );
  },

  /**
   * Event handler called within TodoTextInput.
   * Defining this here allows TodoTextInput to be used in multiple places
   * in different ways.
   * @param {string} text
   */
  _onSave: function(text) {
    if (text.trim()){
      TodoActions.create(text);
    }

  }

});

module.exports = Header;
```



**Actions**              
The dispatcher exposes a method that allows us to trigger a dispatch to the stores, and to include a payload of data, which we call an action. The action's creation may be wrapped into a semantic helper method which sends the action to the dispatcher. For example, we may want to change the text of a to-do item in a to-do list application. We would create an action with a function signature like updateText(todoId, newText) in our TodoActions module. This method may be invoked from within our views' event handlers, so we can call it in response to a user interaction. This action creator method also adds a type to the action, so that when the action is interpreted in the store, it can respond appropriately. In our example, this type might be named something like TODO_UPDATE_TEXT.


```
var AppDispatcher = require('../dispatcher/AppDispatcher');
var TodoConstants = require('../constants/TodoConstants');

var TodoActions = {

  /**
   * @param  {string} text
   */
  create: function(text) {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_CREATE,
      text: text
    });
  },

  /**
   * @param  {string} id The ID of the ToDo item
   * @param  {string} text
   */
  updateText: function(id, text) {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_UPDATE_TEXT,
      id: id,
      text: text
    });
  },

  /**
   * Toggle whether a single ToDo is complete
   * @param  {object} todo
   */
  toggleComplete: function(todo) {
    var id = todo.id;
    var actionType = todo.complete ?
        TodoConstants.TODO_UNDO_COMPLETE :
        TodoConstants.TODO_COMPLETE;

    AppDispatcher.dispatch({
      actionType: actionType,
      id: id
    });
  },

  /**
   * Mark all ToDos as complete
   */
  toggleCompleteAll: function() {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_TOGGLE_COMPLETE_ALL
    });
  },

  /**
   * @param  {string} id
   */
  destroy: function(id) {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_DESTROY,
      id: id
    });
  },

  /**
   * Delete all the completed ToDos
   */
  destroyCompleted: function() {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_DESTROY_COMPLETED
    });
  }

};

module.exports = TodoActions;
```

**effect**   
![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/React/react2.png)





