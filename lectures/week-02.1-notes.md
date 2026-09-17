# Week 2.1 - Notes and Examples

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.js_
(https://snack.expo.dev)

## Common Components

- These are covered in depth in a later week. For now, know they exist and how they differ from HTML.
- Build on the HTML to React Native table from Week 1.2.

| HTML                       | React Native | What is different                                                     |
| -------------------------- | ------------ | --------------------------------------------------------------------- |
| `<div>`                    | `View`       | Container only, cannot hold bare text                                 |
| `<p>`, `<h1>`, `<span>`    | `Text`       | The only place text can render. A `Text` inside a `Text` acts like a `<span>` |
| `<button>`                 | `Button`     | Label goes in the `title` prop. Uses `onPress`, not `onClick`         |
| `<input type="text">`      | `TextInput`  | Uses `onChangeText`, which gives you the string, not an event object  |
| `<input type="checkbox">`  | `Switch`     | Renders as a toggle. Uses `value` and `onValueChange`                 |
| `<img>`                    | `Image`      | Uses `source`, not `src`. Remote images need a width and height       |
| A scrolling page           | `ScrollView` | Screens do not scroll by default, content must be wrapped             |

- Notice the event names change too: `onPress`, `onChangeText`, `onValueChange`. That leads into event handlers.

## Example: Event Handlers

- Create an app with a Button.
- Add the event handler using a function (defined in the component).
- Must run this on a device or emulator (Snack's iOS/Android preview or Expo Go). `Alert.alert` does nothing in the Web preview.

```jsx
import { View, Button, Alert } from "react-native";

function MyButton() {
  function handlePress() {
    Alert.alert("That felt good");
  }

  return <Button onPress={handlePress} title="I'm a button" />;
}

export default function MyApp() {
  return (
    <View style={{ marginTop: 100 }}>
      <MyButton />
    </View>
  );
}
```

- Show that the function can use arrow notation too.

```jsx
function MyButton() {
  const handlePress = () => {
    Alert.alert("That felt good");
  };

  return <Button onPress={handlePress} title="I'm a button" />;
}
```

- Now show the event handler can be defined inline as well.

```jsx
function MyButton() {
  return (
    <Button onPress={() => Alert.alert("That felt good")} title="I'm a button" />
  );
}
```

## Example: Passing Props

- Modify the button to accept props, `{text, enabled}`.
- Set a default value `enabled = true`.
- Set a default value `text = "Button"`.
- Wire the button to read the props.

```jsx
function MyButton({ text = "Button", enabled = true }) {
  return (
    <Button
      disabled={!enabled}
      title={text}
      onPress={() => Alert.alert("That felt good")}
    />
  );
}
```

- Pass in values from the parent.

```jsx
export default function MyApp() {
  return (
    <View style={{ marginTop: 100 }}>
      <MyButton text="Click Me!" />
    </View>
  );
}
```

- Add another prop `message` and set the default value to `That felt good`.
- Wire the alert to receive the message.
- NOTE: You are already in a JS code block, you don't need the `{ }` brackets.

```jsx
import { View, Button, Alert } from "react-native";

function MyButton({
  text = "Button",
  enabled = true,
  message = "That felt good",
}) {
  return (
    <Button
      disabled={!enabled}
      title={text}
      onPress={() => Alert.alert(message)}
    />
  );
}

export default function MyApp() {
  return (
    <View style={{ marginTop: 100 }}>
      <MyButton text="Click Me!" message="That felt great!" />
    </View>
  );
}
```
