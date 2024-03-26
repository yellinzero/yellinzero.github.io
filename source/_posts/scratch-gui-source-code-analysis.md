---
title: Scratch Gui Source Code Analysis(WIP)
date: 2024-03-11 08:35:15
tags: 
    - scratch gui
categories: 
    - source code analysis
---

## Entry Function Overview

```jsx
const appTarget = document.createElement('div');
appTarget.className = styles.app;
document.body.appendChild(appTarget)

if (supportedBrowser()) {
    // require needed here to avoid importing unsupported browser-crashing code
    // at the top level
    require('./render-gui.jsx').default(appTarget);

} else {
    BrowserModalComponent.setAppElement(appTarget);
    const WrappedBrowserModalComponent = AppStateHOC(BrowserModalComponent, true /* localesOnly */);
    const handleBack = () => {};
    // eslint-disable-next-line react/jsx-no-bind
    ReactDOM.render(<WrappedBrowserModalComponent onBack={handleBack} />, appTarget);
});
```

### Utilizing CSS Modules

The `appTarget.className = styles.app` leverages CSS Modules, a technique that scopes class names by automatically generating unique identifiers.This strategy effectively prevents styling conflicts across components. Even if multiple components use a `.app` class, CSS Modules ensure that each class name, such as `index_app_2mqDO` , is unique and scoped to its components. This isolation is achieved through specific webpack configuration:

```js
 {
     test: /\.css$/,
     use: [{
        loader: 'style-loader'
     }, {
         loader: 'css-loader',
         options: {
         modules: true,
         importLoaders: 1,
         localIdentName: '[name]_[local]_[hash:base64:5]',
         camelCase: true
        }...]
 }
```

### Ensuring Browser Compatibility

The core functionality of the snippet involves checking for browser compatibility. This is crucial for preventing the application from crashing in un supported browsers. If the browser is compatible, the application's GUI rendering module is dynamically required and executed. By this way, it ensures that potentially incompatible code is not loaded upfront, enhancing performance and stability. (bowser check by using [bowser](https://www.npmjs.com/package/bowser)).

In cases where the browser is not supported, the code gracefully handles this scenario by utilizing a modal component to inform the user. This is facilitated by libraries such as [react-intl(Format.JS)](https://formatjs.io/docs/react-intl/) for internationalization and react-modal](https://github.com/reactjs/react-modal) for rendering modeldialogs.

### Runtime Props type checking

Although JavaScript lacks the type checking capabilities of TypeScript, runtime type checking is employed via [prop-types](https://github.com/facebook/prop-types) to ensure the integrity of prop types across components.

### State management

In this project, state management is handled using [Redux](https://redux.js.org/). Redux is known for its robustness and high degree of control over application state, making it a powerful choice for complex applications.

### AppStateHOC

Within the entry function, the application introduces an `AppStateHOC` (Higer Order Component) that serves as a foundational component. This HOC is tasked with providing some top-level states via Redux. The configuration of `AppStateHOC` is based on the `localesOnly` flag.

- When `localesOnly` is set to `true` , the application opts for a streamlined state manament strategy focused exclusively on internationalization(i18n).

- Conversely, with `localesOnly` set to `false` (indicating GUI mode). `AppStateHOC` enriches the application with a more comprehensive state management setup. This   includes GUI-related states and reducers, alongside a specialized middleware designed to enhance performance and user experience. The middleware incorporates a throttle mechanism, set at a 300-millisecond interval. This throttle ensures that actions are not triggered too frequently, which can be particularly beneficial in preventing performance bottlenecks and enhancing the application's responsiveness
  
  Here's how the throttling middleware is declared: 
  
  ```javascript
  const guiMiddleware = 
      compose(
          applyMiddleware(
              throttle(300, {leading: true, trailing: true})
          )
      );
  ```
  
  








