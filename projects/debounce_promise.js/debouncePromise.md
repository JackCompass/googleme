# **Debounce Promise (A NPM Package)**
Debounce Promise is a helpful utility library that allows you to return promises from a debounced function. 
It can be used or integrated with any Node.js project. It's completely written in Js and has no dependencies.

Generally we use debounce function (from lodash) to make API calls or do some async task. These debounce method return nothing. They just execute you function after the delay.
But sometimes we need the data back from these debounced functions. In such cases you could use this debounce promise.
On use of it you will get a promise back from the debounced function. And the promise will either resolve or reject depending on the result of the debounced function.

In order to make it accessible I have also released it as an npm package. You can find it [here](https://www.npmjs.com/package/debounce-promise-latest).

## Installation
You can install it using npm or yarn.
```bash
npm install debounce-promise-latest
```
or
```bash
yarn add debounce-promise-latest
```

### Look at the documentation for more details.

### Issues/Feedback
If you have any issues or feedback, please create an issue on the [GitHub repo](https://github.com/JackCompass/debounce-promise/issues)