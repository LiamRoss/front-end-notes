# Ducks Architecture

Notes on a version of Ducks architecture for Redux, and complications within the file structure, as well as when using it with TypeScript.

---

## Basics

The idea behind Ducks is to compartmentalize your code, so that rather then grouping "Rails" style, by components, reducers, actions, etc... you are grouping in a way that mirrors the structure of your application. There are a few things to keep in mind before adopting a ducks-style:

1. Are you willing to do some extra work?  
  > Ducks requires a style of exporting reducers and back to the store level in the folder structure that is not required if you simply have all the reducers in a folder together à la "Rails" style. If your application does not require ducks, it is probably better to begin by simply following the "Rails" style.

2. Does your app require ducks?  
  > If your application matches any (or multiple) of these, ducks may be for you:
  * Your folders (reducers, components, etc.) are full of files, or scrolling is required to find files within folders
  * You are planning your application structure to accommodate scalability
  * You are building a segmented app, and plan on having highly-contained components within it
  
  Ducks is helpful for organizing code based on functionality, allowing for a scalable structure that accommodates highly-segmented applications, and provides organized folder architecture in large-scale applications.

3. How much are you willing to commit to ducks?  
  > In order to get the full benefits of ducks, while avoiding any unnecessary wiring, it is advised you follow (at the bare minimum) the [original ducks proposal](https://github.com/erikras/ducks-modular-redux), which can also be found at the bottom under [Further reading.](#further-reading)

---

## Ducks Structure

So you've decided to use ducks? There are three schools of thought as far as structure go.

Before I delve into these ideas, take this example application:

The application is comprised of a **navigation bar**, and **two alternating sub-applications**. The navigation bar has **contextual buttons** based on the sub-application being viewed, and has a **menu button** that opens a **panel** with selectors to choose which sub-application to view. For this example, lets assume FirstApp represents an administrative view of all users and their groups. Additionally, lets assume that each individual user component will contain the buttons to edit and delete that specific user.

From this information, we can determine a basic state of the following:

```javascript
{
  NavBar: { 
    buttons: [
      addUser
    ],
    panelIsOpen: false
  },
  FirstApp: {
    users: [
      1: {
        id: '1',
        firstName: 'John',
        secondName: 'Doe'
      },
      2: {
        id: '2',
        firstName: 'Marie',
        secondName: 'Curie'
      },
      3: {
        id: '3',
        firstName: 'Albert',
        secondName: 'Einstein'
      },
    ],
    groups: [
      group1: {
        id: 'group1',
        name: 'Famous People'
        members: [
          '2', 
          '3'
        ]
      },
      group2: {
        id: 'group2',
        name: 'John Does'
        members: [
          '1'
        ]
      }
    ]
  },
  SecondApp: {
    // empty while FirstApp is being viewed
  }
}
```

Now this may seem complicated but it's important to have a sufficiently complex app to demonstrate ducks. From this state you can notice a few things:

1. It likely uses redux's combineReducers to combine `NavBar`, `FirstApp`, and `SecondApp`.
2. `FirstApp` will likely render each user and group as their own components, meaning there will be components for the application itself, as well as each user and group.
3. While the `NavBar` component contains the button to add a user, the options to edit and delete users will be on the individual user components, meaning there will be actions and reducers associated with those components.

Now for the options:

### Option 1: Mimic the State

This method of ducks mimics the form of the application in order to compartmentalize all the components. Using this method, components and their logic will be self-contained, making it easy to move around and logical as to where everything is located. 

For the state above, a likely folder structure would be as follows:

```
src
├── middleware
├── App
|   ├── index.js
|   ├── App.js
|   ├── NavBar
|   |   ├── index.js
|   |   ├── Buttons (expanded)
|   |   |   ├── index.js
|   |   |   ├── component.js
|   |   |   ├── container.js
|   |   |   ├── actions.js
|   |   |   └── reducers.js
|   |   └── NavPanel
|   ├── FirstApp
|   |   ├── index.js
|   |   ├── Users
|   |   └── Groups
|   └── SecondApp
|       ├── index.js
|       └── ...Other Components
├── index.js
├── store.js
└── routes.js
```

In the above example, buttons is expanded to show the internals of a ducks folder. As you can see, there is an index.js file at every level within the folder hierarchy which ensures exports all the way up to the store level. Any time the state is nested, similar nesting is found within the folder structure. At the end, all of the reducers will be combined within the store, resulting in the three main components of the application being at the same level in the state. 

#### Option 1 Pros:

* Logical folder structure, matches up with the state
* Highly-segmented, allows for easy transport and packaging of individual components and their logic

#### Option 1 Cons:

* Does not separate view and logic. While not terrible, this does cause issues if you ever wanted to use the redux logic and state management with a different view (ex. react-native), or if you move away from redux at any point
* Could result in a deep state and deep folder nesting

### Option 2: Flat App Folder

This method of ducks keeps all components at the same level within the apps folder. This stops the folder structure from becoming too nested, and allows all component folders to be viewed at once without digging into the folder tree. 

For the state above, a likely folder structure would be as follows:

```
src
├── middleware
├── App
|   ├── index.js
|   ├── App.js
|   ├── NavBar
|   ├── Buttons (expanded)
|   |   ├── index.js
|   |   ├── component.js
|   |   ├── container.js
|   |   ├── actions.js
|   |   └── reducers.js
|   ├── NavPanel
|   ├── FirstApp
|   ├── Users
|   ├── Groups
|   └── SecondApp
├── index.js
├── store.js
└── routes.js
```

In this option, you see a flatter representation of all the components, which are no longer nested to match the state.

#### Option 2 Pros:

* Easier to connect components, as any far-removed connections between components only has to import from the main App folder (no nesting)  
  > ex: a connection between `Buttons` and `FirstApp` only needs to be:
  > `../FirstApp/something`  
  > instead of  
  > `../../FirstApp/something`  
  > *Obviously this example is even more meaningful if further nesting occurs*

* Easier to find all components, as none will be nested deeply in their parent components

#### Option 2 Cons:

* Makes it more difficult to maintain the same state indicated above, as now components that were once nested must now have their reducers imported into their parent component.  
  > This ties to the idea of reducer composition. If you think about the state, each reducer can send a portion of  the state off to be handled by a sub-reducer, which then returns it's portion of the state. While combine reducers serve to combine multiple reducers in a flat formation, reducer composition gives the ability to work with nesting.  

* If you choose to adapt the state to fit the folder structure, see [Fixing Option 2](#fixing-option-2) below. This will result in the state losing some of the inherent meaning that exists due to the nesting.

##### Fixing Option 2

In order to fix the cons for option two, you can restructure your state so that you simply use a single combine reducer across all components. While this may no longer be as intuitive as the nested state, it is flatter and allows for simple importing across all of the components. After doing this, the state will likely look as follows:

```javascript
{
  buttons: [
    addUser
  ],
  panelIsOpen: false,
  users: [
    1: {
      id: '1',
      firstName: 'John',
      secondName: 'Doe'
    },
    2: {
      id: '2',
      firstName: 'Marie',
      secondName: 'Curie'
    },
    3: {
      id: '3',
      firstName: 'Albert',
      secondName: 'Einstein'
    },
  ],
  groups: [
    group1: {
      id: 'group1',
      name: 'Famous People'
      members: [
        '2', 
        '3'
      ]
    },
    group2: {
      id: 'group2',
      name: 'John Does'
      members: [
        '1'
      ]
    }
  ]
}
```

### Option 3: True Ducks

This method of ducks separates the view and state components, leading to a slightly more complicated folder structure, but allowing the separation of the redux logic from react view. 

For the state above, a likely folder structure would be as follows:

```
src
├── state
|   ├── index.js
|   ├── middleware
|   ├── Buttons (expanded)
|   |   ├── index.js
|   |   ├── actions.js
|   |   └── reducers.js
|   ├── NavPanel
|   ├── Users
|   ├── Groups
|   └── ... SecondApp stuff
├── view
|   ├── App.js
|   ├── NavBar
|   ├── Buttons (expanded)
|   |   ├── component.js
|   |   └── container.js
|   ├── NavPanel
|   ├── FirstApp
|   ├── Users
|   ├── Groups
|   └── SecondApp
├── index.js
├── store.js
└── routes.js
```

In this option, you see a separation between state and view. In more extreme versions of this implementation, you could see the store residing inside state, and various other scenarios. See [Further reading](#further-reading) for examples of this implementation

#### Option 3 Pros:

* Ease of wiring up components
* Ease of separation between state and view logic, namely React and Redux live in different folders, which allows for easy technology changes and easy implementation with new views such as mobile via react native

#### Option 3 Cons:

* Difficult to connect dispatch to props, as the action creators are in the state folder and the components are in the view folder
* Not as simple to view, as not all components and state files are found in the same folder

---

Hopefully that gives some insight into the ducks structure. Personally, as long as you are planning on maintaining react and redux for your application, I think any of these could suit your use. My personal preference is for the first option. Although it is not true ducks, it allows for better compartmentalization, and is the easiest in terms of wiring up imports.

---

## Further reading:

[Original Ducks Proposal.](https://github.com/erikras/ducks-modular-redux)

[Medium Article.](https://medium.freecodecamp.com/scaling-your-redux-app-with-ducks-6115955638be)

[Example Repository 1.](https://github.com/FortechRomania/react-redux-complete-example)

[Example Repository 2.](https://github.com/jthegedus/re-ducks-examples)

