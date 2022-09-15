# Render MD files in React/NextJs.
Before we begin to learn how to render markdown in react we should know that markdown is a 
markup language. It is a way to style text on the web. You can control the display of the document.


In order to render markdown in react we need to install the following packages:
```text
1. react-markdown
2. remark-gfm (Optional)
```

Once you have all the packages installed you can simple use the **ReactMarkdown** component to render markdown in react.

```jsx
import ReactMarkdown from 'react-markdown'
const markdown = `# Hello World`
const App = () => {
  return (
    <ReactMarkdown>
      {markdown}
    </ReactMarkdown>
  )
}
```

In order to have more features in your markdown you can use the **remark-gfm** package. This package will allow you to use tables, strikethrough, task lists, and more in your markdown.

```jsx
<ReactMarkdown remarkPlugins={[gfm]}>
  {markdown}
</ReactMarkdown>
```

## Resources
- [React Markdown](https://www.npmjs.com/package/react-markdown)
- [Remark GFM](https://www.npmjs.com/package/remark-gfm)