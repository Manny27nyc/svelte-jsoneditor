# svelte-jsoneditor

A web-based tool to view, edit, format, transform, and validate JSON

The library is written with Svelte, but can be used in any framework (React, Vue, Angular, plain JavaScript).

![JSONEditor screenshot](https://raw.githubusercontent.com/josdejong/svelte-jsoneditor/main/misc/jsoneditor_screenshot.png)

<!-- TODO: describe features -->

## Install

Install via npm:

```
npm install svelte-jsoneditor
```

## Use

### Examples

- Svelte examples: [/src/routes/examples](/src/routes/examples)
- Plain JavaScript examples: [/examples/browser](/examples/browser)
- React example: [https://codesandbox.io/s/svelte-jsoneditor-react-59wxz](https://codesandbox.io/s/svelte-jsoneditor-react-59wxz)

### SvelteKit setup

There is currently an issue in SvelteKit with processing some dependencies (more precisely: Vite used by SvelteKit). `svelte-jsoneditor` depends on some libraries that hit this issue. To work around it, each of these dependencies needs to be listed in the configuration. Without the workaround, you'll see errors like "ReferenceError: module is not defined" (for `debug`, `ajv`, `ace-builds`, etc.).

In your SvelteKit configuration file `svelte.config.js`, add the list with dependencies `viteOptimizeDeps`, available in the `svelte-jsoneditor/config.js`, and use that in the configuration of vite (`config.kit.vite.optimizeDeps.include`):

```js
// svelte.config.js

// ...
import { viteOptimizeDeps } from 'svelte-jsoneditor/config.js'

const config = {
  // ...

  kit: {
    // ...

    vite: {
      optimizeDeps: {
        include: [...viteOptimizeDeps]
      }
    }
  }
}

// ...
```

### Svelte usage

Create a JSONEditor with two-way binding `bind:json`:

```html
<script>
  import { JSONEditor } from 'svelte-jsoneditor'

  let content = {
    text: undefined, // used when in code mode
    json: {
      array: [1, 2, 3],
      boolean: true,
      color: '#82b92c',
      null: null,
      number: 123,
      object: { a: 'b', c: 'd' },
      string: 'Hello World'
    }
  }
</script>

<div>
  <JSONEditor bind:content />
</div>
```

Or one-way binding:

```html
<script>
  import { JSONEditor } from 'svelte-jsoneditor'

  let content = {
    text: undefined, // used when in code mode
    json: {
      greeting: 'Hello World'
    }
  }

  function handleChange(updatedContent) {
    // content is an object { json: JSON } | { text: string }
    console.log('onChange: ', updatedContent)
    content = updatedContent
  }
</script>

<div>
  <JSONEditor {content} onChange="{handleChange}" />
</div>
```

### Standalone bundle (use in React, Vue, Angular, plain JavaScript, ...)

The library provides a standalone bundle of the editor which can be used in any browser environment and framework. In a framework like React, Vue, or Angular, you'll need to write some wrapper code around the class interface.

Browser example loading the ES module:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>JSONEditor</title>
  </head>
  <body>
    <div id="jsoneditor"></div>

    <script type="module">
      import { JSONEditor } from 'svelte-jsoneditor/dist/jsoneditor.js'

      let content = {
        text: undefined,
        json: {
          greeting: 'Hello World'
        }
      }

      const editor = new JSONEditor({
        target: document.getElementById('jsoneditor'),
        props: {
          content,
          onChange: (updatedContent) => {
            // content is an object { json: JSON } | { text: string }
            console.log('onChange', updatedContent)
            content = updatedContent
          }
        }
      })

      // use methods get, set, update, and onChange to get data in or out of the editor.
      // Use updateProps to update properties.
    </script>
  </body>
</html>
```

## API

### constructor

Svelte component:

```html
<script>
  import { JSONEditor } from 'svelte-jsoneditor'
</script>

<div>
  <JSONEditor {content} />
</div>
```

JavasScript class:

```js
import { JSONEditor } from 'svelte-jsoneditor/dist/jsoneditor.js'

const editor = new JSONEditor({
  target: document.getElementById('jsoneditor'),
  props: {
    content,
    onChange: (updatedContent) => {
      // content is an object { json: JSON } | { text: string }
      console.log('onChange', updatedContent)
    }
  }
})
```

### properties

- `content: { json: JSON } | { text: string }` Pass the JSON contents to be rendered in the JSONEditor. Contents is an object containing a property `json` and `text`. Only one of the two must be defined. In case of `tree` mode, `json` is used. In case of `code` mode, `text` is used.
- `mode: 'tree' | 'code'`. Open the editor in `'tree'` mode (default) or `'code'` mode.
- `mainMenuBar: boolean` Show the main menu bar. Default value is `true`.
- `navigationBar: boolean` Show the navigation bar with, where you can see the selected path and navigate through your document from there. Default value is `true`.
- `readOnly: boolean` Open the editor in read-only mode: no changes can be made, non-relevant buttons are hidden from the menu, and the context menu is not enabled. Default value is `false`.
- `indentation: number` Number of spaces use for indentation when stringifying JSON.
- `validator: function (json): ValidationError[]`. Validate the JSON document.
  For example use the built-in JSON Schema validator powered by Ajv:

  ```js
  import { createAjvValidator } from 'svelte-jsoneditor'

  const validator = createAjvValidator(schema, schemaRefs)
  ```

- `onError(err: Error)`.
  Callback fired when an error occurs. Default implementation is to log an error in the console and show a simple alert message to the user.
- `onChange({ json: JSON | undefined, text: string | undefined})`.
  Callback which is invoked on every change made in the JSON document.
- `onChangeMode(mode: string)`. Invoked when the mode is changed.
- `onClassName(path: Array.<string|number>, value: any): string | undefined`.
  Add a custom class name to specific nodes, based on their path and/or value.
- `onRenderMenu(mode: string, items: Array) : Array | undefined`.
  Callback which can be used to make changes to the menu items. New items can
  be added, or existing items can be removed or reorganized. When the function
  returns `undefined`, the original `items` will be applied.

  A menu item can be one of the following types:

  - Button:

    ```ts
    interface MenuButtonItem {
      onClick: () => void
      icon?: FontAwesomeIcon
      text?: string
      title?: string
      className?: string
      disabled?: boolean
    }
    ```

  - Separator (gray vertical line between a group of items):

    ```ts
    interface MenuSeparatorItem {
      separator: true
    }
    ```

  - Space (fills up empty space):

    ```ts
    interface MenuSpaceItem {
      space: true
    }
    ```

- `onFocus()` callback fired when the editor got focus.
- `onBlur()` callback fired when the editor lost focus.

### methods

- `get(): { json: JSON } | { text: string }` Get the current JSON document.
- `set(content: { json: JSON } | { text: string })` Replace the current content. Will reset the state of the editor. See also method `update(content)`.
- `update(content: { json: JSON } | { text: string })` Update the loaded content, keeping the state of the editor (like expanded objects). You can also call `editor.updateProps({ content })`. See also method `set(content)`.
- `patch(operations: JSONPatchDocument)` Apply a JSON patch document to update the contents of the JSON document. A JSON patch document is a list with JSON Patch operations.
- `updateProps(props: Object)` update some or all of the properties. Updated `content` can be passed too; this is equivalent to calling `update(content)`. Example:

  ```js
  editor.updateProps({
    readOnly: true
  })
  ```

- `expand([callback: (path: Path) => boolean])` Expand or collapse paths in the editor. The `callback` determines which paths will be expanded. If no `callback` is provided, all paths will be expanded. It is only possible to expand a path when all of its parent paths are expanded too. Examples:
  - `editor.expand(path => true)` expand all
  - `editor.expand(path => false)` collapse all
  - `editor.expand(path => path.length < 2)` expand all paths up to 2 levels deep
- `transform({ id?: string, onTransform?: ({ operations: JSONPatchDocument, json: JSON, transformedJson: JSON }) => void, onClose?: () => void })` programmatically trigger clicking of the transform button in the main menu, opening the transform model. If a callback `onTransform` is provided, it will replace the build-in logic to apply a transform, allowing you to process the transform operations in an alternative way. If provided, `onClose` callback will trigger when the transform modal closes, both after the user clicked apply or cancel. If an `id` is provided, the transform modal will load the previous status of this `id` instead of the status of the editors transform modal.
- `scrollTo(path: Array.<string|number>)` Scroll the editor vertically such that the specified path comes into view. The path will be expanded when needed.
- `focus()`. Give the editor focus.
- `destroy()`. Destroy the editor, remove it from the DOM.

## Develop

Clone the git repository

Install dependencies (once):

```
npm install
```

Start the demo project (at http://localhost:3000):

```
npm run dev
```

Build the library:

```
npm run package
```

Run unit tests:

```
npm test
```

Run linter:

```
npm run lint
```

Publish to npm (will increase version number and publish to npm):

```
npm run release
```

## License

Released under the [ISC license](LICENSE.md).
