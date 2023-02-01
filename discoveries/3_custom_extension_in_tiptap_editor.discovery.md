# How to create custom extension in TitTap Editor

Before we jump on the code, let me give you a brief idea of what is tiptap. It's a WYSIWYG editor that 
is based on top of Prose-mirror. It's a very powerful and highly customizable editor. It's very easy to integrate in react and 
other libraries. Though, they do provide a pro version of the editor, but it's not necessary to use it. you can create your own custom
extensions to build the features you want.

Now let's jump on the code. Here we are going to use React in the frontend (you can use any other framework you want). Let's install the dependencies first.

```bash
npm install @tiptap/react 
or
yarn add @tiptap/react
```

To get some basic features in the editor, you can install the starter pack as well.
```bash
npm install @tiptap/starter-kit
```

Now let's create a component for the editor.
```jsx
import React from 'react';
import { EditorContent, useEditor } from '@tiptap/react';
import StarterKit from '@tiptap/starter-kit';

const Editor = () => {
  
  const editor = useEditor({
    // Here you will pass the extensions you want to use in the editor.
    // for basic features you can use the starter kit.
    extensions: [
        StarterKit,
    ]
  })
  
  return (
    <EditorContent editor={editor} />
  )
}
```

In the official docs, they provide us a mention extension which can be used to mention someone (provide a list of suggestions).
Similarly, we are going to create a custom extension that will use a different key (say '#' )to show up a different suggestion list.
Firstly, we will create a file called `hastag.js` in the `src/components` dir.

```jsx
// few things that we need to import from tiptap.
import { Node, mergeAttributes } from '@tiptap/core';
import { PluginKey } from 'prosemirror-state';
import Suggestion from '@tiptap/suggestion';
import {ReactRenderer} from "@tiptap/react";
import tippy from "tippy.js";


const Hashtag = Node.create({
  name: 'hashtag', // you can call it anything you want.
  defaultOptions: {
    // This deafult option is used to configure the node view.
    HTMLAttributes: {},
    renderLabel: ({node}) => `#${node.attrs.label}`, // How the node should render in the text Editor.
    suggestion: {
      // This is the suggestion object that will be used to show the suggestions.
      pluginKey: new PluginKey('hashtag'),
      char: '#', // This is the key that will be used to show the suggestions.
    },
  },
  inline: true,
  group: 'inline',
  selectable: false,
  atom: true,
  parseHTML() {
    // This is used to parse the html and convert it to the node.
    return [{ tag: 'span[data-tag=\'hashtag\']'}] // Any span tag with attribute data-tag='hashtag' will be converted to the node.
  },
  renderHTML({ node, HTMLAttributes }) {
    // This is used to render the node in the HTML (not how it would look).
    return ['span', mergeAttributes(this.options.HTMLAttributes, HTMLAttributes, {
      'data-tag': 'hashtag',
      class: 'hashtag',
    }), `#${node.attrs.label}`]
  },
  renderText({ node }) {
    // How it would render as normal text. (There are 3 different ways to render and retrieve the data from the text editor)
    return `#${node.attrs.label}`
  },
  addKeyboardShortcuts() {
    // This is used to add keyboard shortcuts to the node.
    return {}
  },
  addProseMirrorPlugins() {
    // Responsible for registering the plugin to show the list on '#' key.
    Suggestion({
      editor: this.editor,
      items: ['one', 'two', 'three'], // can use async method to fetch list from the server.
      render: () => {
        // Going to use reactRenderer and tippy to render the suggestion box.
        let reactRenderer, popup;
        return {
          onStart: (props) => {
            popup = tippy('#editor-box', {
              // Option to configure tippy box.
              getReferenceClientRect: props.getReferenceClientRect,
              content: reactRenderer.element,
              placement: 'bottom-start',
              trigger: 'manual',
              interactive: true,
            })
          },
          onUpdate: () => {},
          onExit: () => {reactRenderer.destory()}
        }
      }
    })
  },
})
```

Still there is a lot of customizations that can be done. You can check the official repository for more details.
Now let's import this extension in the `Editor.js` file and use it.

```jsx
import React from 'react';
import { EditorContent, useEditor } from '@tiptap/react';
import StarterKit from '@tiptap/starter-kit';
import Hashtag from './hashtag';

const Editor = () => {
  
  const editor = useEditor({
    extensions: [
        StarterKit,
        Hashtag,
        // Another way to register the extension.
        // Hashtag.configure( options: {})
    ]
  })
  
  return (
    <EditorContent editor={editor} />
  )
}
```

Now you can use the hashtag extension in the editor. Hope you would have understood how we can create custom extensions in tiptap.