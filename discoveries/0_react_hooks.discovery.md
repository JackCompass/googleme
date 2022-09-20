# All about react hooks

Hooks were first introduced in React 16.8. And they're great because they let you use more of React's features – like managing your component's state, or performing an after effect when certain changes occur in state(s) without writing a class.

In this article, you will learn how to use Hooks in React and how to create your own custom Hooks. Just keep in mind that you can use hooks solely for functional components.

What is the useState Hook?
The state of your application is bound to change at some point. This could be the value of a variable, an object, or whatever type of data exists in your component.

To make it possible to have the changes reflected in the DOM, we have to use a React hook called useState. It looks like this:


export default App;
Let's look a bit more closely at what's going on in the code above.

import { useState } from "react";
To be able to use this hook, you have to import the useState hook from React. We are using a functional component called app.

After that, you have to create your state and give it an initial value (or initial state) which is "Ihechikara". The state variable is called name, and setName is the function for updating its value.

Having a good understanding of some of the ES6 features will help you grasp the basic functionalities of React. Above, we used the destructuring assignment to assign an initial name value to the state in useState("Ihechikara").

Next, the DOM has a paragraph containing the name variable and a button which fires a function when clicked. The changeName() function calls the setName() function which then changes the value of the name variable to the value passed in to the setName() function.

The values of your state must not be hard coded. In the next section, you will see how to use the useState hook in forms.

For React beginners, note that you create your functions and variables before the return statement.

How to Use the useState Hook in Forms
This section will help you understand how to create state values for your forms and update them when you need to do so. The process is not so different from what we saw in the previous section.

As always, import the useState hook:

import { useState } from "react";
We will create the initial state like we did before. But in this case it is going to be an empty string since we are dealing with the value of an input element. Hard coding the value means the input will have that value whenever the page is reloaded. That is:

Now that we've created the state, let's create the input element in the DOM and assign the name variable as its initial value. It looks like this:

You will notice that we didn't create a function above the return statement to update the value of the state – but it is still okay if you decide to use that method.

Here, we use the onChange event listener which waits for any value change in the input field. Whenever there is a change, an anonymous function (which takes in the event object as a parameter) is fired which in turn calls the setName() function to update the name variable with the current value of the input field.

Here's what the final code looks like: