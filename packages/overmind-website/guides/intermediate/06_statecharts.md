# Statecharts

Just like [operators](/guides/intermediate/04_goingfunctional) is a declarative abstraction over plain actions, **statecharts** is a declarative abtraction over a whole Overmind configuration. That means you will define your charts by:

```js
const configWithStatechart = statechart(config, chart)
```

There are several benefits to using a statechart:

1. You will have a declarative description of what actions should be available in certain states of the application
2. Less bugs because an invalid action will not be executed if called
3. You will be able to implement and test an interaction flow without building the user interface for it
4. Your state definition is cleaned up as your **isDoingSomething** types of state is no longer needed
5. You have a tool to do "top down" design and implementation instead of "bottom up"

You can basically think of a statechart as a way of limiting what actions are available to be executed in certain states. This concept is very old and was originally used to design machines where the user was exposed to all points of interaction, all buttons and switches, at any time. Statecharts would help make sure that at certain states certain buttons and switches would not operate.

A simple example of this is a Walkman. When the Walkman is in a **playing** state you should not be able to hit the **eject** button. On the web this might seem unnecessary as the points of interaction is dynamic. We simply hide and/or disable points of interaction. But this is the exact problem. It is fragile. It is fragile because the visual design is all you depend on to prevent logic from running when it should not. A statechart is a much more resiliant way to ensure what logic can actually run in any given state.

## Defining a statechart

Let us imagine that we have a login flow. This login flow has 4 different **transition states**:

1. **LOGIN**. We are at the point where the user inserts a username and password
2. **AUTHENTICATING**. The user has submitted
3. **AUTHENTICATED**. The user has successfully logged in
4. **ERROR**. Something wrong happened when 

Let us do this properly and design this flow "top down":

```marksy
h(Example, { name: "guide/statecharts/define.ts" })
```

As you can see we have defined what transition states our login flow can be in and what actions we want available to us in each transition state. If the action points to **null** it means we stay in the same transition state. If it points to an other transition state, the execution of that action will cause that transition to occur.

Since our initial state is **LOGIN**, a call to actions defined in the other transition states would simply be ignored.

## Conditions

In our chart above we let the user log in even though there is no **username** or **password**. That seems a bit silly. In statecharts you can define conditions. These conditions receives the state of the configuration and returns a boolean.

```marksy
h(Example, { name: "guide/statecharts/condition.ts" })
```

Now the **login** action can only be executed, causing a transition to the new transition state if there is a username and password inserted.

## State
Our initial state defined for this configuration is:

```marksy
h(Example, { name: "guide/statecharts/state.ts" })
```

As you can see we have no state indicating that we have received an error, like **hasError**. We do not have **isLoggingIn** either. There is no reason, because we have our transition states. That means the configuration is populated with some additional state by the statechart. It will actually look like this:

```js
{
  username: '',
  password: '',
  user: null,
  authenticationError: null,
  state: ['LOGIN'],
  actions: {
    changeUsername: true,
    changePassword: true,
    login: false,
    logout: false,
    tryAgain: false
  }
}
```

The **state** state is the current transition state. It is defined as an array because later you will see that we can have nested charts.

The **actions** state is a derived state. That means it automatically updates based on the current state of the chart. This is helpful for your UI implementation. It can use it to disable buttons etc. to help the user understand when certain actions are possible.

There is also a third derived state called **matches**. This derived state returns a function that allows you to figure out what state you are in:

```marksy
h(Example, { name: "guide/statecharts/matches.ts" })
```

## Actions

Our actions are defined something like:

```marksy
h(Example, { name: "guide/statecharts/actions.ts" })
```

What to take notice of here is that with traditional Overmind we would most likely just set the **user** or the **authenticationError** directly in the **login** action. That is not the case here because our actions are the triggers for transitions. That means whenever we want to deal with transitions we create an action for it, even completely empty actions like **tryAgain**. This simplifies our chart definition and also we avoid having a generic **transition** action that would not be typed in TypeScript etc.

## Nested statecharts

With a more complicated UI we can create nested statecharts. An example of this would be a workspace UI with different tabs. You only want to allow certain actions when the related tab is active. Let us explore an example:

```marksy
h(Example, { name: "guide/statecharts/nested.ts" })
```

What to take notice of in this example is that we simply **spread** a chart into an existing transition state, effectively nesting them. The nested charts has access to the same actions and state as the parent chart.

In this example we also took advantage of the **entry** and **exit** hooks of a transition state. These also points to actions. When a transition is made into the transition state the **entry** will run. This behavior is nested. When an exit hook exists and a transition is made away from the transition state it will also run. This behivor is also nested of course.

## Summary

The point of statecharts in Overmind is to give you an abstraction over your configuration that ensures the actions can only be run in certain states. Just like operators you can choose where you want to use it. Maybe only one namespace needs a statechart, or maybe you prefer using it on all of them. The devtools has its own visualizer for the charts, which allows you to implement and test them without implementing any UI.