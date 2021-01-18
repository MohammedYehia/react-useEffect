
# The useEfect Hook
This built-in function used to perform a side effect operation, but what do we mean by this?
**Side Effect** is any thing that perform out side of our local function for example:
- Making network requests.
- Changing the DOM.
- Listening for KeyEvents.
- Timers.

#### The syntax
```js
//dependencyArray is optional depending on the case
useEffect(callbackFunction, [dependencyArray])
```

Example
```js
useEffect(()=> {
  //...do some side effects
})
```

Let's try and change the title of our page with our previous whenever the counter changes

changeing the title of the page will require us to access the dom which is a side effect so we will need to use`useEffect` 

```js
function Counter () {
  const [count, setCount] = React.useState(0);

  const incrementCounter = () => {
    setCount(c => c + 1);
  };
  
  React.useEffect(()=>{
    document.title=count;
    
    console.log("UseEffect is running");
  })
    
  return (
    <div>
      <p>The Counter state is: {count}</p>
      <button onClick={incrementCounter}>Add 1</button>
    </div>
  );
}
```

now with the example above whenever we change the `count` it will change the title of the page and print on the console.

Now let's add another state to the Component, we will add `Random Number` and display it on the screen.

```js
function Counter () {
  const [count, setCount] = React.useState(0);
  const [randomNum, setRandomNum] = React.useState(0);

  const incrementCounter = () => {
    setCount(c => c + 1);
  };
  
  const getRandomNumber = () => {
    setRandomNum(Math.random()*10);
  }
  
  React.useEffect(()=>{
  
    document.title=count;
    
    console.log("UseEffect is running");
  })
    
  return (
    <div>
      <p>The Counter state is: {count}</p>
      <p>Random Number: {randomNum}</p>
      <button onClick={incrementCounter}>Add 1</button>
      <button onClick={getRandomNumber}>Randomize</button>
    </div>
  );
}
```

now run the app and open the console then click on `Randomize` button, you will notice that the `useEffect` is working even without changing the count! 🤔, to solve this we will get back to our `dependencyArray` the second useEffect optional params.

dependency syntax
```js
React.useEffect(() => {
  // Will be invoked on the initial render
  // and when arg1 or arg2 changes.
}, [arg1, arg2])
```
so to solve this let's add the `count` as a condition for the useEffect to stop the useEffect from working unless we changed the `count`.

```js
function Counter () {
  const [count, setCount] = React.useState(0);
  const [randomNum, setRandomNum] = React.useState(0);

  const incrementCounter = () => {
    setCount(c => c + 1);
  };
  
  const getRandomNumber = () => {
    setRandomNum(Math.random()*10);
  }
  
  React.useEffect(()=>{
    document.title=count;
    
    console.log("UseEffect is running");
    //here we added count as a dependency
  }, [count])
  
    
  return (
    <div>
      <p>The Counter state is: {count}</p>
      <p>Random Number: {randomNum}</p>
      <button onClick={incrementCounter}>Add 1</button>
      <button onClick={getRandomNumber}>Randomize</button>
    </div>
  );
}
```

so the dependency array will help us to stop the useEffect from running unless the dependency array elements changed its values.


now let's assume that you were creating a timer using `setInterval` and this timer should update the state every 5 seconds what will happen if you moved from the component (unmount) before the timer timer return, you will get this Warning on the console `Can't perform a React state update on an unmounted component.`

To demonstrate this we will create main compoent and create a Counter component with a button to show/hide the Counter, after creating the app try to click on `Turn Counter off` and watch the warning on the console after 5 seconds, this could cause [memory leak](https://en.wikipedia.org/wiki/Memory_leak) and some performance issues.

```js

function App() {
  const [isOn, setIsOn] = React.useState(true);

  const turnCounterOff = () => {
    setIsOn(false);
  }

  return (
    <div className="App">
      {isOn ? <Counter /> : <p>Counter is off</p>}
      <button 
        onClick={turnCounterOff}>Turn Counter off</button>
    </div>
  );
}

function Counter() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    setInterval(() => {
      setCount((c) => c + 1);
    }, 5000);

    //we want to invoke this effect
    //only once on the initial render
  }, []);

  return <p>The Counter state is: {count}</p>;
}

```

to Solve this we will need the third part of our useEffect which is the `clean up` function which the return result from the call back function on useEffect which is going to work before unmounting the component and also runs before the re-render to clean up the previous effect

```js
React.useEffect(() => {
  return () => {
    // this will right before running
    // the new effect on a re-render AND
    // right before removing the component
    // from the DOM
  }
})
```

so let's clear the interval before moving to another component on our example

```js

function App() {
  const [isOn, setIsOn] = React.useState(true);

  const turnCounterOff = () => {
    setIsOn(false);
  }

  return (
    <div className="App">
      {isOn ? <Counter /> : <p>Counter is off</p>}
      <button 
        onClick={turnCounterOff}>Turn Counter off</button>
    </div>
  );
}

function Counter() {
  const [count, setCount] = React.useState(0);

  React.useEffect(() => {
    const id = setInterval(() => {
      setCount((c) => c + 1);
    }, 5000);

    return () => {
      clearInterval(id);
      console.log("UseEffect Cleared");
    }
  }, []);

  return <p>The Counter state is: {count}</p>;
}

```


now you will notice that the Warning has gone and interval stopped.

---

### Rules of Hooks:
- Only call Hooks from React functions.
- Only call Hooks at the top level, to be sure that Hooks are called in the same order each time a component renders.
    - avoid calling Hooks inside of loops, conditions, or nested functions.
- The Hook's name must start with `use` and follow camel case convention when creating custom Hooks(ex: useDimension, useLocalStorage) because React uses the linter rules to tell if a function is calling some hooks inside of it.
- React relies on the [order](https://reactjs.org/docs/hooks-rules.html#explanation) in which Hooks are called


### General notes:
- The useEffect will always run on the initial render.
- We can control the Effect using the second param
    - useEffect(cb) —> runs when any state changes
    - useEffect(cb, []) —> runs on mounting only
    - useEffect(cb, [prop1, state]) // runs when one of the dependency changes
- Cleanup function will always runs before useEffect on re-render
- If you used React classes and lifecycle methods then useEffect will replace the three methods `componentDidMount` , `componentDidUpdate` , `componentWillUnmount`.
- You can use [linter rules](https://reactjs.org/docs/hooks-rules.html#eslint-plugin) to help you with the dependency array(this is included by default with [CRA](https://create-react-app.dev/))

---

- You can't use `await` directly inside the useEffect callback

```js
// this will give you an error
React.useEffect( async () => {
  await api.request();
})
```
to solve this either you use a promise or create async function inside hte useEffect

```js

React.useEffect(() => {
  // Using async/await
  async function getData() {
    await api.request();
  } 
  
  getData();
  
  // OR using promise
  
  api.request().then(....)
  
})
```
---

#### Resources:
- [A Complete Guide to useEffect](https://iqkui.com/a-complete-guide-to-useeffect/), Dan Abramov
- [React Docs](https://reactjs.org/docs/hooks-effect.html)

<br/>
<br/>
---

***Exercise 1:***
This exercise will be another way to show you the benefit of returning the cleanup function:
1. Create a `count` state using `useState` hook.
2. Use any JSX tag to show the count value on the screen(ex: p, span,...).
3. Create a function called `incrementCount` and call `setCount` inside it which will increase the `count` state by one.(don't forget to pass a callback function to `setCount` so you could have access to the old count value).
4. Now call `useEffect` hook and inside it add an event listener on the `document` with the event `mousedown` and pass `incrementCount` function as an event handler.
5. Now that everything is set try to run your application and see what it is happening. (*[mousedown](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event)*)
    - This is happening because we are adding a new event listener with each re-render on the component and all of them will be triggered by the `mousedown` event.
6. Let's fix this by returning the cleanup function from our `useEffect` which will be used to remove our last event listener before re-rendering the component again.
7. Now it is fixed 🎉

---

***Exercise 2:***
Try to create this effect
*Hints*: 
* use the dom `mousemove` eventListener, and don't forget to remove it with clean up.
* you can measure the window `innerWidth` and if mouseX is bigger than the half then apply `tomato` color and if not apply `blue` color.

![](https://i.imgur.com/AizMCZA.gif)

---

***Exercise 3:***
use Yandex translate API to translate the input value from english to arabic by listining to the inpput event `onChange` so on every change of the value you should be sending a request to the api and you should clean up the previous requests to avoid any problem

(imagine this scenario you wrote the word `car` and wrote another character which change the word to `cart` and knowing that with every chrachter you are firing a new request to the api and for some reasons the second request(`cart`) got back with the response faster than the first one(`car`) then the first one will override the second one and the translation for `car` will be shown rather than the translation for `cart`)

---

***Exercise 4:***
use https://robohash.org/ api to generate unique images from any text you enter.

***Exercise 4:***
create a complete user profile from [randomUser](https://randomuser.me/) api or [jsonplaceholder](https://jsonplaceholder.typicode.com/users) with edit/delete ... and css style / loading ?
stretch goal: maybe you can use robohash.org api to generate random avatar for the user base on its username


---
