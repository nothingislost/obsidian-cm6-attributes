## Obsidian CM6 Attributes Reference Plugin

This reference plugin implements a ViewPlugin which will parse the markdown syntaxTree and add various Decorations to enhance the editor.

The plugin will add:
- An inline fold widget to list items
- An inline href attribute to barelinks
- A .cm-line based data attribute to all lines containing hashtags which will contain the value of all tags found on the line

The plugin includes examples of the following CodeMirror 6 components:

- [ViewPlugin](https://codemirror.net/6/docs/ref/#view.ViewPlugin)
	- [eventHandlers](https://codemirror.net/6/docs/ref/#view.PluginSpec.eventHandlers)
- Markdown Token Parsing
	- [syntaxTree](https://codemirror.net/6/docs/ref/#language.syntaxTree)
- [Code Folding](https://codemirror.net/6/docs/ref/#folding)
- [Decorations](https://codemirror.net/6/docs/ref/#decorations)
	- [widget](https://codemirror.net/6/docs/ref/#view.Decoration^widget)
	- [mark](https://codemirror.net/6/docs/ref/#view.Decoration^mark)
	- [line](https://codemirror.net/6/docs/ref/#view.Decoration^line)

## Getting Started with CodeMirror 6

Before getting started with CodeMirror 6 development, it is HIGHLY recommended that you first read the CodeMirror 6 documentation. CodeMirror 6 is a very complex architecture with many modular components. The CodeMirror 6 author does a great job of explaining the architecture in the [CodeMirror 6 System Guide](https://codemirror.net/6/docs/guide/). This is a long document but it is worth reading from start to finish.

If you have experience with CodeMirror 5, there's a [CodeMirror 5 -> CodeMirror 6 Migration Guide](https://codemirror.net/6/docs/migration/) that does a great job of explaining how the system has changed between the two versions. It's important that you read this document as these are not subtle changes. The system has changed significantly and if you go into CodeMirror 6 with CodeMirror 5 expectations, you will be swimming upstream.

Additional resources include [CodeMirror 6 Examples](https://codemirror.net/6/examples/), the [CodeMirror 6 Reference Manual](https://codemirror.net/6/docs/ref/) and the [CodeMirror 6 Discussion Forum](https://discuss.codemirror.net/c/next).

## Using CodeMirror 6 with Obsidian

### Managing CodeMirror 6 Package Imports

When building CodeMirror 6 extensions, it is critical that you use the exact CM6 classes that were used to instantiate the editor. For example, if you import a different version of "@codemirror/state" than what is used by Obsidian, your plugin will either fail to register or just not work at all.

The good news is that, as of version 0.13.8, Obsidian provides a way to ensure that you're always using the exact CM6 classes used by the editor. Obsidian does this by overloading calls to `require` for the "@codemirror" packages so that they can return the exact versions that they use internally.

What this means, practically, is that you should mark all of your @codemirror dependencies as external in whatever bundler you are using. By marking these as external, you are telling the bundler to not include them in your plugin. For esbuild, it would look like this:

```js
    external: [
      "obsidian",
      "electron",
      "codemirror",
      "@codemirror/autocomplete",
      "@codemirror/closebrackets",
      "@codemirror/commands",
      "@codemirror/fold",
      "@codemirror/gutter",
      "@codemirror/history",
      "@codemirror/language",
      "@codemirror/rangeset",
      "@codemirror/rectangular-selection",
      "@codemirror/search",
      "@codemirror/state",
      "@codemirror/stream-parser",
      "@codemirror/text",
      "@codemirror/view",
      ...builtins,
    ]
```

Note that the list above is the comprehensive list of @codemirror packages provided by Obsidian as of version 0.13.8. It is not advised to try and use any @codemirror package that is not in this list. If you attempt to import a @codemirror package not in this list, you run a high risk of introducing package conflicts and subtle bugs.

With that done, you can now import packages like this `import { StateEffect, StateField, Transaction } from "@codemirror/state";` and be confident that your version of `StateField` will be the exact `StateField` used by Obsidian.

### Registering a CodeMirror 6 Extension

Before continuing further, make sure you understand the [CodeMirror 6 Extension System](https://codemirror.net/6/docs/guide/#extension) and the components involved in [Extending CodeMirror](https://codemirror.net/6/docs/guide/#extending-codemirror).

Obsidian has provided a helper function to make it easy to register a CM6 extension and manage its lifecycle. The method can be found on the `Plugin` class and is called [registerEditorExtension](https://github.com/obsidianmd/obsidian-api/blob/master/obsidian.d.ts#L2345).

`registerEditorExtension` takes one argument, a CodeMirror 6 Extension. An Extension can be a single extension or an array of multiple extensions.

Once registered with `registerEditorExtension`, your extension will be immediately loaded on all active editor instances and all future editor instances.

`registerEditorExtension` will also handle unloading your extension when your plugin is disabled.

For cases where you want to get more advanced with your extension management, refer to the documentation on [Dynamic Configuration](https://codemirror.net/6/examples/config/#dynamic-configuration) where they explain how to create and manage `compartments`. Warning, this is an advanced topic.

### Additional Obsidian Helper Components

Obsidian exposes two additional components which can be helpful during plugin development. These components are [StateFields](https://codemirror.net/6/docs/guide/#state-fields) that store a reference to the CM6 `EditorView` and the Obsidian `MarkdownView` in the CM6 `EditorState`.

The two `StateField` components can be imported from the 'obsidian' package and are named [editorEditorField](https://github.com/obsidianmd/obsidian-api/blob/master/obsidian.d.ts#L808) and [editorViewField](https://github.com/obsidianmd/obsidian-api/blob/master/obsidian.d.ts#L935).

These fields are useful for getting a reference to the current editor's `EditorView` or `MarkdownView` from a `EditorState` event.

When you register a `StateField`, your `StateField` will start receiving state events from the editor. Since `EditorState` is completely isolated from the actual `EditorView` in CM6, these state events will provide no indication of what editor or view the event is associated to.

For example, here is how one might use the `editorEditorField` `StateField` to remove a class from the associated `div.markdown-source-view` element whenever the EditorState has been created or reset:

```js
    const zoomStateField = StateField.define<DecorationSet>({
      create(state: EditorState) {
        const editorView = state.field(editorEditorField);
        editorView.dom.parentElement.removeClass("is-zoomed-in");
        return Decoration.none;
      }
      ...
    })
```

It is advised to use these `StateField` helpers sparingly. Under normal circumstances, you should keep a clean separation between `EditorState` and `EditorView`.

## Sample Plugin Documentation

### First time developing plugins?

Quick starting guide for new plugin devs:

- Make a copy of this repo as a template with the "Use this template" button (login to GitHub if you don't see it).
- Clone your repo to a local development folder. For convenience, you can place this folder in your `.obsidian/plugins/your-plugin-name` folder.
- Install NodeJS, then run `npm i` in the command line under your repo folder.
- Run `npm run dev` to compile your plugin from `main.ts` to `main.js`.
- Make changes to `main.ts` (or create new `.ts` files). Those changes should be automatically compiled into `main.js`.
- Reload Obsidian to load the new version of your plugin.
- Enable plugin in settings window.
- For updates to the Obsidian API run `npm update` in the command line under your repo folder.

### Releasing new releases

- Update your `manifest.json` with your new version number, such as `1.0.1`, and the minimum Obsidian version required for your latest release.
- Update your `versions.json` file with `"new-plugin-version": "minimum-obsidian-version"` so older versions of Obsidian can download an older version of your plugin that's compatible.
- Create new GitHub release using your new version number as the "Tag version". Use the exact version number, don't include a prefix `v`. See here for an example: https://github.com/obsidianmd/obsidian-sample-plugin/releases
- Upload the files `manifest.json`, `main.js`, `styles.css` as binary attachments. Note: The manifest.json file must be in two places, first the root path of your repository and also in the release.
- Publish the release.

### Adding your plugin to the community plugin list

- Publish an initial version.
- Make sure you have a `README.md` file in the root of your repo.
- Make a pull request at https://github.com/obsidianmd/obsidian-releases to add your plugin.

### How to use

- Clone this repo.
- `npm i` or `yarn` to install dependencies
- `npm run dev` to start compilation in watch mode.

### Manually installing the plugin

- Copy over `main.js`, `styles.css`, `manifest.json` to your vault `VaultFolder/.obsidian/plugins/your-plugin-id/`.

### Improve code quality with eslint (optional)
- [ESLint](https://eslint.org/) is a tool that analyzes your code to quickly find problems. You can run ESLint against your plugin to find common bugs and ways to improve your code. 
- To use eslint with this project, make sure to install eslint from terminal:
  - `npm install -g eslint`
- To use eslint to analyze this project use this command:
  - `eslint main.ts`
  - eslint will then create a report with suggestions for code improvement by file and line number.
- If your source code is in a folder, such as `src`, you can use eslint with this command to analyze all files in that folder:
  - `eslint .\src\`


### API Documentation

See https://github.com/obsidianmd/obsidian-api
