## Share the same object instance between /pages/api and the rest code

Next.js splits each API route into its own bundle, so single instance inside module scope will not work, because module is re-created when user enters another route.

So far only solution that works in case of need of sharing the same class instance between /pages/api and the rest of code (sharing levelDB instance between API routes for example) is to use global namespace object. Like this:
```
const object = { name: 'Anonymous' }
if (!global.__object__) {
    // save DB instance in global object for reuse
    global.__object__ = object
}
```
