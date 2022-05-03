# Hooks

## Lazy Initialization 

Pass a function to a `useState` call, so that the initial value is only calculated once. Could be useful for expensive operations. For example:

```js
const getInitialState = () => Number(window.localStorage.getItem('count'))
const [count, setCount] = React.useState(getInitialState)
```

Learn [more](https://kentcdodds.com/blog/use-state-lazy-initialization-and-function-updates)

<hr>

## Dispatch Functions

Pass a function to a dispatch (setState) call to get access to the previous state. The following example wouldn't work without a function because while `doSomethingAsync()` is running, every call to increment will have the same value for the count variable

```js
function DelayedCounter() {
  const [count, setCount] = React.useState(0)
  const increment = async () => {
    await doSomethingAsync()
    // setCount(count + 1) Will not work
    setCount(previousCount => previousCount + 1)
  }
  return <button onClick={increment}>{count}</button>
}
```

> Any time I need to compute new state based on previous state, I use a function update. - Raymond Felton

Learn [more](https://kentcdodds.com/blog/use-state-lazy-initialization-and-function-updates)

<hr>

## Effect Dependencies
When you want to limit the number of times a `useEffect` callback runs pass an array of dependencies as the second argument. Otherwise the callback will run whenever the component updates. 

```js
const [name, setName] = React.useState('Cake Boss')

  React.useEffect(() => {
    window.localStorage.setItem('name', name)
  }, [name]) // Only runs when name is updated
```

Learn [more](https://blog.logrocket.com/guide-to-react-useeffect-hook/#:~:text=Dependencies%20are%20array%20items%20provided,based%20on%20every%20effect's%20dependencies)
<hr>

## Effect Cleanup
If you return a function in a `useEffect` callback it will be called when the component unmounts

> For this reason, you can't make the callback an `async` function because that would implicitly return a `Promise`

<hr>

## Custom Hooks
A function (whose name starts with use...) that uses other hooks inside of it

```js
function useLocalStorageState({key, defaultValue = ''}) {
  const [state, setState] = React.useState(
    () => window.localStorage.getItem(key) || defaultValue,
  )

  React.useEffect(() => {
    window.localStorage.setItem(key, state)
  }, [key, state])

  return [state, setState]
}
```

## Hook Flow

The order in which operations occur can be viewed in the following diagram.

![hook flow](./images/hook-flow.png)
<hr>

## DOM Interactions
Anytime you need to interact with a DOM node you'll need to `useRef()` to get access to the node

```js
function Tilt({children}) {
  const tiltRef = React.useRef()

  React.useEffect(() => {
    const tiltNode = tiltRef.current
    /** 
     * Do things with tiltNode
    */
  }, [])

  return (
    <div ref={tiltRef} className="tilt-root">
      <div className="tilt-child">{children}</div>
    </div>
  )
}
```
<hr>

## useReducer
Basically it allows you to compare the current state with the new state (passed into the dispatch function) and return a derived state. 

```js
function countReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return {...state, count: state.count + action.step}
    default:
      throw new Error(`Unsupported action type: ${action.type}`)
  }
}

function Counter({initialCount = 0, step = 2}) {
  const [state, dispatch] = React.useReducer(countReducer, {
    count: initialCount,
  })
  const {count} = state
  const increment = () => dispatch({type: 'INCREMENT', step})

  return <button onClick={increment}>{count}</button>
}
```
[Should I useState or useReducer?](https://kentcdodds.com/blog/should-i-usestate-or-usereducer)

## <a id="component-composition" style="color: inherit;text-decoration: none;">Component Composition</a>
Instead of passing one parent component with a bunch of child components in it, for example: 

```js
    return <Dashboard games={games}/>
```

Return the dashboard with the child components inside, like the following:

```js
    return (
        <Dashboard>
            <Breadcrumbs />
            <SplitLayout>
                <Filters />
                <Videogames games={games} />
            </SplitLayout>
        </Dashboard>
    )

```
This avoids two layers of prop drilling and allows for customization. 

> Be aware you'll need to render the {children} in the Dashboard component
<hr>

## useContext
Context is a way of sharing essentially global data with your components. It has some good uses like <b>themes or user preferences</b>, but if you're using it to bypass prop drilling <a href="#component-composition">component composition</a> is a better solution. 

```js
const ThemeContext = React.createContext();

function Button() {
    const {darkMode} = React.useContext(ThemeContext);
    const themeMode = darkMode ? 'dark-btn' : 'light-btn';
    return <button className={themeMode}>Click</button>
}

function App() {
    return (
        <ThemeContext.Provider value={{darkMode: true}}>
         <Button>
        </ThemeContext.Provider>
    )
}

```

<hr>

## useLayoutEffect
Run an effect before the browser paints. Useful when you are making an observable change to the DOM which requires the browser to paint that update. 99% of the time you'll use `useEffect`.

[useEffect vs useLayoutEffect](https://kentcdodds.com/blog/useeffect-vs-uselayouteffect)

<hr>

## Compound Components
A good example of a compound component is the `select` and `options` HTML tags.

```html
    <select>
        <option value="1">Option 1</option>
        <option value="2">Option 2</option>
    </select>
```
To use one without the other wouldn't make sense, but together they make for a very useful pattern. The `select` statement manages the state and the `options` act as configuration. 

A common way to implement this pattern in React is as follows:

```js
function Foo({children}) {
  return React.Children.map(children, (child, index) => {
    return React.cloneElement(child, {
      id: `i-am-child-${index}`,
    })
  })
}

function Bar() {
  return (
    <Foo>
      <div>I will have id "i-am-child-0"</div>
      <div>I will have id "i-am-child-1"</div>
      <div>I will have id "i-am-child-2"</div>
    </Foo>
  )
}
```

More info about [React Children](https://reactjs.org/docs/react-api.html#reactchildren) and
[React Clone](https://reactjs.org/docs/react-api.html#cloneelement).


