# Week 6.1 - Notes and Examples

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.js_
(https://snack.expo.dev)

- Keep Snack on SDK 54 (the default).
- Every example except the last one only needs `react-native`. The Swipe to Delete example uses React Native Gesture Handler, so replace the contents of `package.json` before that one.

_package.json_

```json
{
  "dependencies": {
    "react-native-gesture-handler": "~2.28.0"
  }
}
```

- This week is about fingers, and the web preview only has a mouse. A click is a tap and holding the button down is a long press, so the Pressable examples work on **Web**. Hit slop, momentum scrolling and the swipe examples need a touch screen. Choose **My Device** and open the Snack in Expo Go on your phone for those.

## Why Touch?

- Mobile apps are built around touch, not a mouse and keyboard. Every interaction is a gesture: tap, long press, scroll, swipe.
- Good touch handling feels instant. The user sees a response the moment their finger lands, not when it lifts.
- Gestures come in two kinds. A tap or a long press is discrete: one event, and it is over. A scroll or a swipe is continuous: a stream of movement, updated every frame, that can end in an action or be cancelled.
- React Native has components for the common gestures: `Pressable` for taps, `ScrollView` and `FlatList` for scrolling. Swipes need a little more work.

## Example: Pressable

- `Pressable` is the touch component. It wraps any content and reports what the finger does. It replaces the older `TouchableOpacity` and `TouchableHighlight`, which still work but are not used in new code.
- `Button` from Week 2 is fine for a plain button, but it cannot be styled. `Pressable` has no look of its own, so it can be anything: a button, a card, a list row.

### Starter Code

- Replace everything in `App.js`. It is a box styled to look like a button, and nothing happens when you press it.

```jsx
import { View, Text, StyleSheet } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.container}>
      <View style={styles.button}>
        <Text style={styles.buttonText}>Tap Me</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, alignItems: "center" },
  button: {
    width: 200,
    padding: 20,
    borderRadius: 8,
    alignItems: "center",
    backgroundColor: "#2196F3",
  },
  buttonText: { color: "white", fontSize: 18 },
  message: { marginTop: 20, marginBottom: 10, fontSize: 16 },
});
```

- A `View` has no `onPress`. Looking like a button is not enough.

### Step 1 - Make It Pressable

- Add `useState` and `Pressable` to the imports.

```jsx
import { useState } from "react";
import { View, Text, Pressable, StyleSheet } from "react-native";
```

- Add state for the tap count at the top of `MyApp`.

```jsx
const [taps, setTaps] = useState(0);
```

- Change the button `View` to a `Pressable` with an `onPress`, and show the count under it.

```jsx
<Pressable
  style={styles.button}
  onPress={() => setTaps((prev) => prev + 1)}
>
  <Text style={styles.buttonText}>Tap Me</Text>
</Pressable>
<Text style={styles.message}>Taps: {taps}</Text>
```

- Run it and tap. The count goes up, but nothing on the button changes while your finger is down. On a phone the button feels dead.

### Step 2 - Show Pressed Feedback

- `style` on a `Pressable` can be a function. It receives `{ pressed }` and returns the style to use. Replace the `style` prop.

```jsx
style={({ pressed }) => [styles.button, pressed && styles.pressed]}
```

- Add the pressed style.

```jsx
pressed: { backgroundColor: "#1565C0" },
```

- A style array applies left to right, so `styles.pressed` overrides the background. When `pressed` is `false`, the `false` in the array is ignored.
- Press and hold. The button darkens the moment the press starts and goes back when you let go. That instant change is the visual feedback every touch target needs.

### Step 3 - Log Every Press Event

- `onPress` is one of four events. Add a log to see them all. Under the `taps` state, add the list and a function that adds one entry to it.

```jsx
const [events, setEvents] = useState([]);

function log(name) {
  setEvents((prev) => [...prev, name]);
}
```

- Give the `Pressable` the other three events, and log `onPress` as well. `onPressIn` does not call `log`: it replaces the whole list, so every new press starts with an empty log and shows only its own events.

```jsx
onPress={() => {
  setTaps((prev) => prev + 1);
  log("onPress");
}}
onPressIn={() => setEvents(["onPressIn"])}
onPressOut={() => log("onPressOut")}
onLongPress={() => log("onLongPress")}
```

- Show the log under the count, with its style.

```jsx
{events.map((event, index) => (
  <Text key={index} style={styles.event}>
    {event}
  </Text>
))}
```

```jsx
event: { fontSize: 14, color: "#666" },
```

- Tap: `onPressIn`, `onPressOut`, `onPress`. The finger goes down, comes up, and only then is it a tap.
- The list is cleared in `onPressIn`, not `onPress`, because `onPressIn` is the first event of every press. `onPress` comes last, so clearing there would wipe out the two events before it.
- Hold for half a second: `onPressIn`, `onLongPress`, `onPressOut`, and no `onPress`. The count does not change. A long press replaces the tap; it is not added to it. Long press is for secondary actions, like a context menu.
- On a phone, press the button, slide your finger off it, and let go: `onPressIn`, `onPressOut`, and nothing else. Sliding off cancels the press. That is the first cancellable gesture of the week.

### Finished Code

- Compare against this if something is not working.

```jsx
import { useState } from "react";
import { View, Text, Pressable, StyleSheet } from "react-native";

export default function MyApp() {
  const [taps, setTaps] = useState(0);
  const [events, setEvents] = useState([]);

  function log(name) {
    setEvents((prev) => [...prev, name]);
  }

  return (
    <View style={styles.container}>
      <Pressable
        style={({ pressed }) => [styles.button, pressed && styles.pressed]}
        onPress={() => {
          setTaps((prev) => prev + 1);
          log("onPress");
        }}
        onPressIn={() => setEvents(["onPressIn"])}
        onPressOut={() => log("onPressOut")}
        onLongPress={() => log("onLongPress")}
      >
        <Text style={styles.buttonText}>Tap Me</Text>
      </Pressable>
      <Text style={styles.message}>Taps: {taps}</Text>
      {events.map((event, index) => (
        <Text key={index} style={styles.event}>
          {event}
        </Text>
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, alignItems: "center" },
  button: {
    width: 200,
    padding: 20,
    borderRadius: 8,
    alignItems: "center",
    backgroundColor: "#2196F3",
  },
  pressed: { backgroundColor: "#1565C0" },
  buttonText: { color: "white", fontSize: 18 },
  message: { marginTop: 20, marginBottom: 10, fontSize: 16 },
  event: { fontSize: 14, color: "#666" },
});
```

## Example: Hit Slop

- Touch targets must be big enough for a finger: at least 44 by 44 points on iOS, 48 on Android. Icons are often smaller than that.
- `hitSlop` makes the touchable area bigger without changing the layout. The box looks the same; the finger has more room.
- Run this one on your phone. The web preview ignores `hitSlop`, so a click just outside the box does nothing there.
- Replace everything in `App.js`. It is a 24-point close button, outlined in red so you can see how small it is.

```jsx
import { useState } from "react";
import { View, Text, Pressable, StyleSheet } from "react-native";

export default function MyApp() {
  const [taps, setTaps] = useState(0);

  return (
    <View style={styles.container}>
      <Pressable style={styles.close} onPress={() => setTaps((prev) => prev + 1)}>
        <Text style={styles.closeText}>X</Text>
      </Pressable>
      <Text style={styles.message}>Taps: {taps}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, alignItems: "center" },
  close: {
    width: 24,
    height: 24,
    borderWidth: 1,
    borderColor: "red",
    alignItems: "center",
    justifyContent: "center",
  },
  closeText: { fontSize: 12 },
  message: { marginTop: 40, fontSize: 16 },
});
```

- Tap just outside the red box. Nothing. Tap the edge with your thumb and it is a coin toss.
- Add `hitSlop` to the `Pressable`.

```jsx
hitSlop={20}
```

- Tap just outside the box again. It counts. The target is now 20 points bigger on every side, 64 by 64, and the layout did not move.
- `hitSlop` also takes an object for one side at a time, such as `{ top: 10, bottom: 10 }`.
- Keep slop smaller than the gap to the next target. Two overlapping touch areas mean a tap goes to the wrong one.

## Scrolling

- Scrolling is a continuous gesture. A vertical drag moves the content every frame for as long as the finger moves.
- Mobile scrolling has momentum. A flick keeps scrolling after the finger lifts and slows down on its own. A slow drag moves exactly as far as the finger. React Native uses the platform's own scrolling, so it feels like every other app on the phone.
- From Week 5: `ScrollView` renders all of its children at once and suits a page of content. `FlatList` renders only what is near the screen and suits long or fetched lists.

## Example: ScrollView

- A vertical page of rows, then a horizontal row of cards inside it. Most app home screens have both: a feed that scrolls down, with carousels that scroll sideways.

### Starter Code

- Replace everything in `App.js`.

```jsx
import { ScrollView, View, Text, StyleSheet } from "react-native";

const rows = Array.from({ length: 30 }, (_, i) => `Item ${i + 1}`);

export default function MyApp() {
  return (
    <ScrollView style={styles.container}>
      {rows.map((row) => (
        <View key={row} style={styles.row}>
          <Text style={styles.text}>{row}</Text>
        </View>
      ))}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, marginTop: 60 },
  row: { padding: 20, borderBottomWidth: 1, borderColor: "#ccc" },
  text: { fontSize: 18 },
});
```

- `Array.from({ length: 30 }, ...)` builds 30 strings without typing them. The string doubles as the `key`.
- On a phone, flick the list and watch it coast. Drag slowly and it follows your finger exactly. On the web preview, use the mouse wheel or the trackpad.

### Step 1 - Add a Horizontal Row

- Add the cards above `rows`.

```jsx
const cards = Array.from({ length: 10 }, (_, i) => `Card ${i + 1}`);
```

- Add a heading and a horizontal `ScrollView` inside the outer one, above the rows.

```jsx
<Text style={styles.heading}>Featured</Text>
<ScrollView
  horizontal
  showsHorizontalScrollIndicator={false}
  contentContainerStyle={styles.cards}
>
  {cards.map((card) => (
    <View key={card} style={styles.card}>
      <Text style={styles.text}>{card}</Text>
    </View>
  ))}
</ScrollView>
```

- Add the styles.

```jsx
heading: { fontSize: 22, fontWeight: "600", padding: 20 },
cards: { paddingHorizontal: 20, paddingBottom: 20 },
card: {
  width: 150,
  height: 150,
  marginRight: 15,
  borderRadius: 12,
  backgroundColor: "#dde",
  justifyContent: "center",
  alignItems: "center",
},
```

- `horizontal` is the only change that makes it scroll sideways. It is the same gesture, rotated.
- `contentContainerStyle` styles the content inside the scroll area, so the padding scrolls with the cards. `style` would pad the window around them.
- On a phone, drag sideways on the cards: they scroll and the page stays put. Drag up on the cards: the page scrolls. React Native gives the gesture to whichever scroll view matches the direction of the drag. On the web preview, scroll sideways with Shift and the mouse wheel, or two fingers on a trackpad.
- Nesting is safe when the directions differ. A scroll view inside one that scrolls the same way fights it for every drag. A `FlatList` inside a vertical `ScrollView` even warns about it in the console, because the `FlatList` can no longer tell what is on screen. Keep nesting shallow, and use a `FlatList` for the main list.

### Finished Code

- Compare against this if something is not working.

```jsx
import { ScrollView, View, Text, StyleSheet } from "react-native";

const cards = Array.from({ length: 10 }, (_, i) => `Card ${i + 1}`);
const rows = Array.from({ length: 30 }, (_, i) => `Item ${i + 1}`);

export default function MyApp() {
  return (
    <ScrollView style={styles.container}>
      <Text style={styles.heading}>Featured</Text>
      <ScrollView
        horizontal
        showsHorizontalScrollIndicator={false}
        contentContainerStyle={styles.cards}
      >
        {cards.map((card) => (
          <View key={card} style={styles.card}>
            <Text style={styles.text}>{card}</Text>
          </View>
        ))}
      </ScrollView>
      {rows.map((row) => (
        <View key={row} style={styles.row}>
          <Text style={styles.text}>{row}</Text>
        </View>
      ))}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, marginTop: 60 },
  heading: { fontSize: 22, fontWeight: "600", padding: 20 },
  cards: { paddingHorizontal: 20, paddingBottom: 20 },
  card: {
    width: 150,
    height: 150,
    marginRight: 15,
    borderRadius: 12,
    backgroundColor: "#dde",
    justifyContent: "center",
    alignItems: "center",
  },
  row: { padding: 20, borderBottomWidth: 1, borderColor: "#ccc" },
  text: { fontSize: 18 },
});
```

## Example: Pressable Rows in a FlatList

- A list has to scroll and its rows have to be tappable. Both start with a finger on a row, so the app must decide which one the user meant.
- React Native decides for you. A finger that lifts without moving is a tap. A finger that moves is a scroll, and the press on the row is cancelled.

### Starter Code

- Replace everything in `App.js`. It is the Week 5 `FlatList` with 40 plain rows.

```jsx
import { View, Text, FlatList, StyleSheet } from "react-native";

const rows = Array.from({ length: 40 }, (_, i) => ({
  id: String(i + 1),
  title: `Row ${i + 1}`,
}));

export default function MyApp() {
  return (
    <View style={styles.container}>
      <FlatList
        data={rows}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.row}>
            <Text style={styles.text}>{item.title}</Text>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  row: { padding: 20, borderBottomWidth: 1, borderColor: "#ccc" },
  text: { fontSize: 18 },
});
```

### Step 1 - Make the Rows Pressable

- Add `Pressable` to the `react-native` import.
- Change each row's `View` to a `Pressable` with pressed feedback.

```jsx
renderItem={({ item }) => (
  <Pressable
    style={({ pressed }) => [styles.row, pressed && styles.pressed]}
  >
    <Text style={styles.text}>{item.title}</Text>
  </Pressable>
)}
```

- Add the style.

```jsx
pressed: { backgroundColor: "#eee" },
```

- Tap a row and it greys while your finger is down.
- On a phone, put your finger on a row and start scrolling. The row may grey for an instant, then the list takes over and the row goes back to white. That is the press being cancelled because the finger moved.

### Step 2 - Select Rows

- Add `useState` to the imports. Store the selected rows as an object of ids, so `{ "3": true, "7": true }` means rows 3 and 7 are selected.

```jsx
const [selected, setSelected] = useState({});

function toggle(id) {
  setSelected((prev) => ({ ...prev, [id]: !prev[id] }));
}

const count = Object.values(selected).filter(Boolean).length;
```

- `toggle` copies the previous object and flips one id. The new state depends on the old, so it uses the updater form.
- `count` is derived on every render, like `visible` in Week 5. It is not state.
- Add the count above the `FlatList`.

```jsx
<Text style={styles.heading}>{count} selected</Text>
```

- Give each row an `onPress` and a selected style. The pressed style comes last so it still shows on a selected row.

```jsx
<Pressable
  style={({ pressed }) => [
    styles.row,
    selected[item.id] && styles.selected,
    pressed && styles.pressed,
  ]}
  onPress={() => toggle(item.id)}
>
```

- Add `extraData` to the `FlatList`.

```jsx
extraData={selected}
```

- `FlatList` skips re-rendering its rows when its props look unchanged, and `data` does not change when a row is selected. `extraData` tells it the rows also depend on `selected`. This list would happen to work without it, because the inline `renderItem` is a new function on every render, but a memoized `renderItem` would leave a tapped row looking unselected. Pass `extraData` whenever rows read state from outside `data`.
- Add the styles.

```jsx
heading: { fontSize: 18, fontWeight: "600", padding: 20 },
selected: { backgroundColor: "#BBDEFB" },
```

- Tap a few rows: they turn blue and the count follows. Tap one again to clear it. On a phone, a scroll that starts on a row never selects it.

### Finished Code

- Compare against this if something is not working.

```jsx
import { useState } from "react";
import { View, Text, Pressable, FlatList, StyleSheet } from "react-native";

const rows = Array.from({ length: 40 }, (_, i) => ({
  id: String(i + 1),
  title: `Row ${i + 1}`,
}));

export default function MyApp() {
  const [selected, setSelected] = useState({});

  function toggle(id) {
    setSelected((prev) => ({ ...prev, [id]: !prev[id] }));
  }

  const count = Object.values(selected).filter(Boolean).length;

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>{count} selected</Text>
      <FlatList
        data={rows}
        keyExtractor={(item) => item.id}
        extraData={selected}
        renderItem={({ item }) => (
          <Pressable
            style={({ pressed }) => [
              styles.row,
              selected[item.id] && styles.selected,
              pressed && styles.pressed,
            ]}
            onPress={() => toggle(item.id)}
          >
            <Text style={styles.text}>{item.title}</Text>
          </Pressable>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  heading: { fontSize: 18, fontWeight: "600", padding: 20 },
  row: { padding: 20, borderBottomWidth: 1, borderColor: "#ccc" },
  selected: { backgroundColor: "#BBDEFB" },
  pressed: { backgroundColor: "#eee" },
  text: { fontSize: 18 },
});
```

## Swipe and Cancel

- A swipe is a horizontal drag. Apps use it for quick actions on a row: delete, archive, reveal more options. Mail, Messages and most to-do apps do this.
- A swipe must be cancellable. The user may start one and change their mind, so a short swipe snaps back and does nothing. Only a deliberate swipe acts.
- A horizontal swipe must not block vertical scrolling. The row should only claim the gesture once the finger is clearly moving sideways.
- The swipe examples need a touch screen. Run them on your phone with **My Device**.

## Example: Drag and Snap Back

- Build a swipe by hand first, with two tools that come with React Native. `PanResponder` reports where the finger is. `Animated` moves a view without re-rendering the component on every frame.

### Starter Code

- Replace everything in `App.js`. It is a card that does not move.

```jsx
import { View, Text, StyleSheet } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.container}>
      <View style={styles.card}>
        <Text style={styles.text}>Swipe Me</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 100, paddingHorizontal: 20 },
  card: { padding: 20, borderRadius: 8, backgroundColor: "#eef" },
  text: { fontSize: 18 },
});
```

### Step 1 - Follow the Finger

- Add `useRef`, `Animated` and `PanResponder` to the imports.

```jsx
import { useRef } from "react";
import { View, Text, Animated, PanResponder, StyleSheet } from "react-native";
```

- At the top of `MyApp`, create an animated value for the card's horizontal offset.

```jsx
const translateX = useRef(new Animated.Value(0)).current;
```

- `useRef` is a hook that keeps the same object for the whole life of the component. Writing `new Animated.Value(0)` straight in the component would make a new value, back at 0, on every render. `useRef` creates it once, and `.current` is the value it holds. Changing it does not cause a render.
- Under it, create the pan responder, also kept in a ref.

```jsx
const pan = useRef(
  PanResponder.create({
    onMoveShouldSetPanResponder: (_, gesture) => Math.abs(gesture.dx) > 10,
    onPanResponderMove: (_, gesture) => {
      translateX.setValue(gesture.dx);
    },
  })
).current;
```

- `gesture.dx` is how far the finger has moved sideways since it went down.
- `onMoveShouldSetPanResponder` decides whether this view takes the gesture. It only says yes after 10 points of sideways movement, so a tap or a vertical drag is left for anything else on the screen, like a scroll view. That is the gesture priority rule in one line.
- `onPanResponderMove` runs for every movement and sets the offset. No state, no re-render.
- Change the card `View` to an `Animated.View`, spread the handlers onto it, and move it with a transform.

```jsx
<Animated.View
  {...pan.panHandlers}
  style={[styles.card, { transform: [{ translateX }] }]}
>
  <Text style={styles.text}>Swipe Me</Text>
</Animated.View>
```

- Drag the card. It follows your finger. Let go, and it stays wherever you left it.

### Step 2 - Snap Back on Release

- Add `onPanResponderRelease` to the `PanResponder.create` object.

```jsx
onPanResponderRelease: () => {
  Animated.spring(translateX, {
    toValue: 0,
    useNativeDriver: true,
  }).start();
},
```

- `Animated.spring` animates the value back to 0 with a little bounce. `useNativeDriver: true` runs the animation on the native side, so it stays smooth even when JavaScript is busy.
- Drag and let go. The card springs home. Every swipe is cancelled, which is the safe default, but now nothing ever happens.

### Step 3 - Commit Past a Threshold

- A short swipe should cancel and a long one should act. Add `useState` and `Button` to the imports, and a `dismissed` state at the top of `MyApp`.

```jsx
const [dismissed, setDismissed] = useState(false);
```

- Replace `onPanResponderRelease`. Past 120 points the card slides off the screen, and only then is it dismissed.

```jsx
onPanResponderRelease: (_, gesture) => {
  if (Math.abs(gesture.dx) > 120) {
    Animated.timing(translateX, {
      toValue: gesture.dx > 0 ? 500 : -500,
      duration: 200,
      useNativeDriver: true,
    }).start(() => setDismissed(true));
  } else {
    Animated.spring(translateX, {
      toValue: 0,
      useNativeDriver: true,
    }).start();
  }
},
```

- `.start()` takes a callback that runs when the animation ends. The state changes after the card has left, not before.
- Add a reset function above the `return`, and show a button instead of the card once it is dismissed.

```jsx
function reset() {
  translateX.setValue(0);
  setDismissed(false);
}
```

```jsx
{dismissed ? (
  <Button title="Bring It Back" onPress={reset} />
) : (
  <Animated.View
    {...pan.panHandlers}
    style={[styles.card, { transform: [{ translateX }] }]}
  >
    <Text style={styles.text}>Swipe Me</Text>
  </Animated.View>
)}
```

- Swipe a little and let go: it snaps back. Swipe most of the way: it flies off. That is swipe and cancel. Libraries add velocity, so a fast flick counts even when it is short, but the idea is the same.

### Finished Code

- Compare against this if something is not working.

```jsx
import { useRef, useState } from "react";
import {
  View,
  Text,
  Button,
  Animated,
  PanResponder,
  StyleSheet,
} from "react-native";

export default function MyApp() {
  const [dismissed, setDismissed] = useState(false);
  const translateX = useRef(new Animated.Value(0)).current;

  const pan = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: (_, gesture) => Math.abs(gesture.dx) > 10,
      onPanResponderMove: (_, gesture) => {
        translateX.setValue(gesture.dx);
      },
      onPanResponderRelease: (_, gesture) => {
        if (Math.abs(gesture.dx) > 120) {
          Animated.timing(translateX, {
            toValue: gesture.dx > 0 ? 500 : -500,
            duration: 200,
            useNativeDriver: true,
          }).start(() => setDismissed(true));
        } else {
          Animated.spring(translateX, {
            toValue: 0,
            useNativeDriver: true,
          }).start();
        }
      },
    })
  ).current;

  function reset() {
    translateX.setValue(0);
    setDismissed(false);
  }

  return (
    <View style={styles.container}>
      {dismissed ? (
        <Button title="Bring It Back" onPress={reset} />
      ) : (
        <Animated.View
          {...pan.panHandlers}
          style={[styles.card, { transform: [{ translateX }] }]}
        >
          <Text style={styles.text}>Swipe Me</Text>
        </Animated.View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 100, paddingHorizontal: 20 },
  card: { padding: 20, borderRadius: 8, backgroundColor: "#eef" },
  text: { fontSize: 18 },
});
```

## Example: Swipe to Delete

- Real apps use a library for swipeable rows instead of hand-writing `PanResponder` for every list. React Native Gesture Handler is the standard one, and Expo includes it.
- Its `Swipeable` component does the work of the previous example: it follows the finger, snaps back on a short swipe, and stays open on a long one, showing actions underneath the row.
- Replace `package.json` with the one at the top of these notes.

### Starter Code

- Replace everything in `App.js`. It is an inbox in a `FlatList`, with each row in its own component.

```jsx
import { useState } from "react";
import { View, Text, FlatList, StyleSheet } from "react-native";

const inbox = [
  { id: "1", from: "Registrar", subject: "Your timetable is ready" },
  { id: "2", from: "Library", subject: "Book due Friday" },
  { id: "3", from: "Campus Store", subject: "Textbook back in stock" },
  { id: "4", from: "IT Services", subject: "Password expires in 7 days" },
  { id: "5", from: "Study Group", subject: "Room 2104 at 3 PM" },
  { id: "6", from: "Athletics", subject: "Intramural sign-ups open" },
  { id: "7", from: "Career Centre", subject: "Resume workshop Thursday" },
  { id: "8", from: "Student Union", subject: "Free pancakes Monday" },
  { id: "9", from: "Professor", subject: "Lab 6 posted" },
  { id: "10", from: "Transit", subject: "Bus pass renewal" },
  { id: "11", from: "Food Services", subject: "New menu this week" },
  { id: "12", from: "Library", subject: "Study rooms now bookable" },
];

function MessageRow({ item }) {
  return (
    <View style={styles.row}>
      <Text style={styles.from}>{item.from}</Text>
      <Text style={styles.subject}>{item.subject}</Text>
    </View>
  );
}

export default function MyApp() {
  const [messages, setMessages] = useState(inbox);

  return (
    <View style={styles.container}>
      <FlatList
        data={messages}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => <MessageRow item={item} />}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  row: {
    padding: 20,
    borderBottomWidth: 1,
    borderColor: "#ddd",
    backgroundColor: "white",
  },
  from: { fontSize: 16, fontWeight: "600" },
  subject: { fontSize: 14, color: "#666" },
});
```

- `MessageRow` is declared outside `MyApp`, at the top level, like every component so far. A component declared inside another one is a new component on every render, so React throws the old rows away and builds new ones. Each row would lose its swipe position every time the list changed.
- The row has a white background on purpose. The delete button sits underneath it, and a transparent row would show it before any swipe.

### Step 1 - Add the Gesture Root

- Add the Gesture Handler import under the `react-native` import.

```jsx
import {
  GestureHandlerRootView,
  Swipeable,
} from "react-native-gesture-handler";
```

- Change the outer `View` in `MyApp` to a `GestureHandlerRootView`, keeping its style.

```jsx
<GestureHandlerRootView style={styles.container}>
  ...
</GestureHandlerRootView>
```

- Gesture Handler only works inside this root view. Put it once, around the whole app.
- Run it. Nothing looks different.

### Step 2 - Make the Rows Swipeable

- Wrap the row in `MessageRow` in a `Swipeable`, with a delete panel on the right.

```jsx
function MessageRow({ item }) {
  return (
    <Swipeable
      overshootRight={false}
      renderRightActions={() => (
        <Pressable style={styles.delete}>
          <Text style={styles.deleteText}>Delete</Text>
        </Pressable>
      )}
    >
      <View style={styles.row}>
        <Text style={styles.from}>{item.from}</Text>
        <Text style={styles.subject}>{item.subject}</Text>
      </View>
    </Swipeable>
  );
}
```

- Add `Pressable` to the `react-native` import, and the styles.

```jsx
delete: {
  justifyContent: "center",
  paddingHorizontal: 24,
  backgroundColor: "#E53935",
},
deleteText: { color: "white", fontWeight: "600" },
```

- `renderRightActions` returns what sits under the right edge of the row. Swiping left reveals it.
- `overshootRight={false}` stops the row at the edge of the panel instead of letting it stretch past.
- On your phone, swipe a row left a little and let go: it snaps back. That is the cancel. Swipe past about half the button's width: it stays open. Swipe it back to close it.
- Now drag up and down on the rows: the list scrolls and no row moves. A row only takes the gesture when the finger moves sideways.

### Step 3 - Delete the Message

- Add a delete function to `MyApp`, under the state.

```jsx
function deleteMessage(id) {
  setMessages((prev) => prev.filter((item) => item.id !== id));
}
```

- Pass it to each row.

```jsx
renderItem={({ item }) => (
  <MessageRow item={item} onDelete={deleteMessage} />
)}
```

- In `MessageRow`, accept the prop and call it from the button.

```jsx
function MessageRow({ item, onDelete }) {
```

```jsx
<Pressable style={styles.delete} onPress={() => onDelete(item.id)}>
```

- Add an empty state to the `FlatList`, as in Week 5, with its style.

```jsx
ListEmptyComponent={<Text style={styles.empty}>No messages.</Text>}
```

```jsx
empty: { padding: 20, color: "#666" },
```

- Swipe a row open and tap **Delete**. It is gone. Delete them all to see the empty state.
- The swipe only reveals the button; the tap deletes. Deleting is destructive, so it takes two deliberate actions. `Swipeable` also has `onSwipeableOpen`, which could delete as soon as the row opens, but then a slightly long swipe would lose a message with no way back. Save that for actions that can be undone, like archive.
- `Swipeable` is marked deprecated in favour of `ReanimatedSwipeable`, which needs the Reanimated library. It still works, the replacement's API is very similar, and it needs no extra setup, so it is the simpler one to learn with. The course notes say more.

### Finished Code

- Compare against this if something is not working.

```jsx
import { useState } from "react";
import { View, Text, Pressable, FlatList, StyleSheet } from "react-native";
import {
  GestureHandlerRootView,
  Swipeable,
} from "react-native-gesture-handler";

const inbox = [
  { id: "1", from: "Registrar", subject: "Your timetable is ready" },
  { id: "2", from: "Library", subject: "Book due Friday" },
  { id: "3", from: "Campus Store", subject: "Textbook back in stock" },
  { id: "4", from: "IT Services", subject: "Password expires in 7 days" },
  { id: "5", from: "Study Group", subject: "Room 2104 at 3 PM" },
  { id: "6", from: "Athletics", subject: "Intramural sign-ups open" },
  { id: "7", from: "Career Centre", subject: "Resume workshop Thursday" },
  { id: "8", from: "Student Union", subject: "Free pancakes Monday" },
  { id: "9", from: "Professor", subject: "Lab 6 posted" },
  { id: "10", from: "Transit", subject: "Bus pass renewal" },
  { id: "11", from: "Food Services", subject: "New menu this week" },
  { id: "12", from: "Library", subject: "Study rooms now bookable" },
];

function MessageRow({ item, onDelete }) {
  return (
    <Swipeable
      overshootRight={false}
      renderRightActions={() => (
        <Pressable style={styles.delete} onPress={() => onDelete(item.id)}>
          <Text style={styles.deleteText}>Delete</Text>
        </Pressable>
      )}
    >
      <View style={styles.row}>
        <Text style={styles.from}>{item.from}</Text>
        <Text style={styles.subject}>{item.subject}</Text>
      </View>
    </Swipeable>
  );
}

export default function MyApp() {
  const [messages, setMessages] = useState(inbox);

  function deleteMessage(id) {
    setMessages((prev) => prev.filter((item) => item.id !== id));
  }

  return (
    <GestureHandlerRootView style={styles.container}>
      <FlatList
        data={messages}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <MessageRow item={item} onDelete={deleteMessage} />
        )}
        ListEmptyComponent={<Text style={styles.empty}>No messages.</Text>}
      />
    </GestureHandlerRootView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  row: {
    padding: 20,
    borderBottomWidth: 1,
    borderColor: "#ddd",
    backgroundColor: "white",
  },
  from: { fontSize: 16, fontWeight: "600" },
  subject: { fontSize: 14, color: "#666" },
  delete: {
    justifyContent: "center",
    paddingHorizontal: 24,
    backgroundColor: "#E53935",
  },
  deleteText: { color: "white", fontWeight: "600" },
  empty: { padding: 20, color: "#666" },
});
```

## Touch UX Checklist

- Make touch targets at least 44 points, and use `hitSlop` when the visual has to be smaller.
- Give feedback the moment the finger lands: the `pressed` style.
- Keep gestures simple. A swipe is a shortcut, not the only way to do something important.
- Make gestures forgiving: a short swipe cancels, and a scroll that starts on a row does not tap it.
- Keep the UI fast. Gestures run every frame, so never block the JavaScript thread while one is in progress.
- Test on a real device. The web preview has a mouse, and users have thumbs.
