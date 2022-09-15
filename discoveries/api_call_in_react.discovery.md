# How to make an async call using React useEffect Hook

In this example, we are making an async call using the useEffect Hook.
The Effect Hook lets you perform side effects in function components. Some examples of “side effects” or just “effects” can be data fetching, manually changing the DOM or setting up a subscription.

With the useEffect Hook we can tell React that after every render, the callback passed to this useEffect function as its first parameter (“effect”) should be called. React will remember this callback and call it later after performing the DOM updates.

Before Hooks, we only could manage this effects within the lifecycle methods of class components: componentDidMount, componentDidUpdate and componentWillUnmount. The logic of an effect ended up being spread out throughout the different lifecycle methods.

The example of the previous app implemented without Hooks:
By default, it runs after every render but we can customize it with the second param of the useEffect function. As a second argument, the useEffect function accepts an array that allow us to tell React when we want our effect to be called.

After a render and before calling an effect, React will compare the array of values defined in the second parameter of the effect with the array defined in the same effect from the previous render. React will only call the effect when any value of the array has changed since the previous render.

In the previous example, we don’t need to update the document title (our effect) after every render but only when the state variable name has changed its value since the previous render. That’s why we pass an array with the value of name as the second parameter:
This is how you make an async call using the useEffect Hook:

[Official Website of ReactJs](https://reactjs.org/)


```js
import { signal } from '@preact/react-signal';
const Home = () => {
  const counter = singal(0);
  return (
    <>
      <h2>Here is the value of singal {counter.value}</h2>
    </>
  )
}

export default Home;
```