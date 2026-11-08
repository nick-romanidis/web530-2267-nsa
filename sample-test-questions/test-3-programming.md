# WEB530 Test 3 – Practice Programming Questions

These are practice questions for Test 3. The test has **two programming questions**, and there is one practice question for each. They practise the **same skills at about the same difficulty**:

- **Question 1** loads a list from the internet: `fetch` in `useEffect`, a loading spinner, an error message, and a `FlatList` with keys and an extra part of the list.
- **Question 2** makes list rows respond to touch: `Pressable` with pressed feedback, tapping to select rows, a count worked out from state, `extraData`, and a long press.

The test questions ask for different things than these, in different apps, so don't memorise these answers. If you can explain every line of your answers here, you're ready.

**How the test works.** Each test question comes with its own starter Snack, already set up with the right files. The instructions are in a comment at the top of `App.js`, the same as in the starters below, so you can work without switching tabs. When you finish, you copy `App.js` into the answer box. The **Web** preview is enough for both questions. You do not need a phone or the iOS or Android emulator.

Total: 13 marks. Suggested time: about 25 minutes for both.

## Before you start

- Do each question in its own Snack. Open the React Native shell Snack at https://snack.expo.dev/@nick-romanidis/react-native-shell, and keep it on SDK 54, the default.
- Replace everything in `package.json` with the `package.json` below.
- Add a new file named `styles.js` next to `App.js`, and paste in that question's `styles.js`.
- Replace everything in `App.js` with that question's `App.js`.
- Use the **Web** preview.
- Use the course notes at https://seneca-ictoer.github.io/WEB530/. The Lists and Touch sections have everything you need.
- On the test you will not be able to log in to Expo or save the Snack, so practise working in an unsaved Snack and copying `App.js` somewhere safe every few minutes.

_package.json_ (the same for both questions)

```json
{
  "dependencies": {
    "react-native-paper": "4.9.2",
    "@expo/vector-icons": "^15.0.3"
  }
}
```

---

## Question 1 – Post Comments (7 marks, about 13 minutes)

The app shows the comments on one blog post, loaded from the internet. Everything you need to do is in the comment at the top of `App.js`.

### Starter

_styles.js_

```js
import { StyleSheet } from 'react-native';

export default StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  center: { flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20, gap: 10 },
  heading: { fontSize: 24, fontWeight: '600', paddingHorizontal: 20, paddingBottom: 10 },
  row: { padding: 20, borderBottomWidth: 1, borderColor: '#ddd' },
  title: { fontSize: 16, fontWeight: '600' },
  subtitle: { fontSize: 14, color: '#666' },
  empty: { fontSize: 16, color: '#666', textAlign: 'center', padding: 40 },
  error: { fontSize: 16, color: 'red', textAlign: 'center' },
});
```

_App.js_

```jsx
/*
  WEB530 Test 3 Practice - Question 1: Post Comments (7 marks)

  The app shows the comments on one blog post, loaded from the
  internet. The address is in URL. Only the comments state is
  declared: add any other state you need. The Web preview is enough.
  No styling is needed: every style you need is in styles.js.

  Load the comments (2 marks)
  - When the app opens, fetch the comments from URL and store them in
    comments. It should fetch once, not on every render.

  Loading and errors (2 marks)
  - While the comments load, show a spinner with the text
    "Loading comments..." under it, in the middle of the screen
    (styles.center), instead of the list.
  - If the server answers with an error, show an error message
    (styles.error) in the middle of the screen instead of the list.
    To test it, change "comments" in the URL to "commentsx". The
    server answers 404, and fetch does not treat that as a failure
    on its own.

  Show the comments (3 marks)
  - Show the comments in a FlatList under the heading. Each row shows
    the comment's name (styles.title) with the email under it
    (styles.subtitle), inside a View with styles.row.
  - Every row needs a key. Use the comment's id.
  - If the post has no comments, the list says "No comments yet."
    (styles.empty). It must be part of the list, not a separate
    screen. To test it, change postId=1 in the URL to postId=999.

  Copy this file somewhere safe every few minutes, and again when you
  finish. You may leave this comment in.
*/

import { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator } from 'react-native';
import styles from './styles';

const URL = 'https://jsonplaceholder.typicode.com/comments?postId=1';

export default function MyApp() {
  const [comments, setComments] = useState([]);

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>Comments on Post 1</Text>
    </View>
  );
}
```

### Checking your work

- When the app opens, a spinner with `Loading comments...` shows for a moment, then the list appears under **Comments on Post 1**.
- There are 5 rows. The first is `id labore ex et quam laborum`, with `Eliseo@gardner.biz` under it.
- Change `comments` in the URL to `commentsx`. Your error message shows in red in the middle of the screen. If you see an empty list or a spinner that never stops instead, look again at what `fetch` does with a 404. Change it back.
- Change `postId=1` to `postId=999`. The heading is still there, and under it the list says `No comments yet.` Change it back.
- Add a `console.log` inside your effect. It prints once when the app opens, not every time the screen re-renders. Remove it when you're done.
- Check the browser console. There should be no red errors and no warning about a missing `key`.

---

## Question 2 – Camping Packing List (6 marks, about 12 minutes)

A checklist for a camping trip. The user taps items as they pack them, and holds a button to start over. Everything you need to do is in the comment at the top of `App.js`.

### Starter

_styles.js_

```js
import { StyleSheet } from 'react-native';

export default StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  heading: { fontSize: 20, fontWeight: '600', padding: 20 },
  row: { padding: 16, borderBottomWidth: 1, borderColor: '#ccc' },
  packed: { backgroundColor: '#DCEDC8' },
  pressed: { backgroundColor: '#eee' },
  name: { fontSize: 18 },
  button: {
    margin: 20,
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    backgroundColor: '#E53935',
  },
  buttonText: { color: 'white', fontSize: 16 },
  hint: { fontSize: 14, color: '#666', textAlign: 'center', paddingBottom: 20 },
});
```

_App.js_

```jsx
/*
  WEB530 Test 3 Practice - Question 2: Camping Packing List (6 marks)

  A checklist for a camping trip. The user taps items as they pack
  them, and holds a button to start over. The hint state is declared.
  Add the state you need to remember which items are packed. No
  styling is needed: every style you need is in styles.js.

  The Web preview is enough. A mouse click is a tap, and holding the
  mouse button down for about half a second is a long press.

  Packing items (4 marks)
  - Each row can be pressed, and uses styles.pressed while it is
    held down.
  - Tapping an item marks it packed. Tapping it again marks it not
    packed. Tip: an object of ids, such as { i2: true }, works well.
  - Packed items use styles.packed.
  - Make sure the FlatList re-renders its rows when the packed items
    change.

  Progress (1 mark)
  - The heading shows progress, for example "3 of 8 packed", instead
    of always "0 of 8 packed". It must still be right after an item
    is unpacked again.

  Start over (1 mark)
  - Starting over unpacks everything, so it must not happen by
    accident. Make the "Unpack all" button work only on a long press.
    A normal tap shows the hint "Hold the button to unpack everything."
    instead.

  Copy this file somewhere safe every few minutes, and again when you
  finish. You may leave this comment in.
*/

import { useState } from 'react';
import { View, Text, FlatList } from 'react-native';
import styles from './styles';

const ITEMS = [
  { id: 'i1', name: 'Tent' },
  { id: 'i2', name: 'Sleeping bag' },
  { id: 'i3', name: 'Flashlight' },
  { id: 'i4', name: 'Water bottle' },
  { id: 'i5', name: 'Bug spray' },
  { id: 'i6', name: 'First aid kit' },
  { id: 'i7', name: 'Rain jacket' },
  { id: 'i8', name: 'Marshmallows' },
];

export default function MyApp() {
  const [hint, setHint] = useState('');

  return (
    <View style={styles.container}>
      <Text style={styles.heading}>0 of {ITEMS.length} packed</Text>
      <FlatList
        data={ITEMS}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.row}>
            <Text style={styles.name}>{item.name}</Text>
          </View>
        )}
      />
      <View style={styles.button}>
        <Text style={styles.buttonText}>Unpack all</Text>
      </View>
      <Text style={styles.hint}>{hint}</Text>
    </View>
  );
}
```

### Checking your work

- The heading says `0 of 8 packed`.
- Press and hold the mouse button on a row. It turns grey while held.
- Click **Tent**, **Flashlight** and **Rain jacket**. All three turn green, and the heading says `3 of 8 packed`.
- Click **Flashlight** again. It goes back to white, and the heading says `2 of 8 packed`.
- Click **Unpack all** once. The hint `Hold the button to unpack everything.` appears, and nothing is unpacked.
- Hold the mouse button down on **Unpack all** for about half a second. Every item goes back to white, and the heading says `0 of 8 packed`.
- Check that your `FlatList` has the prop that tells it the rows depend on your packed state. The app works without it, so you can't see that it's missing by testing.
- Check the browser console. There should be no red errors.
