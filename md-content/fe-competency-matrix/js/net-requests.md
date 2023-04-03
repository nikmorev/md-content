# Net requests in the browser

There are two ways to make https request in the browser using javascript:
1. [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
2. [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)

The first one is a "legacy" way and almost not used nowadays.  
The second one is a contemporary instrument that provides a more powerful and 
flexible feature set.  
  
## Fetch API concept

The core objects used in fetch are: [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) and [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response).
These object can be created via constructors and used in Service Workers or in fetch().  
CORS and HTTP are also considered in fetch api concept.

The main instrument is a [fetch()](https://developer.mozilla.org/en-US/docs/Web/API/fetch) function that can be used in Window and WorkerGlobalScope scopes.  

It accepts `Request` object as an argument, or resourse path as first argement and options as second optional argument.
Once responce is executed, `fetch()` returns a promise with a `Response` object.

## Difference from jquery ajax.

* The Promise returned from fetch() won't reject on HTTP error status even if the response is an HTTP 404 or 500. Instead, it will resolve normally (with ok status set to false), and it will only reject on network failure or if anything prevented the request from completing.
* fetch() won't send cross-origin cookies unless you set the credentials [init option](https://developer.mozilla.org/en-US/docs/Web/API/fetch#parameters) (to include).

## Request abort

[AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) and [AbortSignal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) interfaces can be
used to abort incomplete `fetch()` and even `XMLHttpRequest` operations.

```typescript
const controller = new AbortController()
const signal = controller.signal

fetch(url, { signal })
    .then(response => console.log("Download complete", response))
    .catch(err => console.error(`Download error: ${err.message}`))

constroller.abort()
```

In the example above, download will be aborted and fetch will end up with a DOMException named AbortError.

## Sending credentials and more

By default, in cross-origin XMLHttpRequest or Fetch invocations, browsers will not send credentials.
`withCredentials` (XMLHttpRequest) or `creadentials` (fetch()) flags should be set to use request with credentials in case of cors request.  

The `fetch()` method is controlled by the connect-src directive of [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) rather than the directive of the resources it's retrieving.  

There's a `mode` options that allows to limit the request depending on request path.
It can define if an error should be thrown for CORS requests.  

By default, when a `Request` object is created using the `Request()` constructor, the value of the mode property for that `Request` is set to `cors`. Otherwise, it's going to be a `no-cors`.  

For embedded resources where the request is initiated from markup, unless the `crossorigin` attribute is present, the request is in most cases made using the `no-cors` mode — that is, for the `<link>` or `<script>` elements (except when used with modules), or `<img>`, `<audio>`, `<video>`, `<object>`, `<embed>`, or `<iframe>` elements.  

More info about mode is [here](https://developer.mozilla.org/en-US/docs/Web/API/Request/mode).  

There's also a way to define [referrer policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy) using special [option](https://developer.mozilla.org/en-US/docs/Web/API/Request/referrerPolicy).

## Handling body

## Handling response byte by byte

It's a well known fact that all the data transmitted in internet by a pack of chunks of bytes.  
`fetch()` allows to handle the receiving data in real time (byte by byte) without waiting for full data.  
It's possible due to [Stream API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API). 
`body` property contains a [Readable Stream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) with the body contents that have been added to the request.  

Example:  
```typescript
fetch('https://www.example.org')
  .then((response) => response.body)
  .then((rb) => {
    const reader = rb.getReader();

    return new ReadableStream({
      start(controller) {
        // The following function handles each data chunk
        function push() {
          // "done" is a Boolean and value a "Uint8Array"
          reader.read().then(({ done, value }) => {
            // If there is no more data to read
            if (done) {
              console.log('done', done);
              controller.close();
              return;
            }
            // Get the data and send it to the browser via the controller
            controller.enqueue(value);
            // Check chunks by logging to the console
            console.log(done, value);
            push();
          });
        }

        push();
      },
    });
  })
  .then((stream) =>
    // Respond with our stream
    new Response(stream, { headers: { 'Content-Type': 'text/html' } }).text()
  )
  .then((result) => {
    // Do things with result
    console.log(result);
  });
```

## Cache handling

It's possible to define a cache mode that controls how interaction with a browsers cache.  
`cache` option is used for it.  More about it [here](https://developer.mozilla.org/en-US/docs/Web/API/Request/cache).

## Tell what is being requested

There's a way to tell a user agent what kind of data is being requested (document, audio etc.).  
The `destination` option is used for it.


## Manipulating request info

To set up a request info, `new Request()` constructor can be used.   
To set up a header, `new Headers()` constructor can be used.


## Usage in web workers

*\*About web workers you can read [here](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API).*

## References
1. [About fetch api](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)  
2. [Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)  
3. [About request with credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#requests_with_credentials)  
