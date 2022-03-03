---
title: "Need to quickly implement toast notifications in React? Try react-hot-toast"
date: 2022-02-28
description: "How to quickly and simply implement toast notifications in a React app"
draft: false
---

> Disclaimer: I have used `react-hot-toast` in one of my projects, but I have no affiliation with this library or its author.

Are you building an app with React and want to add toast notifications but spend the least amount of time possible doing so? `react-hot-toast` is a simple, batteries-included library for toast notifications.

In this article, you'll learn how to quickly install and setup `react-hot-toast` to get toast notifications working as quick as possible. And at the end, if you want to dig a bit deeper, you'll learn about a couple additional features and ways to customize toast notifications.

## Preparation

This article assumes you already have a React project setup with `create-react-app` or equivalent. If you don't already have a project, check out [this article on how to start one with `create-react-app`](https://www.freecodecamp.org/news/how-to-build-a-react-project-with-create-react-app-in-10-steps/).

Installing `react-hot-toast` is as simple as:

```shell
npm install react-hot-toast
```

Now, you'll set up `react-hot-toast`.

## Setup

Setting up `react-hot-toast` is as simple as installing it. Just place the `<Toaster>` component somewhere within the app.

For example, in the default `create-react-app` sample app, place the `<Toaster>` right inside the main `<App>` component.

```jsx {hl_lines=[6]}
import { Toaster } from "react-hot-toast";

function App() {
  return (
    <div className="App">
      <Toaster />
      <header className="App-header">...</header>
    </div>
  );
}
```

And you're done! Now, let's create some toast notifications!

## Usage

Creating toast notifications with `react-hot-toast` is as simple as calling `toast`. No need to use `useContext` or make sure you put `<ToastProvider>` higher up in your app.

Let's add a quick sample button to test it out.

```jsx {hl_lines=[8]}
import { Toaster, toast } from "react-hot-toast";

function App() {
  return (
    <div className="App">
      <Toaster />
      <header className="App-header">
        <button onClick={() => toast("I'm toast")}>Make Toast</button>
        ...
      </header>
    </div>
  );
}
```

> In this example, `<Toaster>` and `toast` are in the same file, but they don't need to be. As long as `<Toaster>` is somewhere in the app, you can call `toast` from anywhere.

It really is as simple as that. Your app now has toast notifications!

## Extras

Well, that was quick! We only scratched the surface up there, so let's dig a bit deeper and learn a couple ways to customize the toast notifications.

### Customizing toasts

> For a more in-depth tutorial about how to customize `react-hot-toast`, check out [the docs](https://react-hot-toast.com/docs/toast). There's so much more than what we talk about here!

`react-hot-toast` provides a few more built-in types of toast notifications than just `toast`. For example, it provides success and failure types that add icons to help you effectively convey meaning:

```js
toast.success("You did it!"); // Adds an animated green checkmark

toast.error("Something went wrong..."); // Adds an animated red X
```

`react-hot-toast` also has built-in loading toasts that work well with JS promises:

```js
const testPromise = fetch("https://URL_HERE");

toast.promise(testPromise, {
  loading: "Fetching...",
  success: "Fetched!",
  error: "Failed to fetch",
}); // Shows a loading animation
```

You can also use loading toasts without promises:

```js
const toastID = toast.loading("Loading data");
loadData();
toast.dismiss(toastID); // Dismiss loading toast
```

### Customizing <Toaster\>

> There's many more ways to customize `<Toaster>`. For more information, check out [the docs](https://react-hot-toast.com/docs/toaster).

You can change the position of the `<Toaster>` with the `position` prop. Valid options are `top-left`, `top-center`, `top-right`, `bottom-left`, `bottom-center`, and `bottom-right`.

```jsx
<Toaster position="bottom-center" />
```

## Conclusion

You've learned how to quickly implement toast notifications in your React app.

Now, go use what you've learned. Create something amazing!
