# React patterns

React patterns are the way to create such components that, depending on component's sophistication,
become easy to support, change, implement and understand.  

Possible criterias are:
* flexibility
* lucidness
* stability
* extensibility
* clear api

## Inversion of controll (not pattern, but related to patterns like custom hooks or render props)


## Compound components

For such components common state can be shared via `react context`.
Example: 
// index.tsx
```tsx
import React from "react"
import { Counter } from "./Counter"

function Example() {
  const handleChangeCounter = (count) => console.log("count", count)

  return (
    <Counter onChange={handleChangeCounter} initialValue={5}>
      <Counter.Decrement/>
      <Counter.Label>Counter</Counter.Label>
      <Counter.Count max={10} />
      <Counter.Increment/>
    </Counter>
  );
}
```
// useCounterContext.tsx
```tsx
import React from "react"

const CounterContext = React.createContext(undefined)

function CounterProvider({ children, value }) {
  return <CounterContext.Provider value={value}>{children}</CounterContext.Provider>
}

function useCounterContext() {
  const context = React.useContext(CounterContext)
  if (context === undefined) throw new Error("useCounterContext must be used within a CounterProvider")
  return context;
}

export { CounterProvider, useCounterContext }
```
// Counter.tsx
```tsx
import React, { useEffect, useRef, useState } from "react"
import { CounterProvider } from "./useCounterContext"
import { Count, Label, Decrement, Increment } from "./components"

function Counter({ children, initialValue = 0 }) {
  const [count, setCount] = useState(initialValue);

  const handleIncrement = () => setCount(value => value + 1)
  const handleDecrement = () => setCount(value => Math.max(0, value - 1))

  return (
    <CounterProvider value={{ count, handleIncrement, handleDecrement }}>
      <div>{children}</div>
    </CounterProvider>
  );
}

Counter.Count = Count;
Counter.Label = Label;
Counter.Increment = Increment;
Counter.Decrement = Decrement;

export { Counter };
```
// components.ts
```tsx
import React from "react"
import { useCounterContext } from "./useCounterContext"

const Count = ({ max }) => {
  const { count } = useCounterContext()
  return <span style={{color: max && count >= max} && 'red'}>{count}</span>
}

const Decrement = () => <button onClick={useCounterContext().handleDecrement}>-</button>
const Increment = () => <button onClick={useCounterContext().handleIncrement}>+</button>
const Label = ({ children }) => <p>{children}</p>;

export { Count, Decrement, Increment, Label };
```
___

## With contolled props

Common state is defined outside the component
Example:
// index.tsx
```tsx
import React from "react"
import { Counter } from "./Counter"

function Example() {
  const [count, setCount] = useState(0)
  const handleChangeCounter = (newCount) => setCount(newCount)

  return (
    <Counter value={count} onChange={handleChangeCounter}>
      <Counter.Decrement/>
      <Counter.Label>Counter</Counter.Label>
      <Counter.Count max={10} />
      <Counter.Increment/>
    </Counter>
  );
}
```
// useCounterContext.tsx
```tsx
import React from "react"

const CounterContext = React.createContext(undefined)

function CounterProvider({ children, value }) {
  return <CounterContext.Provider value={value}>{children}</CounterContext.Provider>
}

function useCounterContext() {
  const context = React.useContext(CounterContext)
  if (context === undefined) throw new Error("useCounterContext must be used within a CounterProvider")
  return context;
}

export { CounterProvider, useCounterContext }
```
// Counter.tsx
```tsx
import React, { useEffect, useRef, useState } from "react"
import { CounterProvider } from "./useCounterContext"
import { Count, Label, Decrement, Increment } from "./components"

function Counter({ children, value = null, initialValue = 0, onChange }) {
    const [count, setCount] = useState(initialValue)

    const isControlled = value !== null && !!onChange

    const getCount = () => (isControlled ? value : count)

    const firstMounded = useRef(true)
    
    useEffect(() => {
        if (!firstMounded.current && !isControlled) {
            onChange && onChange(count)
        }
        firstMounded.current = false
    }, [count, onChange, isControlled]);

    const handleIncrement = () => handleCountChange(getCount() + 1)

    const handleDecrement = () => handleCountChange(Math.max(0, getCount() - 1))

    const handleCountChange = (newValue) => isControlled 
        ? onChange(newValue) 
        : setCount(newValue)

    return (
        <CounterProvider
            value={{ count: getCount(), handleIncrement, handleDecrement }}
        >
            <div>{children}</div>
        </CounterProvider>
    );
}

Counter.Count = Count;
Counter.Label = Label;
Counter.Increment = Increment;
Counter.Decrement = Decrement;

export { Counter };
```
// components.ts
```tsx
import React from "react"
import { useCounterContext } from "./useCounterContext"

function Count({ max }) {
  const { count } = useCounterContext()
  return <span style={{color: max && count >= max} && 'red'}>{count}</span>
}

const Decrement = () => <button onClick={useCounterContext().handleDecrement}>-</button>
const Increment = () => <button onClick={useCounterContext().handleIncrement}>-</button>
const Label = ({ children }) => <p>{children}</p>

export { Count, Decrement, Increment, Label }
```
___

## Custom hooks

Custom hooks can be used to incupsulate the main logic of the component.  
Example:  
// index.tsx
```tsx
import React from "react"
import { Counter } from "./Counter"
import { useCounter } from "./useCounter"

function Usage() {
    const { count, handleIncrement, handleDecrement } = useCounter(0)
    const MAX_COUNT = 10;

    const handleClickIncrement = () => {
        //Put your custom logic
        if (count < MAX_COUNT) handleIncrement()
    };

    return (
        <>
            <Counter value={count}>
                <Counter.Decrement
                    icon={"minus"}
                    onClick={handleDecrement}
                    disabled={count === 0}
                />
                <Counter.Label>Counter</Counter.Label>
                <Counter.Count />
                <Counter.Increment
                    icon={"plus"}
                    onClick={handleClickIncrement}
                    disabled={count === MAX_COUNT}
                />
            </Counter>
            <div>
                <button onClick={handleClickIncrement} disabled={count === MAX_COUNT}>
                    Increment button
                </button>
            </div>
        </>
    );
}

export { Usage }
```
// useCounter.ts
```tsx
import { useState } from "react";

function useCounter(intialeCount) {
  const [count, setCount] = useState(intialeCount);

  const handleIncrement = () => setCount((prevCount) => prevCount + 1)
  const handleDecrement = () => setCount((prevCount) => Math.max(0, prevCount - 1))

  return { count, handleIncrement, handleDecrement }
}
```
// Counter.tsx
```tsx
import React, { useRef, useEffect } from "react"
import styled from "styled-components"
import { CounterProvider } from "./useCounterContext"
import { Count, Label, Decrement, Increment } from "./components"

function Counter({ children, value: count, onChange }) {
  const firstMounded = useRef(true)
  useEffect(() => {
    if (!firstMounded.current) onChange?.(count)
    firstMounded.current = false
  }, [count, onChange]);

  return (
    <CounterProvider value={{ count }}>
      <div>{children}</div>
    </CounterProvider>
  );
}

Counter.Count = Count
Counter.Label = Label
Counter.Increment = Increment
Counter.Decrement = Decrement

export { Counter }
```
// components.ts
```tsx
import React from "react"
import { useCounterContext } from "./useCounterContext"

function Count({ max }) {
  const { count } = useCounterContext()
  return <span style={{color: max && count >= max} && 'red'}>{count}</span>
}

const Decrement = ({onClick}) => <button onClick={onClick}>-</button>
const Increment = ({onClick}) => <button onClick={onClick}>+</button>
const Label = ({ children }) => <p>{children}</p>;

export { Count, Decrement, Increment, Label };
```
___

## Props getter
// index.tsx
```tsx
import React from "react"
import { Counter } from "./Counter"
import { useCounter } from "./useCounter"

const MAX_COUNT = 10

function Usage() {
  const {
    count,
    getCounterProps,
    getIncrementProps,
    getDecrementProps
  } = useCounter({
    initial: 0,
    max: MAX_COUNT
  })

  const handleBtn1Clicked = () => console.log("btn 1 clicked")

  return (
    <>
      <Counter {...getCounterProps()}>
        <Counter.Decrement {...getDecrementProps()} />
        <Counter.Label>Counter</Counter.Label>
        <Counter.Count />
        <Counter.Increment {...getIncrementProps()} />
      </Counter>
      <div>
        <button {...getIncrementProps({ onClick: handleBtn1Clicked })}>
          Custom increment 1
        </button>
      </div>
      <div>
        <button {...getIncrementProps({ disabled: count > MAX_COUNT - 2 })}>
          Custom increment 2
        </button>
      </div>
    </>
  );
}

export { Usage }
```

// useCounter.tsx
```tsx
import { useState } from "react";

//Function which concat all functions together
const callFnsInSequence = (...fns) => (...args) =>
  fns.forEach((fn) => fn && fn(...args));

function useCounter({ initial, max }) {
  const [count, setCount] = useState(initial);

  const handleIncrement = () => {
    setCount((prevCount) => Math.min(prevCount + 1, max));
  };

  const handleDecrement = () => {
    setCount((prevCount) => Math.max(0, prevCount - 1));
  };

  //props getter for 'Counter'
  const getCounterProps = ({ ...otherProps } = {}) => ({
    value: count,
    "aria-valuemax": max,
    "aria-valuemin": 0,
    "aria-valuenow": count,
    ...otherProps
  });

  //props getter for 'Decrement'
  const getDecrementProps = ({ onClick, ...otherProps } = {}) => ({
    onClick: callFnsInSequence(handleDecrement, onClick),
    disabled: count === 0,
    ...otherProps
  });

  //props getter for 'Increment'
  const getIncrementProps = ({ onClick, ...otherProps } = {}) => ({
    onClick: callFnsInSequence(handleIncrement, onClick),
    disabled: count === max,
    ...otherProps
  })

  return {
    count,
    handleIncrement,
    handleDecrement,
    getCounterProps,
    getDecrementProps,
    getIncrementProps
  }
}

export { useCounter }
```

// Counter.tsx
```tsx
import React, { useRef, useEffect } from "react"
import { CounterProvider } from "./useCounterContext"
import { Count, Label, Decrement, Increment } from "./components"

function Counter({ children, value: count, onChange }) {
  const firstMounded = useRef(true)
    
  useEffect(() => {
    if (!firstMounded.current) onChange?.(count)
    firstMounded.current = false
  }, [count, onChange])

  return (
    <CounterProvider value={{ count }}>
      <div>{children}</div>
    </CounterProvider>
  )
}
Counter.Count = Count
Counter.Label = Label
Counter.Increment = Increment
Counter.Decrement = Decrement

export { Counter }
```

// components.ts
```tsx
import React from "react"
import { useCounterContext } from "./useCounterContext"

function Count({ max }) {
  const { count } = useCounterContext()
  return <span style={{color: max && count >= max} && 'red'}>{count}</span>
}

const Decrement = ({onClick, ...props}) => <button onClick={onClick} {...props}>-</button>
const Increment = ({onClick, ...props}) => <button onClick={onClick} {...props}>+</button>
const Label = ({ children }) => <p>{children}</p>

export { Count, Decrement, Increment, Label }
```

## Render props
https://www.developerway.com/posts/higher-order-components-in-react-hooks-era

## HOC pattern

## Composition pattern
https://www.developerway.com/posts/components-composition-how-to-get-it-right


## References
1. https://proglib.io/p/5-prodvinutyh-patternov-react-razrabotki-2021-05-30
2. https://reactjs.org/docs/render-props.html
