##

Restaurant Biscotte offers a cuisine based on fresh,

## [Header 1](#anchor-for-url-1)

![some alt](https://picsum.photos/id/1/200/300)

```js
import React from "react";
import ReactDOM from "react-dom";
import ReactMarkdown from "react-markdown";
import rehypeHighlight from "rehype-highlight";

ReactDOM.render(
  <ReactMarkdown rehypePlugins={[rehypeHighlight]}>
    {"# Your markdown here"}
  </ReactMarkdown>,
  document.querySelector("#content")
);
```

:::iframe{width="480" height="270" src="https://www.youtube.com/embed/aoLhACqJCUg?feature=oembed" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen}
Hello iframe
:::

Welcome to Biscotte restaurant! Restaurant Biscotte offers

- Follows [CommonMark](https://commonmark.org)
- Optionally follows [GitHub Flavored Markdown](https://github.github.com/gfm/)
- Renders actual React elements instead of using dangerouslySetInnerHTML
- Lets you define your own components (to render MyHeading instead of h1)
- Has a lot of plugins

## [Table of contents](#anchor-for-url-1)

react-markdown is a React component that converts Markdown text into the corresponding HTML code. It is built on remark, which is a Markdown preprocessor. react-markdown enables you to safely render markdown because it does not rely on the dangerouslySetInnerHTML prop.
`;

export { markdownString };
