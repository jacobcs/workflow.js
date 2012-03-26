# workflow.js

Finite-state machine (FSM) for Backbone.js with an **elegant** syntax. Works as a drop-in extension of Backbone.Model. What's it got on other JS-based state machines? It's simple, it works out-of-the-box with Backbone.js, and it supports multiple state machines. This FSM is loosely modeled after the [Ruby workflow gem from geekq](https://github.com/geekq/workflow).

### Dependencies
* JQuery
* [Backbone.js](http://documentcloud.github.com/backbone/) (Tested in 0.9.1, but probably works in most previous versions.)
* [Underscore.js](http://documentcloud.github.com/underscore/) (Tested in 1.3.1, but same as above – probably works in past versions.)

## Setup

Just add these dependencies to your site's `<head>`, **in order**:

```
<script src="jquery.js"></script>
<script src="underscore.js"></script>
<script src="backbone.js"></script>
<script src="workflow.js"></script>
```

## Usage

Let's start with a complete example (given in [CoffeeScript](http://coffeescript.org/)):

```
class User extends Backbone.Model
  defaults:
    signed_up_at: null

  workflow:
    initial: 'visitor'
    events: [
      { name: 'signUp', from: 'visitor', to: 'user' }
      { name: 'bail', from: 'visitor', to: 'lostUser' }
      { name: 'closeAccount', from: 'user', to: 'visitor' }
      { name: 'promote', from: 'user', to: 'superUser' }
    ]

  initialize: =>
    _.extend @, new Backbone.Workflow(@)
  
  onSignUp: =>
    @set { signed_up_at: new Date() }


user = new User()
user.get('workflow_state') # => "visitor"

user.triggerEvent('signUp')
user.get('workflow_state') # => "user"
user.get('signed_up_at') # => Fri Feb 17 2012 17:07:41 GMT-0700 (MST)
```

Here's a more detailed rundown.

### Step 1: Extend Backbone.Model

Add this code to your `initialize` function:

```
initialize: =>
    _.extend @, new Backbone.Workflow(@)
```

### Step 2: Define Your Workflow

Create an object named `workflow`, as a property on your model, and define its events and states:

```
class User extends Backbone.Model
  workflow:
    initial: 'visitor'
    events: [
      { name: 'signUp', from: 'visitor', to: 'user' }
      { name: 'bail', from: 'visitor', to: 'lostUser' }
      { name: 'closeAccount', from: 'user', to: 'visitor' }
      { name: 'promote', from: 'user', to: 'superUser' }
    ]
```

### Step 3: Instantiate Your Model

The helper function `workflowState()` will always return your model's current state.

```
user = new User()
user.workflowState() # => "visitor"
```

### Step 5: Initiate a Transition

```
user.triggerEvent('signUp')
user.workflowState() # => "user"
```

## Binding Events To Transitions

Perhaps the most helpful feature of workflow.js is the ability to bind Backbone events to transitions in your views (or model). For example:

```
user.bind 'transition:from:visitor', -> alert("I'll only be a visitor for a moment longer!")
```

Or, its near equivalent:

```
user.bind 'transition:to:user', -> alert("I'm no longer just a visitor!")
```

Transitions are handled before (`transition:from`), and after (`transition:to`), the user defined call back.

## Customizations

Customize workflow.js by passing an attributes hash to the Backbone.Workflow constructor:

```
new Backbone.Workflow(@, { attrName: 'my_custom_db_field' })
```

Configurable parameters include:

* `attrName`: Backbone.Model attribute used to persist state at the server level. Default is "workflow_state"

## Multiple Workflows

To define multiple workflows, simply create a workflow object with a name of your choosing. Instantiate the workflows by passing a configuration array to the `Workflow` constructor.

Like this:

```
class DualPersonalityUser extends Backbone.Model
  jekyll_workflow:
    initial: 'happy'
    events: [
      { name: 'stub_toe', from: 'happy', to: 'hurting' }
      { name: 'get_massage', from: 'hurting', to: 'happy' }
    ]

  hyde_workflow:
    initial: 'catatonic'
    events: [
      { name: 'stub_toe', from: 'catatonic', to: 'ticked' }
      { name: 'get_massage', from: 'ticked', to: 'catatonic' }
    ]

  initialize: =>
    workflows = [
      { name: 'jekyll_workflow', attrName: 'jekyll_workflow_state' }
      { name: 'hyde_workflow', attrName: 'hyde_workflow_state' }
    ]
    _.extend @, new Backbone.Workflow(@, {}, workflows)
```

`attrName` refers to the Backbone fields you want to persist the workflow states to. You will also need to modify the methods you use to interact with the workflows.

To trigger an event, pass as the second argument the name of your workflow:

```
@user.triggerEvent('stub_toe', 'jekyll_workflow')
```

You'll handle transitions simililarly:

```
@user.on 'transition:from:jekyll_workflow:happy'
@user.on 'transition:to:jekyll_workflow:hurting'
```
