# Ceramic

Ceramic is a tool that helps to build (mobile) applications with vite and jsx.
It requires no (except for vite) external dependencies and is incredibly fast.

THIS TOOL IS IN VERY EARLY STAGES AND SHOUDN'T BE USED IN A PRODUCTION APP YET

## How does it work

### App

The entry point of your application is the `CeramicApp` function

```jsx
const App = CeramicApp({
  root: document.querySelector(".app"),
});
```

Now that you have setup your app, you should render it

```jsx
App.render();
```

This will render the page provided at the current pathname, see [routing](#routing) for more info.
The App object also provides a way to listen to events

```jsx
App.on("appload", (e) => { ... });
```

When building a mobile application this can be helpfull to hide the splashscreen at the right time.
But make sure to register the `appload` event before calling the `render` function, otherwise it won't be called

### Pages

The syntax is a bit inspired by a flutter app, this is what a page will probably look like

```jsx
import { CeramicPage } from "ceramic-app";

export default CeramicPage({
  navbar: <Navbar />,
  body: (
    <>
      <h1>Hello,</h1>
      <h1>World</h1>
    </>
  ),
});
```

Now the exported value contains all the necessary information for ceramic to render it.
At the moment you can only render it with the builtin router, but this will change in the future

### Routing

NOTE: Routing isn't finalized yet, so it will experience a lot of changes in the future
Routes are defined by calling the `defineRoutes` function, which looks something like this

```js
import { defineRoutes } from "ceramic-app";

import Home from "./pages/home";
import About from "./pages/about";

export default defineRoutes([
  {
    name: "home",
    route: "/",
    page: Home,
  },
  {
    name: "about",
    route: "/about",
    page: About,
  },
]);
```

### JSX

Ceramic uses vite's (or esbuild's) builtin jsx transformer alongside a custom jsx parser that transforms it into h functions,
which get parsed to native HTML elements by `ceramic/jsx-parser`.
The vite config gets automatically updated once `ceramic/plugin` is used.

```js
import { defineConfig } from "vite";
import ceramic from "ceramic-app/plugin";

export default defineConfig({
  plugins: [ceramic()],
});
```

NOTE: when a function returns a Fragment, it will be transformed into a native [documentFragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment).
Please make sure to read the documentation for this, as it can act a bit strange sometimes

### Parser

For the parsing off the jsx itself ceramic relies on vite's (or esbuild) builtin jsx parser,
but ceramic does have a custom parser for on\* event parameters (onclick, etc.) in jsx, as vite just puts the function source as the parameter.
This is what a transformation might look like:

```jsx
...
export function ClickableButton() {
  return (
    <button class="logo" onclick={back}>
      <IconButton icon="chevron_left" />
    </button>
  );
}
...
```

Gets transformed to something like this

```jsx
import { registerEventListener as __key_registerEventListener } from "ceramic-app/events";
const __key_click_1476ea87be4e1 = __key_registerEventListener("click", back);
...
export function ClickableButton() {
  return (
    <button class="logo" key={__key_click_1476ea87be4e1}>
      <IconButton icon="chevron_left" />
    </button>
  );
}
...
```

Now Ceramic takes it over and registers the required eventlisteners and calls the provided callbacks accordingly.
At the moment only default behaviour (as shown above) is supported, more customization options might come in the future.

## TODO

Router

- Route validation
- Wildcard/urlparams support

JSX runtime

- Improve code structure and efficiency
- Rewrite to support reactivity
  - Fist create an modal representing the dom as an object, which is useful for observing reactive elements
  - When rendering parse the modal
  - When reactive value changes it should know exactly what element to edit
  - It can also drop unneccesary event listeners that aren't used in the current context
- Add a custom global event listener class that can drop event listeners when they are out of scope of the page

Parser

- Add reactivity > https://indepth.dev/posts/1289/solidjs-reactivity-to-rendering

APP

- Add app `preset`: mobile, web
  - For changing behaiviour of the app > What behaviour ?
- Support for scoped css with `@scope` rule and for importing a css file
- Better logging
- Transitions
  - Transition handler function, user can determine what transition to use based on the `from` and `to` parameters which define the current url and the target url
  - Custom transitions,
  - More presets

Package

- Create a better name, `fluide` maybe?
- Update docs

## Reactivity strategy

```jsx
const [count, setCount] = hookFunction(initialvalue);
```

return a reference to the hook on `count.ref`, while count can still be a string, or integer, or array, etc.
