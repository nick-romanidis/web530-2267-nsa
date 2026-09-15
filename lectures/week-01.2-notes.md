# Week 1.2 - Notes and Examples

## React Playground

React Playground (Professor's CodeSpace Template)
(https://codesandbox.io/p/sandbox/react-basic-3rqll4)

## Example: PropTypes and Validation

- A component has no way to say what props it expects. PropTypes lets us declare that contract.
- React checks the contract **in development only** and prints a warning in the console. Nothing is enforced in production.
- The checks come from the separate `prop-types` package, so it must be installed and imported.
- PropTypes is deprecated. React 19 no longer checks it, and TypeScript replaces it. We will revisit this when we learn TypeScript.

```jsx
import PropTypes from "prop-types";

function Greeting({ name, age, isStudent }) {
  return (
    <div>
      <p>Hello {name}</p>
      <p>Age: {age}</p>
      <p>{isStudent ? "Student" : "Not a student"}</p>
    </div>
  );
}

// Prop validation
Greeting.propTypes = {
  name: PropTypes.string.isRequired, // must be a string
  age: PropTypes.number.isRequired, // must be a number
  isStudent: PropTypes.bool, // optional boolean
};

function MyApp() {
  return <Greeting name="Nicholas" age={42} isStudent={false} />;
}
```

- Show what happens when the contract is broken. Open the browser console and try each line.
- The app still renders. PropTypes only warns, it does not stop the component.

```jsx
function MyApp() {
  return (
    <>
      {/* OK */}
      <Greeting name="Nicholas" age={42} isStudent={false} />

      {/* Warning: Failed prop type: Invalid prop `age` of type `string`
          supplied to `Greeting`, expected `number`. */}
      <Greeting name="Nicholas" age="42" />

      {/* Warning: Failed prop type: The prop `name` is marked as required
          in `Greeting`, but its value is `undefined`. */}
      <Greeting age={42} />
    </>
  );
}
```

- Optional props should have a default so the component never renders something odd.
- Reuse the destructuring default from Week 1.1. PropTypes says "optional", the default says "what it is when omitted".

```jsx
function Greeting({ name, age, isStudent = false }) {
  return (
    <div>
      <p>Hello {name}</p>
      <p>Age: {age}</p>
      <p>{isStudent ? "Student" : "Not a student"}</p>
    </div>
  );
}

Greeting.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  isStudent: PropTypes.bool,
};
```

- There are validators for more than primitives. The ones you will use most:
  - `PropTypes.oneOf([...])` restricts a prop to a fixed set of values.
  - `PropTypes.func` for callbacks.
  - `PropTypes.arrayOf(type)` for lists.
  - `PropTypes.shape({...})` for objects with known fields.

```jsx
import PropTypes from "prop-types";

function Course({ code, term, topics, instructor, onSelect }) {
  return (
    <div>
      <h2>
        {code} ({term})
      </h2>
      <p>Instructor: {instructor.name}</p>
      <ul>
        {topics.map((topic) => (
          <li key={topic}>{topic}</li>
        ))}
      </ul>
      <button onClick={() => onSelect(code)}>Select</button>
    </div>
  );
}

Course.propTypes = {
  code: PropTypes.string.isRequired,
  term: PropTypes.oneOf(["Fall", "Winter", "Summer"]).isRequired,
  topics: PropTypes.arrayOf(PropTypes.string).isRequired,
  instructor: PropTypes.shape({
    name: PropTypes.string.isRequired,
    email: PropTypes.string,
  }).isRequired,
  onSelect: PropTypes.func.isRequired,
};

function MyApp() {
  return (
    <Course
      code="WEB530"
      term="Fall"
      topics={["JSX", "Components", "Props"]}
      instructor={{ name: "Nicholas" }}
      onSelect={(code) => alert(`Selected ${code}`)}
    />
  );
}
```

- Try `term="Spring"` or `topics="JSX"` and read the warning. The message names the prop, the component, what it got, and what it expected.

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.js_
(https://snack.expo.dev)

## Example: Introduce React Native Components (not HTML)

- Rewrite the this example using proper React Native components.

```jsx
function MyButton() {
  const buttonText = "I'm a button";
  const enable = false;

  return <button disabled={!enable}>{buttonText}</button>;
}

function MyApp() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <MyButton />
    </div>
  );
}
```

- Notice React Native component names begin with capital letters.

```jsx
import { View, Text, Button } from "react-native";

function MyButton() {
  const buttonText = "I'm a button";
  const enable = false;

  return <Button disabled={!enable} title={buttonText} />;
}

export default function MyApp() {
  return (
    <View>
      <Text>Welcome to my app</Text>
      <MyButton />
    </View>
  );
}
```

- Show that text must live inside `Text`. React Native does not allow a bare string inside a `View`.
- This crashes the app. Remove the `Text` tags below and read the error.

```jsx
import { View, Text } from "react-native";

export default function MyApp() {
  return (
    <View>
      <Text>Welcome to my app</Text>
    </View>
  );
}
```

- Error: `Text strings must be rendered within a <Text> component.`

## HTML to React Native

| HTML                 | React Native | Notes                                          |
| -------------------- | ------------ | ---------------------------------------------- |
| `<div>`              | `View`       | Container, no text directly inside             |
| `<p>`, `<h1>`, `<span>` | `Text`    | All text, no separate heading components       |
| `<button>`           | `Button`     | Label goes in the `title` prop, not as children |
| `<input>`            | `TextInput`  |                                                |
| `<img>`              | `Image`      | Needs a `source` prop                          |
| `<ul>`, `<li>`       | `View` + `Text` | Lists get their own component later       |
