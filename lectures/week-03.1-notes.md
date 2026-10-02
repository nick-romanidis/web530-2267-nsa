# Week 3.1 - Notes and Examples

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.js_
(https://snack.expo.dev)

## Example: Inline Styles

- React Native does not use CSS files.
- Styles are JavaScript objects passed to the `style` prop.
- They look like CSS, but property names are camelCase (`fontSize`, not `font-size`) and sizes are plain numbers, not `px`.
- Show a `View` and a `Text` styled inline.

```jsx
import { View, Text } from "react-native";

export default function MyApp() {
  return (
    <View style={{ padding: 20, marginTop: 100 }}>
      <Text style={{ fontSize: 24, color: "blue" }}>Hello Styles!</Text>
    </View>
  );
}
```

- Notice the double braces. The outer `{ }` switches JSX into JavaScript, the inner `{ }` is the style object.
- Inline styles are fine for quick demos, but they get messy fast and cannot be reused.

## Example: StyleSheet.create()

- Move the styles into a `StyleSheet` at the bottom of the file.
- Each key (`container`, `title`) is a named style you can reuse.

```jsx
import { View, Text, StyleSheet } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello Styles!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
    marginTop: 100,
    backgroundColor: "#eee",
  },
  title: {
    fontSize: 28,
    fontWeight: "bold",
  },
});
```

- It keeps the JSX readable and puts all the styling in one place.
- A named style can be reused by as many components as you like.
- Show that a typo is silently ignored. Change `fontSize` to `fontSise`: there is no warning, the title just goes back to the default size. Older versions of React Native checked style names here, but that check was removed. TypeScript catches these typos, which we cover later in the course.
- `styles` is used before it is declared. That works because `MyApp` does not run until after the whole file has loaded.

## Example: Layout with Flexbox

- Flexbox lays items out in one direction at a time, a row or a column.
- It solves the classic CSS headaches: centring content vertically, giving children equal widths, and making columns the same height.
- Flexbox guide: (https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- Practice: Flexbox Froggy (https://flexboxfroggy.com/)
- Start with a container holding three coloured boxes.
- The `style` prop also accepts an array. Styles are merged left to right, so later entries win.

```jsx
import { View, StyleSheet } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.container}>
      <View style={[styles.box, styles.red]} />
      <View style={[styles.box, styles.green]} />
      <View style={[styles.box, styles.blue]} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 100,
    backgroundColor: "#eee",
  },
  box: {
    width: 80,
    height: 80,
  },
  red: { backgroundColor: "tomato" },
  green: { backgroundColor: "mediumseagreen" },
  blue: { backgroundColor: "dodgerblue" },
});
```

- The boxes stack top to bottom. In React Native `flexDirection` defaults to `"column"`. On the web it defaults to `"row"`.
- `flex: 1` on the container makes it fill the space it is given, which here is the rest of the screen.
- Change the direction to a row.

```jsx
  container: {
    flex: 1,
    marginTop: 100,
    backgroundColor: "#eee",
    flexDirection: "row",
  },
```

- Centre the boxes both ways.
- `justifyContent` works along the main axis (the `flexDirection`).
- `alignItems` works along the cross axis (the other direction).

```jsx
  container: {
    flex: 1,
    marginTop: 100,
    backgroundColor: "#eee",
    flexDirection: "row",
    justifyContent: "center",
    alignItems: "center",
  },
```

- Try `justifyContent` with `"space-between"`, `"space-around"`, and `"space-evenly"`.
- Switch `flexDirection` back to `"column"`. The boxes are still centred, but now `justifyContent` moves them up and down and `alignItems` moves them left and right. The axes swap with the direction.
- Now build equal columns. Remove the fixed `width` and give each column `flex: 1` so they share the row evenly.
- Put a different amount of text in each column. They all end up the same height, because `alignItems` defaults to `"stretch"`.

```jsx
import { View, Text, StyleSheet } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.container}>
      <View style={styles.row}>
        <View style={[styles.column, styles.red]}>
          <Text>Short</Text>
        </View>
        <View style={[styles.column, styles.green]}>
          <Text>
            This column has a lot more text, but every column is still the
            same height.
          </Text>
        </View>
        <View style={[styles.column, styles.blue]}>
          <Text>Medium length</Text>
        </View>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: 100,
  },
  row: {
    flexDirection: "row",
  },
  column: {
    flex: 1,
    padding: 10,
  },
  red: { backgroundColor: "tomato" },
  green: { backgroundColor: "mediumseagreen" },
  blue: { backgroundColor: "dodgerblue" },
});
```

- Change the green column to `flex: 2` by adding `{ flex: 2 }` to the end of its style array. It now takes twice the space of the others.
- In React Native, `flex` takes a single number. It is not the `flex: 1 1 auto` shorthand from CSS.

## Example: Platform.OS

- iOS and Android sometimes need small adjustments, for example the status bar is a different height on each.
- `Platform.OS` returns a string: `"ios"`, `"android"`, or `"web"`.
- Switch between the Web, iOS, and Android tabs in the Snack preview to see each message.

```jsx
import { View, Text, Platform } from "react-native";

export default function MyApp() {
  return (
    <View style={{ marginTop: 100 }}>
      <Text>
        {Platform.OS === "ios" && "Running on iOS"}
        {Platform.OS === "android" && "Running on Android"}
        {Platform.OS === "web" && "Running on Web"}
      </Text>
    </View>
  );
}
```

- This is conditional rendering with `&&`, a very common React pattern.
  - JavaScript's `&&` returns the right side when the left side is true.
  - When the left side is false, it returns `false`.
  - React renders nothing for `false`, `null`, and `undefined`, so only one message shows.
  - Careful: React _does_ render `0`. `{items.length && <Text>...</Text>}` shows a stray `0` when the list is empty.
- `Platform.Version` gives the OS version. On Android it is a number (like `34`), on iOS a string (like `"17.4"`).

## Example: Platform-Specific Styles

- You can branch on `Platform.OS` inside the StyleSheet.

```jsx
import { View, Text, StyleSheet, Platform } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Platform-Specific Styling</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
    marginTop: 100,
    backgroundColor: Platform.OS === "ios" ? "#d0e8ff" : "#ffe8d0",
  },
  title: {
    fontSize: Platform.OS === "android" ? 30 : 24,
    fontWeight: "bold",
  },
});
```

- Useful when spacing or font sizes need to differ between platforms.

## Example: Platform.select()

- `Platform.select()` takes an object keyed by platform and returns the value for the one the app is running on.
- This is cleaner than several `Platform.OS === ...` checks.
- Keys:
  - `ios` - used only on iOS.
  - `android` - used only on Android.
  - `native` - used on both iOS and Android, but not web.
  - `web` - used only on the web.
  - `default` - the fallback when no other key matches. Optional, but recommended.
- Inside a StyleSheet, `Platform.select()` can return a whole style object. Spread it into a style with `...`.
- `StatusBar.currentHeight` is the height of the Android status bar. It is `undefined` on iOS, which is why iOS gets its own padding.

```jsx
import { View, Text, StyleSheet, Platform, StatusBar } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.container}>
      <Text>Clear of the status bar on every platform</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#eaeaea",
    ...Platform.select({
      ios: { paddingTop: 60 },
      android: { paddingTop: StatusBar.currentHeight },
      default: { paddingTop: 20 },
    }),
  },
});
```

- `Platform.select()` is not limited to styles. It can pick any value, such as a string.

```jsx
import { View, Text, Platform } from "react-native";

export default function MyApp() {
  const message = Platform.select({
    ios: "Hello from iOS",
    android: "Hello from Android",
    web: "Hello from Web",
    default: "Hello from somewhere",
  });

  return (
    <View style={{ marginTop: 100 }}>
      <Text>{message}</Text>
    </View>
  );
}
```

## Example: Platform-Specific Files

- When a component is very different on each platform, give each platform its own file.
- React Native picks the matching file automatically. The import does not mention the platform.

```
MyButton.ios.js
MyButton.android.js
MyButton.js
```

```jsx
import MyButton from "./MyButton";
```

- No conditional logic is needed in the component.
- `MyButton.js` is the fallback for any platform without its own file (the web here).

## Common Components

- Last week's table introduced these. Now we style them.

### View

- The most fundamental building block. `<div>` in HTML, `UIView` on iOS, `android.view.View` on Android.
- Can hold any number of children, including other Views.
- Styled with padding, margin, borders, background colour, and flexbox.
- Cannot hold bare text. Text must go inside a `Text`.

```jsx
import { View, Text } from "react-native";

export default function MyApp() {
  return (
    <View
      style={{ padding: 20, marginTop: 100, borderWidth: 2, borderRadius: 8 }}
    >
      <Text>Hello again</Text>
    </View>
  );
}
```

### Text

- The only component that can display text.
- An outer `Text` acts like a `<p>` (block). A `Text` inside a `Text` acts like a `<span>` (inline).
- Nested `Text` inherits styles from its parent `Text`. This is the one place React Native styles cascade. A `View` does not pass its styles down.

```jsx
import { View, Text, StyleSheet } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.container}>
      <Text style={styles.paragraph}>
        This whole sentence is grey and size 18, but{" "}
        <Text style={styles.highlight}>these words are also bold and red</Text>.
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
    marginTop: 100,
  },
  paragraph: {
    fontSize: 18,
    color: "#555",
  },
  highlight: {
    fontWeight: "bold",
    color: "crimson",
  },
});
```

- Show that styles do not cascade from a `View`. Remove `color` from `paragraph`, then add `color: "blue"` to `container` instead. In HTML and CSS, a `<div>`'s colour passes down to the text inside it. In React Native it does not, on every platform including Snack's Web preview, so the sentence turns black instead of blue.

### Button

- Renders as a native button, so it looks different on each platform.
- It does not accept a `style` prop. You can only change its `color`.
- Later we will build custom buttons that can be fully styled.

```jsx
import { View, Button, Alert } from "react-native";

export default function MyApp() {
  return (
    <View style={{ marginTop: 100, padding: 20 }}>
      <Button
        title="Save"
        color="seagreen"
        onPress={() => Alert.alert("Saved")}
      />
    </View>
  );
}
```

- To space buttons out, wrap each one in a `View` and style the `View`.
- FYI: to reuse a colour, keep it in a constant. It is not a global, just a normal variable you could move into its own file and `import`.
- On iOS `color` sets the text colour. On Android it sets the background.

```jsx
import { View, Button, Alert } from "react-native";

const colors = {
  primary: "#2196F3",
  danger: "crimson",
};

export default function MyApp() {
  return (
    <View style={{ marginTop: 100, padding: 20 }}>
      <Button
        title="Save"
        color={colors.primary}
        onPress={() => Alert.alert("Saved")}
      />
      <Button
        title="Delete"
        color={colors.danger}
        onPress={() => Alert.alert("Deleted")}
      />
    </View>
  );
}
```

### Image

- Displays an image from the web, the project's files, or a data URI.
- `source` replaces the HTML `src`.
- A remote image must be given a `width` and `height`. React Native cannot know the size until it downloads the file, so without them nothing shows.

```jsx
import { View, Image } from "react-native";

export default function MyApp() {
  return (
    <View style={{ marginTop: 100, padding: 20 }}>
      <Image
        style={{ width: 200, height: 200, borderRadius: 10 }}
        source={{ uri: "https://picsum.photos/201" }}
      />
    </View>
  );
}
```

- Notice the remote `source` is an object with a `uri` key.
- Local images use `require()` instead.
- A new Snack already has an image in its `assets` folder, `snack-icon.png`. Use that one.

```jsx
<Image source={require("./assets/snack-icon.png")} />
```

- A local image already knows its size, so `width` and `height` are optional. Add them to resize it.
- The `require()` path must be a fixed string. You cannot build it from a variable.
- Some Image props only work on Android or only on iOS. Check the docs before relying on one.

### ScrollView

- A screen does not scroll by default. Content that runs past the bottom is cut off.
- Wrap long content in a `ScrollView`.

```jsx
import { ScrollView, Text } from "react-native";

// An array of the numbers 1 to 50
const lines = Array.from({ length: 50 }, (_, i) => i + 1);

export default function MyApp() {
  return (
    <ScrollView style={{ marginTop: 100, padding: 20 }}>
      {lines.map((n) => (
        <Text key={n} style={{ fontSize: 20 }}>
          Line {n}
        </Text>
      ))}
    </ScrollView>
  );
}
```

- The 50 lines run off the bottom of the screen. Scroll to see them all.
- Change `ScrollView` to `View` and try again. The content is cut off and will not scroll.
- `.map()` with a `key` is the same pattern we used for lists in Week 1.2.
- `style` styles the scrolling window itself. `contentContainerStyle` styles the content inside it. Use `contentContainerStyle` for padding around the content and for flexbox properties like `alignItems`.
- Replace the opening `ScrollView` tag with this one. The lines are now centred and the padding scrolls with the content.

```jsx
<ScrollView
  style={{ marginTop: 100 }}
  contentContainerStyle={{ padding: 20, alignItems: "center" }}
>
```
- A `ScrollView` renders every child at once. That is fine for a page of content, but not for a list of hundreds of items. That is `FlatList`, later in the course.

### TextInput

- Captures text from the keyboard.
- Use `onChangeText`, which gives you the new string. `onChange` gives an event object instead.
- Make it a controlled component: the state holds the value, and `value` shows it.

```jsx
import { useState } from "react";
import { View, Text, TextInput, StyleSheet } from "react-native";

export default function MyApp() {
  const [name, setName] = useState("");

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="Enter your name"
        value={name}
        onChangeText={setName}
      />
      <Text style={styles.greeting}>Hello {name}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
    marginTop: 100,
  },
  input: {
    borderWidth: 1,
    borderRadius: 6,
    padding: 10,
  },
  greeting: {
    marginTop: 20,
    fontSize: 18,
  },
});
```

- A `TextInput` has no border by default. Without `borderWidth` it is invisible until you tap it.
- `onChangeText={setName}` passes the setter directly. It is the same as `onChangeText={(text) => setName(text)}`.
- Show a few of the props that change the keyboard and behaviour. Tap each field on a device to see the keyboard change.

```jsx
<TextInput
  style={styles.input}
  placeholder="Email"
  keyboardType="email-address"
  autoCapitalize="none"
  autoCorrect={false}
/>
<TextInput style={styles.input} placeholder="Password" secureTextEntry />
<TextInput style={styles.input} placeholder="Comments" multiline />
```

### Switch

- Renders as an on/off toggle for a true/false value.
- It is a controlled component. It only moves if you update the state in `onValueChange`.
- It may not render on the web, so test it in the iOS or Android preview.
- Use the switch to change the styles. A `false` entry in a style array is ignored, so `isOn && styles.dark` only applies when the switch is on.

```jsx
import { useState } from "react";
import { View, Text, Switch, StyleSheet } from "react-native";

export default function MyApp() {
  const [isOn, setIsOn] = useState(false);

  return (
    <View style={[styles.container, isOn && styles.dark]}>
      <Text style={[styles.label, isOn && styles.lightText]}>
        Dark mode is {isOn ? "on" : "off"}
      </Text>
      <Switch value={isOn} onValueChange={setIsOn} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: 100,
    alignItems: "center",
    backgroundColor: "#fff",
  },
  dark: {
    backgroundColor: "#222",
  },
  label: {
    fontSize: 18,
    marginBottom: 10,
    color: "#222",
  },
  lightText: {
    color: "#fff",
  },
});
```
