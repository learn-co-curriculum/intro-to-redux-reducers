# Intro to Reducers

## Objectives
1. Describe how a reducer takes in an action and returns updated state
2. Determine why reducers must be pure functions
3. Organize reducers more cleanly using reducer composition

## You've got your actions ... now what?

As we discussed in the previous lab, Redux holds the state for our whole application. Our actions send data from our application to the store using the dispatch method.

```javascript
{
    type: ADD_TODO,
    text: 'Learn about reducers'
}

{
    type: ADD_MIXER,
    text: 'vermouth'
}
```

Actions simply convey that something happened, they don't tell the store how to update state based on those actions. **That's where reducers come in.**

## Reducers

A reducer is a pure function that takes in the previous state and the action and returns the next state.

What's a pure function? No, it's not a function that's gone organic, vegan, and gluten free.

A pure function:
- Has no side effects
- Has no mutations
- Its return value depends entirely on what is passed in
- It doesn't modify values passed do it

Pure functions don't make calls to an API, change the database or alter the arguments passed in. Given the same arguments, it would return the same value every time.

```javascript
const square = (x) => {
  return x ** 2
}
```
vs
```javascript
const updateDatabase = (x, y) => {
  x.name = y
}
```

As you can see, in the first function, it will always return the same output if given the same input. When we pass in ```2```, we'll always get ```4```. The second function is mutating ```x``` rather than returning an altered copy of the object.

Since Redux calls our reducers with an undefined state for the first time, we have an opportunity to define the initial state of our app. Using ES6 default arguments syntax, we set our initial state to an empty array. An empty array is effective for a list of items, but you could also initialize your state as an object or another initial value as needed.

```javascript
function recipeApp(state = [], action) {
  return state
}
```

Let's build a reducer that allows us to add recipes to our store.

We'll dispatch the following action to our reducer:

```javascript
{
    type: ADD_RECIPE,
    name: 'chicken soup',
    cookTime: '1 hour'
}
```

The reducer sees that the the action type is 'ADD_RECIPE', so it returns a new state object (using the spread operator) with that recipe added.

```javascript
function recipeApp(state = [], action) {
  switch (action.type) {
    case ADD_RECIPE:
      return [...state,
        {
          recipe: action.name,
          cookTime: action.cookTime
        }
    ]
      default:
        return state
    }
}
```

Beautiful! Our new state is
```javascript
[ { recipe: 'chicken soup', cookTime: '1 hour' } ]
```

There are two critical things to note:

1. We don't mutate or alter the state. One of first things we mentioned was that reducers return a new state object, they don't modify old state. We create a copy of our state using the spread operator. We passed it the original state array and added our new recipe object.

2. We return the previous state as a default action. If a reducer receives an unknown action, it must return the previous state. Reducers cannot return undefined.


Now we'll update our reducer to prepare it to receive multiple actions and multiple pieces of state. Our new initial state with have an empty array for ingredients and an empty array for recipes.

```javascript
function recipeApp({ ingredients: [], recipes: [] }, action) {
  switch (action.type) {
    case ADD_PANTRY_INGREDIENT:
      return Object.assign({}, state, {
        ingredients: [
          ...state.ingredients,
          {
            ingredient: action.name,
            quantity: action.quantity
          }
        ]
      })
    case ADD_RECIPE:
      return Object.assign({}, state, {
        recipes: [
          ...state.recipes,
          {
            recipe: action.name
            cookTime: action.cookTime
          }
        ]
      })

    case CHANGE_INGREDIENT_QUANTY:
      return Object.assign({}, state, {
        ingredients: state.ingredients.map((ingredient, index) =>{
          if(index === action.index) {
            return Object.assign({}, ingredient, {
              quantity: ingredient.quantity - action.quantity
            })
          }
          return ingredient
      })
  })
  default:
    return state
  }
}
```

Note that we used `Object.assign({})` to create a new object rather than mutate the original state. We pass it an empty object, fill it in with the original state of ingredients, and then add a new ingredient and quantity based on the data passed along from the action.

## Reducer Composition
Great, we've set up a reducer that establishes our initial state as an object with an empty ingredients array and an empty recipe array. We're able to add ingredients to our pantry with the `ADD_PANTRY_INGREDIENT` action. We can add a recipe with the `ADD_RECIPE` action, and finally we can change the quantity of our ingredients in our pantry if we use `CHANGE_INGREDIENT_QUANTY`.

Our reducer is getting pretty long. Who knows how many more actions we'll have ... cooking ingredients? Removing expired pantry ingredients? Will our reducer reach into the ether?

...no!

That's where reducer composition comes in. Our current reducer is responsible for the ingredients array and the recipes array, both of which are updated independently. Let's split those out into separate functions

```javascript
function recipes(state = [], action) {
  switch (action.type) {
    case ADD_RECIPE:
      return [
          ...state.recipes,
          {
            recipe: action.name
            cookTime: action.cookTime
          }
        ]
      })
  }
  default:
    return state
}

function ingredients(state = [], action) {
  switch (action.type) {
    case ADD_PANTRY_INGREDIENT:
      return [
          ...state,
          {
            ingredient: action.name,
            quantity: action.quantity
          }
        ]
      })

      case CHANGE_INGREDIENT_QUANTY:
        return state.map((ingredient, index) =>{
            if(index === action.index) {
              return Object.assign({}, ingredient, {
                quantity: ingredient.quantity - action.quantity
              })
            }
            return ingredient
        })
    })
    default:
      return state
    }
}
```

**Each reducer is in charge of managing it's own portion of the state.** Now we can create a single reducer function that combines our individual reducers:

```javascript
function kitchenApp(state = {}, action) {
  return {
    recipes: recipes(state.recipes, action),
    ingredients: ingredients(state.ingredients, action)
  }
}
```




## Resources

- [Redux](http://redux.js.org/docs/basics/Reducers.html)
