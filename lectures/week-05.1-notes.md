# Week 5.1 - Notes and Examples

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.js_
(https://snack.expo.dev)

- Keep Snack on SDK 54 (the default). SDK 55 still fails to load `react-native-screens`, so the navigation examples do not run on it.
- The first three examples only need `react-native`. The List and Detail example adds React Navigation, so replace the contents of `package.json` before that one. It is the Week 4 file with the tab, drawer and Jotai packages removed.

_package.json_

```json
{
  "dependencies": {
    "@react-navigation/native": "^7.0.0",
    "@react-navigation/native-stack": "^7.0.0",
    "react-native-screens": "~4.16.0",
    "react-native-safe-area-context": "~5.6.0"
  }
}
```

- The fetch examples load `https://jsonplaceholder.typicode.com/posts`, a free fake API that returns 100 posts. Open it in a browser tab first so everyone sees the shape of the data: `userId`, `id`, `title`, `body`.

## Why Lists?

- Most mobile apps are list-driven: messages, contacts, products, posts, notifications.
- A `ScrollView` with `.map()` from Week 3 renders every child at once. Fine for a page of content, a problem for a thousand rows.
- `FlatList` is the list component. It only renders the rows that are on screen plus a small buffer (windowing), and it reuses row components as you scroll (recycling).
- Each item becomes a row. Keep rows consistent: a clear hierarchy (title, subtitle), touch-friendly spacing, and visual separation (a border or a card). A row that leads somewhere should look tappable.

## Example: FlatList

- Show a list of fruit. Replace everything in `App.js`.

```jsx
import { View, Text, FlatList, StyleSheet } from "react-native";

const fruits = [
  { id: "1", name: "Apple", color: "Red" },
  { id: "2", name: "Cherry", color: "Red" },
  { id: "3", name: "Banana", color: "Yellow" },
  { id: "4", name: "Grape", color: "Purple" },
  { id: "5", name: "Lemon", color: "Yellow" },
  { id: "6", name: "Kiwi", color: "Green" },
  { id: "7", name: "Orange", color: "Orange" },
  { id: "8", name: "Blueberry", color: "Blue" },
];

export default function MyApp() {
  return (
    <View style={styles.container}>
      <FlatList
        data={fruits}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.row}>
            <Text style={styles.title}>{item.name}</Text>
            <Text style={styles.subtitle}>{item.color}</Text>
          </View>
        )}
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
  },
  title: { fontSize: 18, fontWeight: "600" },
  subtitle: { fontSize: 14, color: "#666" },
});
```

- Three props do the work: `data` is the array, `renderItem` says what a row looks like, and `keyExtractor` says how to identify each row.
- There is no `.map()` and no `key` prop. `FlatList` calls `renderItem` once per visible row and handles the keys itself.
- `renderItem` receives an object, and the row's data is in its `item` property. That is why the parameter is written `({ item })`. The object also has `index`, which you rarely need.
- `keyExtractor` must return a string. Without it, `FlatList` looks for `item.key`, then `item.id`, so this list would work without one. Write it anyway: real data uses other names (`_id`, `postId`), and numbers need `.toString()`.
- The `container` needs `flex: 1`. A `FlatList` scrolls inside the space its parent gives it, and with no height it gets none.
- Show that only the visible rows render. Give `renderItem` a body with a `console.log` in it.

```jsx
renderItem={({ item }) => {
  console.log(item.name);
  return (
    <View style={styles.row}>
      <Text style={styles.title}>{item.name}</Text>
      <Text style={styles.subtitle}>{item.color}</Text>
    </View>
  );
}}
```

- Replace the array with a generated one and scroll. On load, only the first ten or so names appear in the console. More appear as you scroll, and only then. The list is still instant.

```jsx
const fruits = Array.from({ length: 1000 }, (_, i) => ({
  id: String(i + 1),
  name: `Fruit ${i + 1}`,
  color: "Green",
}));
```

- Look closer at the console: it is about 100 names, not 10. "Only what is on screen" is a simplification. `FlatList` renders a window around the visible area, `windowSize` screens tall, and the default is 21: ten screens above, ten below. At the top there is nothing above, so it fills ten screens below, and ten rows per screen is about 100 rows. The window is big on purpose, so fast scrolling never shows blank rows.
- Show the window shrink. Add this prop to the `FlatList` and reload.

```jsx
windowSize={3}
```

- Now the console shows the visible rows plus about a screen of spare, and the rest appear only as you scroll. Fling the list and blank cells flash by before they fill in. That is why the default is 21. Remove the prop; it is a demo setting, not something to ship.
- Do not go lower than 3. With `windowSize={1}` the window has no room to grow, and the list never renders past the first ten rows: it just re-renders them on every scroll.
- Before moving on, go back to the original example: paste the FlatList code from the top of this section into `App.js` again. That removes the 1000 rows, the `console.log` and `windowSize` in one go. The next example builds on that code.

## Example: Sorting and Filtering

- Users expect a predictable order, and a way to narrow a long list down.
- Neither one touches the UI. You change the array, and `FlatList` renders whatever it gets.
- Start from the finished FlatList code.

### Step 1 - Sort with a Button

- Add `useState` and `Button` to the imports.

```jsx
import { useState } from "react";
import { View, Text, Button, FlatList, StyleSheet } from "react-native";
```

- Inside `MyApp`, above the `return`, add state for the direction and build the sorted array from it.

```jsx
const [ascending, setAscending] = useState(true);

const visible = [...fruits].sort((a, b) =>
  ascending ? a.name.localeCompare(b.name) : b.name.localeCompare(a.name)
);
```

- Add a button above the `FlatList`.

```jsx
<Button
  title={ascending ? "Sort Z to A" : "Sort A to Z"}
  onPress={() => setAscending((prev) => !prev)}
/>
```

- Point the list at the sorted array.

```jsx
data={visible}
```

- Run it. Each press flips the order.
- `[...fruits]` copies the array first. `.sort()` sorts in place, and the original must stay as it is. Here it is a module-level constant; later it is state, and state is never mutated.
- `localeCompare` sorts strings the way people expect, with `"apple"` before `"Banana"`. For numbers, subtract: `a.price - b.price`.
- `visible` is not state. It is derived from state on every render, so there is nothing to keep in sync. The only state is the direction.
- The course notes wrap this in `useMemo` so it only recalculates when the inputs change. Same idea. Skip it until a list is big enough to notice.

### Step 2 - Filter with a TextInput

- Add `TextInput` to the `react-native` import.
- Add state for the search text, and filter before sorting. Replace the `visible` lines.

```jsx
const [search, setSearch] = useState("");

const visible = fruits
  .filter((item) => item.name.toLowerCase().includes(search.toLowerCase()))
  .sort((a, b) =>
    ascending ? a.name.localeCompare(b.name) : b.name.localeCompare(a.name)
  );
```

- `.filter()` returns a new array, so the `[...fruits]` copy is gone. `.sort()` sorts that new array, and `fruits` is still untouched.
- Filter first, then sort. Sorting a shorter list is less work, and the result is the same.
- Add a controlled `TextInput` between the button and the list, as in Week 3.

```jsx
<TextInput
  style={styles.input}
  placeholder="Search"
  value={search}
  onChangeText={setSearch}
/>
```

- Add its style.

```jsx
input: {
  borderWidth: 1,
  borderColor: "#ccc",
  padding: 8,
  marginHorizontal: 20,
  marginVertical: 10,
},
```

- Type `an`: Banana and Orange. Type `rr`: Cherry and Blueberry. Every keystroke re-renders, the array is rebuilt, and the list follows.
- Type `zz`. The list is empty, and nothing says so. A blank screen looks broken.

### Step 3 - Show an Empty State

- Add `ListEmptyComponent` to the `FlatList`. It renders only when `data` is empty.

```jsx
ListEmptyComponent={<Text style={styles.empty}>No fruit matches.</Text>}
```

- Add its style.

```jsx
empty: { padding: 20, color: "#666" },
```

- Empty is a state the user will see, so say something. The same prop handles a fetched list that comes back with nothing in it.

### Finished Code

- Compare against this if something is not working.

```jsx
import { useState } from "react";
import {
  View,
  Text,
  Button,
  TextInput,
  FlatList,
  StyleSheet,
} from "react-native";

const fruits = [
  { id: "1", name: "Apple", color: "Red" },
  { id: "2", name: "Cherry", color: "Red" },
  { id: "3", name: "Banana", color: "Yellow" },
  { id: "4", name: "Grape", color: "Purple" },
  { id: "5", name: "Lemon", color: "Yellow" },
  { id: "6", name: "Kiwi", color: "Green" },
  { id: "7", name: "Orange", color: "Orange" },
  { id: "8", name: "Blueberry", color: "Blue" },
];

export default function MyApp() {
  const [ascending, setAscending] = useState(true);
  const [search, setSearch] = useState("");

  const visible = fruits
    .filter((item) => item.name.toLowerCase().includes(search.toLowerCase()))
    .sort((a, b) =>
      ascending ? a.name.localeCompare(b.name) : b.name.localeCompare(a.name)
    );

  return (
    <View style={styles.container}>
      <Button
        title={ascending ? "Sort Z to A" : "Sort A to Z"}
        onPress={() => setAscending((prev) => !prev)}
      />
      <TextInput
        style={styles.input}
        placeholder="Search"
        value={search}
        onChangeText={setSearch}
      />
      <FlatList
        data={visible}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={styles.row}>
            <Text style={styles.title}>{item.name}</Text>
            <Text style={styles.subtitle}>{item.color}</Text>
          </View>
        )}
        ListEmptyComponent={<Text style={styles.empty}>No fruit matches.</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    padding: 8,
    marginHorizontal: 20,
    marginVertical: 10,
  },
  row: {
    padding: 20,
    borderBottomWidth: 1,
    borderColor: "#ddd",
  },
  title: { fontSize: 18, fontWeight: "600" },
  subtitle: { fontSize: 14, color: "#666" },
  empty: { padding: 20, color: "#666" },
});
```

## Example: Fetching List Data

- Real lists come from an API, and that changes the component. The data is not there when the screen first renders, and it might never arrive.
- Fetching brings three states with it: loading, error and empty. The lifecycle is: start loading, show a spinner, fetch, render the list, and if it failed, say so and offer a retry.

### Starter Code

- This is a new app, so replace everything in `App.js`. It fetches the posts and shows them, and nothing else.

```jsx
import { useState, useEffect } from "react";
import { View, Text, FlatList, StyleSheet } from "react-native";

const URL = "https://jsonplaceholder.typicode.com/posts";

export default function MyApp() {
  const [posts, setPosts] = useState([]);

  function loadPosts() {
    return fetch(URL)
      .then((res) => res.json())
      .then((json) => setPosts(json));
  }

  useEffect(() => {
    // Demo only: wait two seconds so the empty screen is visible.
    const id = setTimeout(() => {
      loadPosts();
    }, 2000);
    return () => clearTimeout(id);
  }, []);

  return (
    <View style={styles.container}>
      <FlatList
        data={posts}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.row}>
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.subtitle}>{item.body}</Text>
          </View>
        )}
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
  },
  title: { fontSize: 18, fontWeight: "600" },
  subtitle: { fontSize: 14, color: "#666" },
});
```

- `posts` starts as an empty array, so the first render has something to give the list.
- `useEffect` with `[]` runs once, after the first render, as in Week 2. That is where the request starts.
- The timer is only there for the demo. On the classroom network the posts arrive too fast to see anything, so the request waits two seconds. The clean-up cancels the timer if the component unmounts first, the same pattern as Week 2. The finished code at the end leaves the timer out.
- `fetch` returns a promise for the response, and `res.json()` returns a promise for the parsed body. `loadPosts` returns the chain so the next steps can wait on it.
- `item.id` is a number here, so `keyExtractor` converts it.
- Run it. For two seconds the screen is blank, then the list appears. Nothing tells the user that anything is happening, and if the request fails, the screen stays blank forever.

### Step 1 - Show a Spinner While Loading

- Add `ActivityIndicator` to the `react-native` import.
- Add a `loading` state under `posts`. It starts as `true` because the first load starts right away.

```jsx
const [loading, setLoading] = useState(true);
```

- Change the line inside the timer so it clears the flag when the request finishes.

```jsx
loadPosts().finally(() => setLoading(false));
```

- Above the main `return`, return a different screen while loading.

```jsx
if (loading) {
  return (
    <View style={styles.center}>
      <ActivityIndicator size="large" />
      <Text>Loading posts...</Text>
    </View>
  );
}
```

- Add its style.

```jsx
center: { flex: 1, justifyContent: "center", alignItems: "center" },
```

- `.finally()` runs after success or failure, so the spinner always goes away.
- An early `return` for each state keeps the main `return` simple. The list code never has to ask whether the data is ready.
- Run it. The two seconds are now a spinner and a message instead of a blank screen.

### Step 2 - Handle Errors

- Add `Button` to the `react-native` import.
- Add an `error` state. `null` means no error.

```jsx
const [error, setError] = useState(null);
```

- Replace `loadPosts`. It checks the response, stores the posts, and records a message if anything went wrong.

```jsx
function loadPosts() {
  return fetch(URL)
    .then((res) => {
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return res.json();
    })
    .then((json) => {
      setPosts(json);
      setError(null);
    })
    .catch(() =>
      setError("Could not load the posts. Check your connection and try again.")
    );
}
```

- `fetch` only rejects when the network fails. A 404 or a 500 is a successful response as far as `fetch` is concerned, so check `res.ok` and throw to reach the `catch`.
- The message is for the user. No status codes, say what happened and what to do.
- Add a retry function under the effect. It shows the spinner again and reloads.

```jsx
function retry() {
  setLoading(true);
  loadPosts().finally(() => setLoading(false));
}
```

- Add the error screen under the loading one.

```jsx
if (error) {
  return (
    <View style={styles.center}>
      <Text style={styles.error}>{error}</Text>
      <Button title="Try Again" onPress={retry} />
    </View>
  );
}
```

- Add its style.

```jsx
error: { color: "red", textAlign: "center", padding: 20 },
```

- Show it. Change `URL` to end in `/postz`. The API answers 404, `res.ok` is false, and the error screen appears. Press **Try Again** to watch the spinner come back and the error return. Fix the URL.
- The order of the early returns matters. `loading` is checked first, so pressing **Try Again** shows the spinner even though `error` is still set.

### Step 3 - Show an Empty State

- Add `ListEmptyComponent` to the `FlatList`, as in the previous example.

```jsx
ListEmptyComponent={<Text style={styles.empty}>No posts yet.</Text>}
```

- Add its style.

```jsx
empty: { padding: 20, color: "#666" },
```

- Show it. Change `URL` to end in `/posts?userId=99`. The API filters by user, nobody has that id, and it returns `[]`. The list renders with the message. Fix the URL.
- That is the whole lifecycle: a spinner, the list, an error with a retry, and a message when there is nothing to show. The next example builds on this code.

### Finished Code

- Compare against this if something is not working.

```jsx
import { useState, useEffect } from "react";
import {
  View,
  Text,
  Button,
  FlatList,
  ActivityIndicator,
  StyleSheet,
} from "react-native";

const URL = "https://jsonplaceholder.typicode.com/posts";

export default function MyApp() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  function loadPosts() {
    return fetch(URL)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((json) => {
        setPosts(json);
        setError(null);
      })
      .catch(() =>
        setError("Could not load the posts. Check your connection and try again.")
      );
  }

  useEffect(() => {
    loadPosts().finally(() => setLoading(false));
  }, []);

  function retry() {
    setLoading(true);
    loadPosts().finally(() => setLoading(false));
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
        <Text>Loading posts...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{error}</Text>
        <Button title="Try Again" onPress={retry} />
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={posts}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.row}>
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.subtitle}>{item.body}</Text>
          </View>
        )}
        ListEmptyComponent={<Text style={styles.empty}>No posts yet.</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  row: {
    padding: 20,
    borderBottomWidth: 1,
    borderColor: "#ddd",
  },
  title: { fontSize: 18, fontWeight: "600" },
  subtitle: { fontSize: 14, color: "#666" },
  error: { color: "red", textAlign: "center", padding: 20 },
  empty: { padding: 20, color: "#666" },
});
```

## Example: List and Detail

- Most mobile apps use this pattern. The list is the overview, and tapping a row opens the detail. Mail, Contacts and Instagram all work this way.
- It is progressive disclosure: show what the user needs right now, and reveal the rest when they ask for it. The list stays clean, and the detail has room.
- Under the hood it is the stack navigator from Week 4, with the row passed as a param.
- Start from the finished Fetching List Data code, and replace `package.json` with the one at the top of these notes.

### Step 1 - Put the List in a Stack

- Add the navigation imports under the `react-native` import.

```jsx
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
```

- Create the stack under the imports, outside every component.

```jsx
const Stack = createNativeStackNavigator();
```

- Rename `MyApp` to `PostsScreen`, remove its `export default`, and give it the `navigation` prop.

```jsx
function PostsScreen({ navigation }) {
```

- Add a new `MyApp` at the bottom, above the styles.

```jsx
export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Posts" component={PostsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

- Remove `paddingTop` from `container`. The header provides the space now.

```jsx
container: { flex: 1 },
```

- Run it. The same list, with a **Posts** header.

### Step 2 - Make Each Row Tappable

- Add `Pressable` to the `react-native` import.
- Replace the row. The `View` becomes a `Pressable` that navigates, and the body is cut to one line.

```jsx
renderItem={({ item }) => (
  <Pressable
    style={styles.row}
    onPress={() => navigation.navigate("Details", { post: item })}
  >
    <Text style={styles.title}>{item.title}</Text>
    <Text style={styles.subtitle} numberOfLines={1}>
      {item.body}
    </Text>
  </Pressable>
)}
```

- `Pressable` is the general touch wrapper. It has no look of its own, so the row style stays the same. There is more on touch later in the course.
- `numberOfLines={1}` trims the body to a preview with an ellipsis. The list is the summary; the full text belongs on the detail screen.
- The whole post goes along as a param, so the detail screen has nothing to fetch.
- Press a row. Nothing opens, and the console says the `NAVIGATE` action was not handled, because there is no `Details` screen yet.

### Step 3 - Add the Details Screen

- Add the screen under `PostsScreen`. It reads the post from `route.params`, as in Week 4.

```jsx
function DetailsScreen({ route }) {
  const { post } = route.params;

  return (
    <View style={styles.detail}>
      <Text style={styles.title}>{post.title}</Text>
      <Text style={styles.body}>{post.body}</Text>
    </View>
  );
}
```

- Register it under the **Posts** screen, with the post's title in the header.

```jsx
<Stack.Screen
  name="Details"
  component={DetailsScreen}
  options={({ route }) => ({ title: route.params.post.title })}
/>
```

- Add the styles.

```jsx
detail: { flex: 1, padding: 20 },
body: { fontSize: 16, lineHeight: 24, marginTop: 10 },
```

- Run it. Press a row, read the post, and go back with the header button. The list is exactly where you left it, because the `Posts` screen stayed on the stack.
- Params should be small and serializable. A post is fine. For big records, pass only the `id` and let the detail screen fetch the rest. The course notes have that version.

### Finished Code

- Compare against this if something is not working.

```jsx
import { useState, useEffect } from "react";
import {
  View,
  Text,
  Button,
  FlatList,
  Pressable,
  ActivityIndicator,
  StyleSheet,
} from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";

const URL = "https://jsonplaceholder.typicode.com/posts";

const Stack = createNativeStackNavigator();

function PostsScreen({ navigation }) {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  function loadPosts() {
    return fetch(URL)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((json) => {
        setPosts(json);
        setError(null);
      })
      .catch(() =>
        setError("Could not load the posts. Check your connection and try again.")
      );
  }

  useEffect(() => {
    loadPosts().finally(() => setLoading(false));
  }, []);

  function retry() {
    setLoading(true);
    loadPosts().finally(() => setLoading(false));
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
        <Text>Loading posts...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{error}</Text>
        <Button title="Try Again" onPress={retry} />
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={posts}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <Pressable
            style={styles.row}
            onPress={() => navigation.navigate("Details", { post: item })}
          >
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.subtitle} numberOfLines={1}>
              {item.body}
            </Text>
          </Pressable>
        )}
        ListEmptyComponent={<Text style={styles.empty}>No posts yet.</Text>}
      />
    </View>
  );
}

function DetailsScreen({ route }) {
  const { post } = route.params;

  return (
    <View style={styles.detail}>
      <Text style={styles.title}>{post.title}</Text>
      <Text style={styles.body}>{post.body}</Text>
    </View>
  );
}

export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Posts" component={PostsScreen} />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={({ route }) => ({ title: route.params.post.title })}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  row: {
    padding: 20,
    borderBottomWidth: 1,
    borderColor: "#ddd",
  },
  title: { fontSize: 18, fontWeight: "600" },
  subtitle: { fontSize: 14, color: "#666" },
  error: { color: "red", textAlign: "center", padding: 20 },
  empty: { padding: 20, color: "#666" },
  detail: { flex: 1, padding: 20 },
  body: { fontSize: 16, lineHeight: 24, marginTop: 10 },
});
```

## Example: Pull to Refresh

- Users expect to refresh a list by pulling down on it. Social feeds, email and dashboards all do it.
- `FlatList` has it built in through two props, `refreshing` and `onRefresh`. There is no gesture code to write. React Native handles the pull, you handle the data.
- Pulling needs a touch screen, so it does nothing in the web preview. Switch the preview to **iOS** or **Android**, or choose **My Device** and open it in Expo Go.
- Start from the finished List and Detail code. Every change is inside `PostsScreen`.

### Step 1 - Track the Refresh

- Add a `refreshing` state under `error`, and a function that reloads the posts, next to `retry`.

```jsx
const [refreshing, setRefreshing] = useState(false);
```

```jsx
function refreshPosts() {
  setRefreshing(true);
  loadPosts().finally(() => setRefreshing(false));
}
```

- `loading` and `refreshing` are separate on purpose. The first load replaces the whole screen with a spinner. A refresh keeps the list on screen and spins at the top.

### Step 2 - Wire Up the List

- Add the two props to the `FlatList`.

```jsx
refreshing={refreshing}
onRefresh={refreshPosts}
```

- Pull down. The indicator appears at the top, the posts reload, and the indicator goes away. The list keeps its position.

### Step 3 - Prove It Reloaded

- The API returns the same 100 posts every time, so nothing visibly changed. Record when the data arrived.
- Add one more state.

```jsx
const [updatedAt, setUpdatedAt] = useState(null);
```

- Set it where the posts are stored, in `loadPosts`.

```jsx
.then((json) => {
  setPosts(json);
  setUpdatedAt(new Date());
  setError(null);
})
```

- Show it above the rows with `ListHeaderComponent`. It scrolls with the list, unlike a `Text` placed above the `FlatList`.

```jsx
ListHeaderComponent={
  <Text style={styles.updated}>
    Updated {updatedAt?.toLocaleTimeString()}
  </Text>
}
```

- Add its style.

```jsx
updated: { padding: 10, textAlign: "center", color: "#666" },
```

- Pull again. The time changes.
- Keep pull to refresh for manual reloads, and `useEffect` for the first load. Keep it fast: if the API is slow, the user is left holding the list.

### Finished PostsScreen

- Only `PostsScreen` changed. Compare against this if something is not working.

```jsx
function PostsScreen({ navigation }) {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [refreshing, setRefreshing] = useState(false);
  const [updatedAt, setUpdatedAt] = useState(null);

  function loadPosts() {
    return fetch(URL)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((json) => {
        setPosts(json);
        setUpdatedAt(new Date());
        setError(null);
      })
      .catch(() =>
        setError("Could not load the posts. Check your connection and try again.")
      );
  }

  useEffect(() => {
    loadPosts().finally(() => setLoading(false));
  }, []);

  function retry() {
    setLoading(true);
    loadPosts().finally(() => setLoading(false));
  }

  function refreshPosts() {
    setRefreshing(true);
    loadPosts().finally(() => setRefreshing(false));
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
        <Text>Loading posts...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{error}</Text>
        <Button title="Try Again" onPress={retry} />
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={posts}
        keyExtractor={(item) => item.id.toString()}
        refreshing={refreshing}
        onRefresh={refreshPosts}
        renderItem={({ item }) => (
          <Pressable
            style={styles.row}
            onPress={() => navigation.navigate("Details", { post: item })}
          >
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.subtitle} numberOfLines={1}>
              {item.body}
            </Text>
          </Pressable>
        )}
        ListHeaderComponent={
          <Text style={styles.updated}>
            Updated {updatedAt?.toLocaleTimeString()}
          </Text>
        }
        ListEmptyComponent={<Text style={styles.empty}>No posts yet.</Text>}
      />
    </View>
  );
}
```

- Add the `updated` style to the `StyleSheet` at the bottom.

```jsx
updated: { padding: 10, textAlign: "center", color: "#666" },
```
