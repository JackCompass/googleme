# Smart implementation of SWR hook in Next.js

When it comes to data fetching on the client side in React or Next applications, one of the popular choices is the SWR (stale-while-revalidate) hook coming directly from the Vercel team. Those who are familiar with it knows that it is simple yet powerful for fetching data, caching it, and revalidating it in the background.

But have you ever wondered how does it work under the hood? Does it manage multiple states for you? How does it manage to handle errors and loading states? How we can use suspense with it? In this article, we will try to understand the SWR hook implementation and how it works in frontend applications.

## Let's start with a headless component

```jsx
const useCustomSWR = (key: string, fetcher: () => Function) => {
  return {};
};
```

Let's start with a headless component that will take a key and a fetcher function. We will talk about the key later but fetcher function is responsible for fetching the data from the server. It can be any function that returns a promise. For example, it can be an API call or a database query.

## The magic happens here

Instead of creating a try-catch block and then calling a fetcher function, we will create a Map outside the component to store the response data against the provided key. This is where the key becomes important as it can be used to identify the data in the Map.

In the first attempt we will build it with all the three different states: data, error, and loading.

```jsx
const dataMap = new Map(); // Could use a WeakMap or localstorage.

const useCustomSWR = (key: string, fetcher: () => Function) => {
  const [fetcherData, setFetcherData] = useState(null);
  const [error, setError] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  const fetchData = async () => {
    setIsLoading(true);
    setError(null);

    // Check if data is already in the map
    if (dataMap.has(key)) {
      setFetcherData(dataMap.get(key));
      setIsLoading(false);
      return;
    }

    try {
      const data = await fetcher();
      dataMap.set(key, data); // Store the fetched data in the map
      setFetcherData(data);
    } catch (err) {
      setError(err);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [key]); // Re-fetch data if the key changes
  return {
    data: fetcherData,
    error,
    isLoading,
    revalidate: fetchData,
  };
};
```

With this implementation, we now understand why the swr hook relies on the key. But can we improve it futher? Yes, we can. We can remove all the states from the component and let the Suspense and ErrorBoundary handle it for us. This will make the component cleaner and more readable.

## Using Suspense and ErrorBoundary

To use Suspense and ErrorBoundary, we need to make a few changes to our implementation. We will remove the loading and error states from the component and let the Suspense and ErrorBoundary handle it for us.

```jsx
const dataMap = new Map();

const useCustomSWR = (key: string, fetcher: () => Function) => {
  const data = dataMap.get(key);
  const fetchData = async () => {
    // Will need to throw a promise to trigger Suspense.
    throw new Promise(async (resolve, reject) => {
      const value = await fetcher();
      dataMap.set(key, value);
      resolve(value);
    });
  };
  if (!data && key) {
    fetchData();
  }

  return {
    data: data,
    revalidate: fetchData,
  };
};
```

Now, we can use this hook in our components and wrap them with Suspense and ErrorBoundary to handle loading and error states.

```jsx
import { Suspense } from "react";
import { ErrorBoundary } from "./ErrorBoundary"; // Assume you have an ErrorBoundary component
import { useCustomSWR } from "./useCustomSWR"; // The custom SWR hook we created
const MyComponent = () => {
  const { data, revalidate } = useCustomSWR("/api/data", fetchData);

  return (
    <div>
      <h1>Data</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
      <button onClick={revalidate}>Revalidate</button>
    </div>
  );
};

const App = () => {
  return (
    <ErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <MyComponent />
      </Suspense>
    </ErrorBoundary>
  );
};
``` 


Neat! Right, Question arises what does swr hook uses to store the data? In our case, we used a Map but in the real world, it uses a more sophisticated caching mechanism that can handle multiple keys and values. It also has built-in support for revalidation, deduplication, and other features that make it a powerful tool for data fetching in React applications.

## Conclusion
We explored how the SWR hook works under the hood and how we can implement a custom version of it. Important thing is you can throw a Promise (that can trigger Suspense in React) as well in Js, but you would have to add a lot more checks to confirm if it's a Promise and not an error.