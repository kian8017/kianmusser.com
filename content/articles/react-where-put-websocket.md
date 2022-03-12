---
title: "React: Where should I put my Websocket?"
description: The pro's and con's of different places to implement Websockets in React
date: 2022-03-11T20:29:17-06:00
draft: false
lastmod: 2022-03-12T16:08:08.353Z
---

## Introduction

Have you been wondering where to put that `const ws = new WebSocket(...)`? Not sure whether to use Context, a Hook, or just put it in a component? Well look no further!

In this article, you'll learn the different places you can put Websockets, and the advantages and disadvantages of each.

## Prerequisites

This tutorial was written for those who know how to use React (including React hooks and context), and know how to use Websockets in Javascript.

For an introduction to Websockets and how to implement them in React, see [this tutorial on using Websockets with React](https://blog.logrocket.com/websockets-tutorial-how-to-go-real-time-with-node-and-react-8e4693fbf843/).

## Can I put my Websocket...

Now we'll compare the different places you can put a Websocket in a React app, starting with components.

## In a component?

Can you put a Websocket in a component? Yes you can!

Placing your Websocket in a React component is common in examples on how to get started (e.g., [this tutorial](https://dev.to/finallynero/using-websockets-in-react-4fkp)). It's simple, and it works!

### Class Components

The typical way to implement Websockets in a class component is like this:

```jsx {hl_lines=[2,"13-25",30]}
export class WsClass extends Component {
  ws = new WebSocket("wss://echo.websocket.events/");

  constructor(props) {
    super(props);

    this.state = {
      val: null,
    };
  }

  componentDidMount() {
    this.ws.onopen = () => {
      console.log("opened");
      this.ws.send("test"); // message to send on Websocket ready
    };

    this.ws.onclose = () => {
      console.log("closed");
    };

    this.ws.onmessage = (event) => {
      console.log("got message", event.data);
      this.setState({ val: event.data });
    };
  }

  componentWillUnmount() {
    console.log("closing websocket...");
    this.ws.close();
  }

  render() {
    return <div>Value: {this.state.val}</div>;
  }
}
```

You create the Websocket as an instance variable of the class, then assign the `onopen`, `onclose`, and `onmessage` (and `onerror` if you need) events in the `componentDidMount` lifecycle function. You use `setState` to keep track of what the Websocket returns, and close the Websocket in `componentWillUnmount`.

Simple enough, right?

Before we dive into the pro's and con's of putting Websockets in components, let's take a quick look at functional components.

### Functional Components

Functional components can't use instance variables like class components can, so we'll need to use hooks to implement the same functionality.

```jsx {hl_lines=[3,6,"8-19",21,24]}
export const WsFunc = () => {
  const [val, setVal] = useState(null);
  const ws = useRef(null);

  useEffect(() => {
    const socket = new WebSocket("wss://echo.websocket.events/");

    socket.onopen = () => {
      console.log("opened");
    };

    socket.onclose = () => {
      console.log("closed");
    };

    socket.onmessage = (event) => {
      console.log("got message", event.data);
      setVal(event.data);
    };

    ws.current = socket;

    return () => {
      socket.close();
    };
  }, []);

  return <div>Value: {val}</div>;
};
```

We use the `useState` hook to keep track of what the Websocket returns, and `useRef` for keeping track of the Websocket connection.

> **Why use `useRef` for the Websocket?**
>
> `useRef` is not just for references, [according to the docs](https://reactjs.org/docs/hooks-reference.html#useref). It's basically the same as instance variables on a class.
>
> As to why to use it: `useState` triggers rerenders, while `useRef` doesn't.
>
> We don't need to rerender the component when the Websocket connection is created, only when we receive a new message (and that's where we use `useState`).

Now let's review the pro's and con's.

**Pro's:**

- It's simple. Putting the Websocket in a component is the easiest way to get up and running.
- It's self-contained. All Websocket code will stay in this component, so it's easier to keep track of.

**Con's:**

- It's not easily accessible to other components (except children if you pass it down through props). This is fine if only one part of the app needs a Websocket, but when that increases to multiple parts, it starts becoming unwieldy.

Now we've gone over the advantages and disadvantages of putting the Websocket in a component, but what about in a hook?

## In a hook?

Can you put a Websocket in a hook? Yes you can, but it might not be a good fit.

You might implement a Websocket hook like this:

```jsx {hl_lines=[5,8,"10-12",14,17]}
export const useWs = ({ url }) => {
  const [isReady, setIsReady] = useState(false);
  const [val, setVal] = useState(null);

  const ws = useRef(null);

  useEffect(() => {
    const socket = new WebSocket(url);

    socket.onopen = () => setIsReady(true);
    socket.onclose = () => setIsReady(false);
    socket.onmessage = (event) => setVal(event.data);

    ws.current = socket;

    return () => {
      socket.close();
    };
  }, []);

  // bind is needed to make sure `send` references correct `this`
  return [isReady, val, ws.current?.send.bind(ws.current)];
};
```

And then create a component that uses it like so:

```jsx {hl_lines=[2,"4-8"]}
export const WsHook = () => {
  const [ready, val, send] = useWs("wss://echo.websocket.events/");

  useEffect(() => {
    if (ready) {
      send("test message");
    }
  }, [ready, send]); // make sure to include send in dependency array

  return (
    <div>
      Ready: {JSON.stringify(ready)}, Value: {val}
    </div>
  );
};
```

It works, and it's easy to understand and reuse! However, it's not for everyone.

Putting the Websocket in a hook works best when there are multiple parts of the app that each need to connect to different servers. For multiple parts that all need to connect to the same server, using hooks might not be a good fit.

Let's review the pro's and con's.

**Pro's:**

- Works well for multiple Websocket connections to different servers. It abstracts away all the connection logic, making it easy to connect to different servers with the same hook.
- Is easy to use with components. All you need are the `useWs` and `useEffect` hooks in each part of the app.

**Con's:**

- Not a good fit for a single connection. If multiple parts of the app need to access the same Websocket, this hook won't work. It creates a new connection each time you use `useWs`, which could end up creating multiple connections to the same server.

But what if multiple parts of the app need to connect to the same server? Is using a parent component and passing down the connection through props our best bet?

Not quite. React has something built in for this very use case: Context.

## In a context?

Can you put a Websocket in a context? Yes you can!

The code to do so is very similar to the hook above.

```jsx {hl_lines=[10,13,"15-17",19,22]}
export const WebsocketContext = createContext(false, null, () => {});
//                                            ready, value, send

// Make sure to put WebsocketProvider higher up in
// the component tree than any consumers.
export const WebsocketProvider = ({ children }) => {
  const [isReady, setIsReady] = useState(false);
  const [val, setVal] = useState(null);

  const ws = useRef(null);

  useEffect(() => {
    const socket = new WebSocket("wss://echo.websocket.events/");

    socket.onopen = () => setIsReady(true);
    socket.onclose = () => setIsReady(false);
    socket.onmessage = (event) => setVal(event.data);

    ws.current = socket;

    return () => {
      socket.close();
    };
  }, []);

  const ret = [isReady, val, ws.current?.send.bind(ws.current)];

  return (
    <WebsocketContext.Provider value={ret}>
      {children}
    </WebsocketContext.Provider>
  );
};
```

And there's our context! To use it, we just need to create a consumer.

```jsx {hl_lines=[3,"5-9"]}
// Very similar to the WsHook component above.
export const WsConsumer = () => {
  const [ready, val, send] = useContext(WebsocketContext); // use it just like a hook

  useEffect(() => {
    if (ready) {
      send("test message");
    }
  }, [ready, send]); // make sure to include send in dependency array

  return (
    <div>
      Ready: {JSON.stringify(ready)}, Value: {val}
    </div>
  );
};
```

Let's review the pro's and con's.

**Pro's:**

- Only one Websocket. All parts of the app that use `WebsocketContext` use the same, single Websocket.
- Easy to use with components. It's just as easy as using the hook method above.

**Con's:**

- All parts receive all messages. You may need to implement some sort of multiplexing to ensure that only certain parts of the app receive certain messages.
- It isn't easy to connect to multiple servers. You'd need to duplicate the code or create a function that creates multiple contexts.

For most use cases (single Websocket connection and multiple parts that need it), Contexts are what you should use.

## Conclusion

Let's quickly review the different places you can put a Websocket:

- **Components:** best for simple projects
- **Hooks:** best for multiple connections to different servers
- **Contexts:** best for managing a single connection.

There are many places where you can put your Websocket and each has its own pro's and con's. Now you know what works best for you!
