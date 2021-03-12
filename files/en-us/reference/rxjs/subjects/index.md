---
title: Subjects - RxJS Reference | indepth.dev
slug: reference/rxjs/subjects
---

# What is RxJS Subject

Subjects allow multiple observers to subscribe to them and then broadcast the received value to those observers. The concept is similar to an `EventEmitter` that keeps a registry of multiple listeners. When an event happens, e.g. new value arrives, it notifies those listeners. In the context of Subjects and RxJS, delivering values to multiple observers is called multicasting.

Here’s a simple example:

```javascript
const subject = new Subject();

subject.subscribe((value) => console.log(value));
subject.subscribe((value) => console.log(value));
subject.subscribe((value) => console.log(value));

subject.next('some value');
```
