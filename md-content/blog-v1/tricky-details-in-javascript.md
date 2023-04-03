## Tricks with console

### How to paint console log to colors?

`console.log` support formatter of input data.  
To apply css styles to a message printed with `console.log` - apply a `%c` [formatter](https://console.spec.whatwg.org/#log) before the message in console, 
and pass a styling in css formation as the second argument.  
  
For example:
```
const consoleStyling = `
    color: red; 
    background: black; 
    padding: 12px;
    font-weight: bold; 
    border-radius: 4px;
`

console.log(`%chello world!`, consoleStyling)
```

Or, use bash color flags:
```
const redColorFlag = `x1B[31m`
const blueColorFlag = `\x1B[34m`

console.log(`${redColorFlag}hello ${blueColorFlag}world`);
```

### How to log the code trace?

Use `console.trace()` to print the call stack of functions called before the line `console.trace`.

A bit more info [here](https://blog.devgenius.io/how-to-add-colors-in-javascript-console-log-output-e41efeffa8dd).


## Trick with setTimeout

Not many people knows that set timeout could be called with a text argument. Trick under the hood of execution is that text will be compiled to a code and executed.  

So code like this is valid:
```
const message = 'Hello'
setTimeout(`alert('${message}')`)

setTimeout({
    toString() {
        return `alert('${message}')`
    }
})
```

## Tricks with function

### Was function called as a constructor?

Use `new.target` inside a function (no the arrow one) to find out if function is called as a constructor (with `new`). If so - `new.target` returns function, otherwise it returns `undefined`.

Like:
```
function User() {
    if (new.target) console.log(`It's a contructor call`)
    else console.log(`It's a simple function call (without new)`)
}
