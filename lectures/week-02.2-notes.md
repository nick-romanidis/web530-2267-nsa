# Week 2.2 - Notes and Examples

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.js_
(https://snack.expo.dev)

## Example: State

- Start from this version.

```jsx
import { View, Button } from "react-native";

function MyButton({ enabled = true }) {
  return <Button disabled={!enabled} title="Click Me" />;
}

export default function MyApp() {
  return (
    <View style={{ marginTop: 100 }}>
      <MyButton />
    </View>
  );
}
```

- Add a variable to the button to count how many times it is clicked.

```jsx
import { View, Button, Alert } from "react-native";

function MyButton({ enabled = true }) {
  let count = 0;

  return (
    <Button
      disabled={!enabled}
      onPress={() => {
        count++;
        Alert.alert(`${count}`);
      }}
      title="Click Me"
    />
  );
}
```

- Show the click count on the button.
- It will not refresh because we are not using state.

```jsx
function MyButton({ enabled = true }) {
  let count = 0;

  return (
    <Button
      disabled={!enabled}
      onPress={() => {
        count++;
        Alert.alert(`${count}`);
      }}
      title={`Button clicked ${count} times`}
    />
  );
}
```

- Let's fix this by adding state.
- Let's remove the Alert as well.

```jsx
import { useState } from "react";
import { View, Button } from "react-native";

function MyButton({ enabled = true }) {
  const [count, setCount] = useState(0);

  return (
    <Button
      disabled={!enabled}
      onPress={() => {
        setCount(count + 1);
      }}
      title={`Button clicked ${count} times`}
    />
  );
}

export default function MyApp() {
  return (
    <View style={{ marginTop: 100 }}>
      <MyButton />
    </View>
  );
}
```

- Show what goes wrong when the new value depends on the old one and we do not pass a function.
- Change the button to add 2 by calling `setCount` twice.

```jsx
function MyButton({ enabled = true }) {
  const [count, setCount] = useState(0);

  return (
    <Button
      disabled={!enabled}
      onPress={() => {
        setCount(count + 1);
        setCount(count + 1);
      }}
      title={`Button clicked ${count} times`}
    />
  );
}
```

- Press it. The count goes up by 1, not 2.
- `count` is a plain variable that was filled in when the component rendered. Both calls read the same value, so both say "make it 1". React batches them and applies the same value twice.
- Now change both calls to `setCount((prev) => prev + 1)`. React runs each function on the latest value, so the count goes up by 2.
- To avoid stale state, use this syntax. When the new value depends on the old one, pass a function.

```jsx
function MyButton({ enabled = true }) {
  const [count, setCount] = useState(0);

  return (
    <Button
      disabled={!enabled}
      onPress={() => {
        setCount((prev) => prev + 1);
      }}
      title={`Button clicked ${count} times`}
    />
  );
}
```

## Example: Lifting State Up

- Lift state up, so that the parent knows when the value changes.
- Move the press handler outside of the function and pass as a prop.

```jsx
import { useState } from "react";
import { Text, View, Button } from "react-native";

function MyButton({ enabled = true, handlePress, count }) {
  return (
    <Button
      disabled={!enabled}
      onPress={handlePress}
      title={`Button clicked ${count} times`}
    />
  );
}

export default function MyApp() {
  const [count, setCount] = useState(0);

  return (
    <View style={{ marginTop: 100 }}>
      <Text>Button clicked {count} times</Text>
      <MyButton
        count={count}
        handlePress={() => {
          setCount((prev) => prev + 1);
        }}
      />
    </View>
  );
}
```

## Example: useEffect

- Add a side effect.
- The function returned from the effect is the clean-up. Here it cancels the timer.

```jsx
import { useState, useEffect } from "react";
import { Text, View, Button } from "react-native";

function MyButton({ enabled = true, handlePress, count }) {
  useEffect(() => {
    // Initialize the component

    // Start async task
    const id = setTimeout(() => {
      handlePress();
    }, 1000);

    return () => {
      // Cleanup code here
      clearTimeout(id);
    };
  });

  return (
    <Button
      disabled={!enabled}
      onPress={handlePress}
      title={`Button clicked ${count} times`}
    />
  );
}

export default function MyApp() {
  const [count, setCount] = useState(0);

  return (
    <View style={{ marginTop: 100 }}>
      <Text>Button clicked {count} times</Text>
      <MyButton
        count={count}
        handlePress={() => {
          setCount(count + 1);
        }}
      />
    </View>
  );
}
```

- Notice the program keeps counting and never stops!
- Each value change causes a render which causes useEffect to fire.
- Add a dependency array so it runs only after the first render.
- Notice the warning due to the empty `[]`.

```jsx
function MyButton({ enabled = true, handlePress, count }) {
  useEffect(() => {
    // Initialize the component

    // Start async task
    const id = setTimeout(() => {
      handlePress();
    }, 1000);

    return () => {
      // Cleanup code here
      clearTimeout(id);
    };
  }, []); // Safe to ignore

  return (
    <Button
      disabled={!enabled}
      onPress={handlePress}
      title={`Button clicked ${count} times`}
    />
  );
}
```

- To get rid of the warning we can use `useCallback`.
- `useCallback` keeps the same function between renders, so `[handlePress]` does not change and the effect only runs once.

```jsx
import { useState, useEffect, useCallback } from "react";
import { Text, View, Button } from "react-native";

function MyButton({ enabled = true, handlePress, count }) {
  useEffect(() => {
    // Initialize the component

    // Start async task
    const id = setTimeout(() => {
      handlePress();
    }, 1000);

    return () => {
      // Cleanup code here
      clearTimeout(id);
    };
  }, [handlePress]);

  return (
    <Button
      disabled={!enabled}
      onPress={handlePress}
      title={`Button clicked ${count} times`}
    />
  );
}

export default function MyApp() {
  const [count, setCount] = useState(0);

  const handlePress = useCallback(() => {
    setCount((prev) => prev + 1);
  }, []);

  return (
    <View style={{ marginTop: 100 }}>
      <Text>Button clicked {count} times</Text>
      <MyButton count={count} handlePress={handlePress} />
    </View>
  );
}
```

## Example: Using Context for State

- Start from this version. It is the lifted state example with the `useEffect` and `useCallback` removed. Copy and paste it into `App.js`.

```jsx
import { useState } from "react";
import { Text, View, Button } from "react-native";

function MyButton({ enabled = true, handlePress, count }) {
  return (
    <Button
      disabled={!enabled}
      onPress={handlePress}
      title={`Button clicked ${count} times`}
    />
  );
}

export default function MyApp() {
  const [count, setCount] = useState(0);

  const handlePress = () => {
    setCount((prev) => prev + 1);
  };

  return (
    <View style={{ marginTop: 100 }}>
      <Text>Button clicked {count} times</Text>
      <MyButton count={count} handlePress={handlePress} />
    </View>
  );
}
```

- Create the context and a provider component.
- Merge the context into the button.
- Create a new component for displaying the information.
- Update the MyApp component.

```jsx
import { useState, createContext, useContext } from "react";
import { Text, View, Button } from "react-native";

// Create the Context
const CountContext = createContext();

function CountProvider({ children }) {
  const [count, setCount] = useState(0);

  const handlePress = () => {
    setCount((prev) => prev + 1);
  };

  return (
    <CountContext.Provider value={{ count, handlePress }}>
      {children}
    </CountContext.Provider>
  );
}

function MyButton({ enabled = true }) {
  const { count, handlePress } = useContext(CountContext);

  return (
    <Button
      disabled={!enabled}
      onPress={handlePress}
      title={`Button clicked ${count} times`}
    />
  );
}

function CountDisplay() {
  const { count } = useContext(CountContext);
  return <Text>Button clicked {count} times</Text>;
}

export default function MyApp() {
  return (
    <CountProvider>
      <View style={{ marginTop: 100 }}>
        <CountDisplay />
        <MyButton />
      </View>
    </CountProvider>
  );
}
```

## Example: Using Jotai

- Jotai makes it so much easier.
- A tiny state-management library that gives you global state with the same simplicity as useState.
- Instead of building context providers, reducers, or stores, you create small "atoms" that behave like shared pieces of state.
- Any component can read or update an atom directly, without prop-drilling or boilerplate.

_package.json_

```js
{
  "dependencies": {
    "react-native-paper": "4.9.2",
    "@expo/vector-icons": "^15.0.3",
    "jotai": "*",
    "@babel/template": "*",
    "@types/react": "*"
  }
}
```

- Start from this version. It is the lifted state example with `CountDisplay` added and the Context removed. Copy and paste it into `App.js`.

```jsx
import { useState } from "react";
import { Text, View, Button } from "react-native";

function MyButton({ enabled = true, handlePress, count }) {
  return (
    <Button
      disabled={!enabled}
      onPress={handlePress}
      title={`Button clicked ${count} times`}
    />
  );
}

function CountDisplay({ count }) {
  return <Text>Button clicked {count} times</Text>;
}

export default function MyApp() {
  const [count, setCount] = useState(0);

  const handlePress = () => {
    setCount((prev) => prev + 1);
  };

  return (
    <View style={{ marginTop: 100 }}>
      <CountDisplay count={count} />
      <MyButton count={count} handlePress={handlePress} />
    </View>
  );
}
```

- Create a global atom above the components.
- Replace the `useState` in `MyApp` with nothing. `MyApp` no longer owns the count.
- In `MyButton` and `CountDisplay`, remove the props and call `useAtom(countAtom)` instead. It returns the same `[value, setter]` pair as `useState`.
- Notice there is no provider to wrap and no props to pass.

_App.js_

```jsx
import { atom, useAtom } from "jotai";
import { Text, View, Button } from "react-native";

// Global atom
const countAtom = atom(0);

function MyButton({ enabled = true }) {
  const [count, setCount] = useAtom(countAtom);

  return (
    <Button
      disabled={!enabled}
      onPress={() => setCount((prev) => prev + 1)}
      title={`Button clicked ${count} times`}
    />
  );
}

function CountDisplay() {
  const [count] = useAtom(countAtom);
  return <Text>Button clicked {count} times</Text>;
}

export default function MyApp() {
  return (
    <View style={{ marginTop: 100 }}>
      <CountDisplay />
      <MyButton />
    </View>
  );
}
```
