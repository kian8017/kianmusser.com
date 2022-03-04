---
title: Custom WebSocket Multiplexing with ReactJS
date: 2022-02-22
description: My learning process implementing multiplexing on WebSockets
draft: false
lastmod: 2022-03-04T20:16:52.980Z
---

> Disclaimer: The following explanation and code is how I decided to do things on this project, and may not be right for you. Also, this article is more of a story, so it may be less formal than other tutorials.

## Background

I've been working on a project for a family member who loves genealogy and family history. It's basically a fancy search engine for names, designed to help with deciphering old handwriting.

I started off with a basic frontend written in ReactJS, and a simple HTTP server written in Golang. This worked well for a while, but as the complexity of the project increased, I started seeing weaknesses to this approach.

I needed to provide:

- Search results,
- The ability to cancel a search,
- Similar results if they didn't find what they needed, and
- The number of names the project was currently searching through.

While you probably could accomplish this with HTTP, it didn't feel like a good fit for this project.

> If you search by requesting `example.com/search?query=test&country=USA`, how would you cancel the search?

## Why WebSockets?

> I won't get into the basics of WebSockets here. For an introduction, check out [this article](https://sookocheff.com/post/networking/how-do-websockets-work/).

As I searched for a solution, I stumbled upon WebSockets. With WebSockets, I could send my search request, and receive back multiple messages from the server **as soon as they were available**.

This was a game changer!

I could now display results as soon as they were loaded, and fallback results whenever they were loaded! (before, I just had to wait for everything to load before getting a response back from the server.) And I could cancel a search at any time!

I quickly adopted WebSockets.

## Initial Implementation

I decided to start simple.

```json
{
  "type": "message_type",
  "data": "json_encoded_message_data"
}
```

This is how I formatted Socket messages in my initial implementation. It wasn't the prettiest, but it worked out well. The website would send a `SEARCH` type message with the query in data, and receive back a `COUNT` message (since I needed the number of names the server was searching through), a `RESULTS` message, and an `EXTENDED_RESULTS` (fallback similar results) message.

And I was happy! This worked out well. I thought it was so cool that it was sending JSON messages back and forth and encoding and decoding them all behind the scenes to make the website work!

### In ReactJS

Whenever I wanted to add additional functionality to the website, I'd create the new component, say, `WelcomeMessage`, and connect it to the WebSocket like this.

```jsx
const WelcomeMessage = (props) => {
  const [message, setMessage] = useState("");

  // The WebSocket object is stored in a Context
  const [addWsCb, removeWsCb, sendWs] = useContext(WebSocketContext);

  // When the WebSocket receives a message matching a message type to listen for, it calls this function
  const wsCallback = (message) => {
    // Since this is only one case, you could replace this with an if-else,
    // But in other places (like search), you'd be listening for multiple message types
    switch (message.type) {
      case MT_SET_MESSAGE: // MT = messageType
        setMessage(json.parse(message.data))
        break;
      default:
        // Shouldn't ever happen
        console.error("unhandled message in WelcomeMessage", message);
        break;
    }
  }

  // This is where we subscribe and unsubscribe to the WebSocket
  useEffect(() => {
    // First argument is a list of message types to listen for
    const wsId = addWsCb(["MT_SET_MESSAGE"], wsCallback);

    sendWs({type: MT_GET_MESSAGE, data: ""}); // Get the welcome message

    // Unsubscribe on unmount
    return () => {
      removeWsCb(wsId);
    }
  }

    return (
        <div>{message}</div>
    );
}
```

It's not too bad, but it's still a lot of repetitive code (especially that `useEffect` hook that every Component that uses the WebSocket connection needs). But for the moment, it worked, and so I used it.

Until it stopped working...

## Need for Proper Multiplexing

Previously, I had added a welcome message to the website and was currently working on the admin panel. I added a section to allow admins to change the welcome message, and it worked! Admins could update the welcome message without having to go and manually edit the database.

I refreshed the page to double-check to make sure everything worked. The main page loaded and welcome message showed up. I opened the admin panel, and...

_The welcome message showed up again..._

...

I should have seen it coming.

While this wasn't a dealbreaker (after all, it's just a welcome message popping up again, and only admins would ever see it), it wasn't ideal either.

With my naive implementation of multiplexing, I made the assumption that only one part of the website would ever use a certain type of message. For example, the `WelcomeMessage` component would be the only component to use the `GET_MESSAGE` and `SET_MESSAGE` types.

I could fix this by adding additional types, like `ADMIN_GET_MESSAGE` and `ADMIN_SET_MESSAGE`, but that didn't seem like a sustainable solution. And so I decided to implement proper multiplexing.

## Implementing IDs

To implement proper multiplexing, I added a unique ID to each message.

```json
{
  "type": "message_type",
  "data": "json_encoded_message_data",
  "id": 1
}
```

Now, instead of components on the website listening in to certain message types, each component had its own "channel" or sub-connection to communicate with the server. It could send whatever messages it needed to without interference from any other part of the website.

### ReactJS Hook

I also created a ReactJS hook to reduce all the boilerplate code I had been writing. It simplifies the old boilerplate code earlier in this article to this:

```jsx
const WelcomeMessage = (props) => {
  const [message, setMessage] = useState("");

  const wsCallback = (message) => {
    switch (message.type) {
      case MT_SET_MESSAGE:
        setMessage(json.parse(message.data))
        break;
      default:
        console.error("unhandled message in WelcomeMessage", message);
        break;
    }
  }

  // This hook handles keeping track of subscribing and unsubscribing to the WebSocket
  const sendWs = useWs(wsCallback);

  useEffect(() => {
    // type, data (id added by hook)
    sendWs(MT_GET_MESSAGE, ""); // Get the welcome message
  }

  return (
    <div>{message}</div>
  );
}
```

Compared to the larger `useEffect` hook you saw earlier, now we only have one line of code to connect to the WebSocket and one line to send off the initial request!

As to the `useWs` hook itself, it works like this:

```jsx
const useWs = (wsCb) => {
  const [addWsCb, removeWsCb, sendRawWs] = useContext(WebsocketContext);
  const id = useRef(-1);

  // Some code about making sure the WebSocket connection is ready before sending omitted for brevity

  useEffect(() => {
    // addWsCb and removeWsCb now don't need a list of messageTypes
    id.current = addWsCb(wsCb); // The WebSocket Context assigns the IDs

    return () => {
      removeWsCb(id.current);
    };
  }, []);

  const sendWs = (messageType, messageData) => {
    sendRawWs({
      type: messageType,
      data: messageData,
      id: id.current,
    });
  };

  return sendWs;
};
```

It serves two functions, subscribing and unsubscribing from the WebSocket context (which I had to do manually before in the `useEffect` hook), and adding the ID automatically, preventing even more boilerplate.

## Summary

Throughout this process I learned two things:

1. Just how awesome WebSockets are! They're incredibly versatile. With WebSockets combined with this custom multiplexing layer, adding new features is so much easier!
2. The importance of React hooks! I'd created hooks before, but this was my first implementation of a hook that had such a large effect on a project of mine. It abstracted away all the details of managing the connections and WebSocket, leaving me free to focus on adding functionality to the website.

I hope that by reading, you've gotten a sense of just how cool these technologies are!

Try them out sometime! You won't look back.
