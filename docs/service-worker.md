# Service Worker

## What is it?
A gateway between local web applications, the browser and network when available. They allow for the creation of offline caches, operations, and some other activities.

Can be thought of as "Shared Workers that can start, process events, and die without ever handling messages from documents"

**If you are familiar with MVC, the service worker can be thought of as taking position as the Controller for the page itself**

## MultiThreading?
With the introduction of Web Workers in html 5, an api was introduced to allow actions or scripts to be run a separate thread. This is costly however for the browsers memory resources, so they should be used lightly.

### Key features
- It is event-driven
- Is a javascript file that can control web page it is registered on, modfiying navigation and resource requests, and caches resources in a granual fashion ( to improve loading time of your page)
- worker context: scope, no DOM access and so is non-blocking. Runs on different thread to main js which powers the app.
- only on https sites only.
- Service workers may be started by user agents without an attached document and may be killed by the user agent at nearly any time
- New tab can preserve service worker

### Limitations
- can't use local storage or XHR inside a service worker
- only on https sites only.
- user needs to manage and be more aware of the interaction between service workers and websites. 

## Differences with shared worker
Like Shared Worker
- Runs in its own global script context (usually in its own - - thread)
- Isn’t tied to a particular page
- Has no DOM access

Unlike shared worker
- Can run without any page at all
- Can terminate when it isn’t in use, and run again when needed (i.e., it’s event-driven)
- Has a defined upgrade model
- Is HTTPS only (more on that in a bit)

## How to implement it
Begin by registering your code using ServiceWorkerContainer.register() which returns a promise if the registraton fails

```js
// The service worker script must be on the same origin as the page, although you can import scripts from other origins using importScripts 
navigator.serviceWorker.register('sw.js', {
        // The scope cannot be parent to the script url
        scope: './'
      });

// LIFECYCLE Donwload -> Innstall --> Activate

self.addEventListener('install', function(event) {
  event.waitUntil(
    fetchStuffAndInitDatabases()
  );
});

self.addEventListener('activate', function(event) {
  // You're good to go!
});

```

## Inspecting in chrome
- Frist open Chrome dev tools
- You can see the current running service workers by going to `Application`
- You can also see the events run through a service worker in the network tab.

## Heavy use of Promises

Promise is a class which takes a resolve or reject parameter and returns the resolved value. If the result is sucessful, then you can chain another promise or simply return the value.

```js
var promise = new Promise(function(resolve, reject) {
  // do a thing, possibly async, then…

  if (/* everything turned out fine */) {
    resolve("Stuff worked!");
  }
  else {
    reject(Error("It broke"));
  }
});

// use the promise
promise.then(function(result) {
  console.log(result); // "Stuff worked!"
}, function(err) {
  console.log(err); // Error: "It broke"
}); //thens can be chained
```

## Future work
With streams, intercepting the byte code values and more granual control of the information being sent to the DOM. This will allos for support for:
- periodic sync (sync in specified intervals)
or 
- geofencing( lets webapps setup geographic boundaries around specific locations and then receive notifications when the hosting device enters or leaves those areas). 

### Brwoser sync
- defer actions until user has a stable connection

```js
// Register your service worker:
navigator.serviceWorker.register('/sw.js');

// Then later, request a one-off sync:
navigator.serviceWorker.ready.then(function(swRegistration) {
  return swRegistration.sync.register('myFirstSync');
});
Then listen for the event in /sw.js:

self.addEventListener('sync', function(event) {
  if (event.tag == 'myFirstSync') {
    event.waitUntil(doSomeStuff());
  }
});


// intercept fetch requests and throw a custom response
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch(event.request).catch(function() {
      return new Response("Request failed!");
    })
  );
});
```

[Demo For RespondWith](https://jakearchibald.github.io/isserviceworkerready/demos/img-rewrite/)

### The Cache
Service worker comes with a caching API, letting you create stores of responses keyed by request.

```js
self.addEventListener('install', function(event) {
  // pre cache a load of stuff:
  event.waitUntil(
    caches.open('myapp-static-v1').then(function(cache) {
      return cache.addAll([
        '/',
        '/styles/all.css',
        '/styles/imgs/bg.png',
        '/scripts/all.js'
      ]);
    })
  )
});

self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(cachedResponse) {
      return cachedResponse || fetch(event.request);
    })
  );
});
```

## Uses at H30
- secure persistence of encrypted authentication data across tabs.
- better transaction experiences (registration, submission, etc)
- caching for faster page loads
- geofencing... selecting services based on location of the user.

## Online examples
[Patterns To Emulate](https://serviceworke.rs)

## When can I use it?
[Is service worker ready?](https://jakearchibald.github.io/isserviceworkerready/)


## More Service Worker Articles
[w3c service woker](https://w3c.github.io/ServiceWorker/#service-worker-url)

[explainer for service workers](https://github.com/w3c/ServiceWorker/blob/master/explainer.md)

[More on Background Sync...](https://developers.google.com/web/updates/2015/12/background-sync)