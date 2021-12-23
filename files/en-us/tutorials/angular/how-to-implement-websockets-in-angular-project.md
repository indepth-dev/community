---
title: How to implement WebSockets in Angular Project | indepth.dev
author_name: David Roncal
author_link: https://twitter.com/sd9321
discussion_link: https://github.com/indepth-dev/community/discussions/232
display_name: How to implement WebSockets in Angular Project
---

# How to implement WebSockets in Angular Project

### In this tutorial we will learn how to implement a service to interact through WebSocket API.

The WebSocket API is an advanced technology that allows communication between the client's browser and the server, both to send messages and to receive responses without the need to make a new request to the server to get the answer.

## WebSocket Flow

To make a WebSocket connection it is necessary to use the HTTP protocol upgrade mechanism, this allows us to move from an HTTP 1.1 connection to HTTP 2.0 (which allows using server push feature), or as in this case to move from an HTTP 1.1 connection to a WebSocket connection.

The client makes the request using HTTP methods such as GET or POST, the server evaluates the request and if the response is positive the server returns a status code 101 with the headers "Connection" and "Upgrade".
After that, as the connection between client and server exists, it is possible to exchange messages and event-based responses until one of the two sides closes the connection.

The following image describes the flow explained in previous paragraphs.

![WebSocket flow](https://i.imgur.com/x3keabd.png "WebSocket flow")

## WebSocket Server

It’s a TCP application that can be written in any server-side programming language. Its function will be to manage connections, sending and receiving messages. It’s recommended to separate these servers from other applications and use a reverse proxy.

You can use this Node + Express WebSocket Server for angular client implementation: https://github.com/davrv93/ws-server.

## Implementation

We will create an angular project using the command `ng new`, in this case the name of the project will be `socketrv`, likewise we create 2 services `socket.service.ts` and `websocket.service.ts` inside a folder called services.

![Project folder structure](https://i.imgur.com/HvISMcL.png "Project folder structure")

## WebSocketService

We will use a service called `WebSocketService` which will allow us to connect to the WebSocket server. `WebSocketService` class has following overall structure:
A list with bullet points:


1. Properties
a) subject: Is an instance of AnonymousSubject class with MessageEvent property. The AnonymousSubject class allows extending a Subject by defining the source and destination observables.
b) message: Every Subject is an Observable and an Observer. We'll subscribe to this Subject, and we'll be able to call next to feed values as well as error and complete. 
2. Methods
a) connect: This method validates if subject property doesn’t exist and then calls create method for creating the subject.
b) create: This method creates the AnonymousSubject that will be used for subscriptions. 
3. Interfaces
a) Message: It is a dict that  defines the behavior of an object, but does not specify its content, the attributes will be `source` and `content`, both of type `string`. 

And the code for the same is given below:

```typescript
// src\app\services\websocket.service.ts
import { Injectable } from "@angular/core";
import { Observable, Observer } from 'rxjs';
import { AnonymousSubject } from 'rxjs/internal/Subject';
import { Subject } from 'rxjs';
import { map } from 'rxjs/operators';

const CHAT_URL = "ws://localhost:5000";

export interface Message {
    source: string;
    content: string;
}

@Injectable()
export class WebsocketService {
    private subject: AnonymousSubject<MessageEvent>;
    public messages: Subject<Message>;

    constructor() {
        this.messages = <Subject<Message>>this.connect(CHAT_URL).pipe(
            map(
                (response: MessageEvent): Message => {
                    console.log(response.data);
                    let data = JSON.parse(response.data)
                    return data;
                }
            )
        );
    }

    public connect(url): AnonymousSubject<MessageEvent> {
        if (!this.subject) {
            this.subject = this.create(url);
            console.log("Successfully connected: " + url);
        }
        return this.subject;
    }

    private create(url): AnonymousSubject<MessageEvent> {
        let ws = new WebSocket(url);
        let observable = new Observable((obs: Observer<MessageEvent>) => {
            ws.onmessage = obs.next.bind(obs);
            ws.onerror = obs.error.bind(obs);
            ws.onclose = obs.complete.bind(obs);
            return ws.close.bind(ws);
        });
        let observer = {
            error: null,
            complete: null,
            next: (data: Object) => {
                console.log('Message sent to websocket: ', data);
                if (ws.readyState === WebSocket.OPEN) {
                    ws.send(JSON.stringify(data));
                }
            }
        };
        return new AnonymousSubject<MessageEvent>(observer, observable);
    }
}
```

## AppComponent

We will design a component with a textarea, a button for sending messages and two lists for sent and received messages. The resulting component would be as follows.

```html
<!--src\app\app.component.html-->
<h1>
  Angular + WebSockets
</h1>
<br>
<div class="row">
  <div class="column">
    <textarea [(ngModel)]="content" name="contentInput" cols="40" rows="5"></textarea>
    <br>
    <button (click)="sendMsg()">Send Message</button>
  </div>

</div>
<br>
<div>
  <h1>Sent Messages</h1>
  <br>
  <ul>
    <li *ngFor="let item of sent">Source: {{item.source}} -> Content {{item.content}}</li>
  </ul>
</div>

<br>
<div>
  <h1>Received Messages</h1>
  <br>
  <ul>
    <li *ngFor="let item of received">Source: {{item.source}} -> Content {{item.content}}</li>
  </ul>
</div>
```

The typescript component must be able to connect while maintaining a WebSocket subscription and be able to send messages over the connection.
`AppComponent` class has following overall structure:

1. Properties
a) content: Will be used for textarea value.
b) received: Array for received messages.
c) sent: Array for sent messages.
2. Methods
a) sendMsg: Gets the textarea value, pushes the message to the `sent` array and uses the `next` method.
 
The `sendMsg` method declares an object type variable message with properties “source and content”, we assign to source the value localhost and to content we assign the value of the textarea. The value of source is localhost to differentiate whether it is an outgoing or incoming message.
We will add an item with the variable message in the list of sent messages and finally we will use the Subject’s next method to send the message to the Socket server.

```typescript
//src\app\app.component.ts
  sendMsg() {
    let message = {
      source: '',
      content: ''
    };
    message.source = 'localhost';
    message.content = this.content;

    this.sent.push(message);
    this.SocketService.messages.next(message);
  }
```

The `AppComponent` class would be as follows.

```typescript
//src\app\app.component.ts
import { Component } from '@angular/core';
import { WebsocketService } from "./services/websocket.service";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"],
  providers: [WebsocketService]
})

export class AppComponent {
  title = 'socketrv';
  content = '';
  received = [];
  sent = [];

  constructor(private WebsocketService: WebsocketService) {
    WebsocketService.messages.subscribe(msg => {
      this.received.push(msg);
      console.log("Response from websocket: " + msg);
    });
  }

  sendMsg() {
    let message = {
      source: '',
      content: ''
    };
    message.source = 'localhost';
    message.content = this.content;

    this.sent.push(message);
    this.WebsocketService.messages.next(message);
  }
}
```

Our deployed app will be like this:

![Deployed app](https://i.imgur.com/XPWKHld.png "Deployed app")

In the console of our browser we can validate the sending of the message

### Console tab

![Console tab](https://i.imgur.com/i60wBWA.png "Console tab")

### Network tab

![Network tab](https://i.imgur.com/AKt6eKk.png "Network tab")

## Result

The application will look like this:

![Network tab](https://i.imgur.com/2j0thAM.gif "Network tab")

## Summary

The WebSocket API allows us to communicate between client and server through messages and event-based responses. Its implementation in an Angular project can be done through services and interfaces that allow the connection and management of messages.

## References

https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications
https://developer.mozilla.org/en/docs/Web/API/WebSockets_API/Writing_WebSocket_servers
https://developer.mozilla.org/en/docs/Web/HTTP/Protocol_upgrade_mechanism
