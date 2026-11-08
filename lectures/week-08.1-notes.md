# Week 8.1 - Notes and Examples

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.tsx_
(https://snack.expo.dev)

- Keep Snack on SDK 54 (the default).
- Every example this week only needs `react-native`. Leave `package.json` alone.
- This week's examples are TypeScript, so the file is `App.tsx`, not `App.js`. In the file list, hover over `App.js`, open its menu (the three dots on the left) and choose **Rename to App.tsx**. Do this once per Snack. The same menu renames it back.
- Type errors show as red squiggles in the editor, with a red mark in the scrollbar on the right. Hover over a squiggle to read the message. The "No errors" indicator at the bottom only counts runtime errors, so look for squiggles, not the indicator.
- Everything this week runs on **Web**.

## Why TypeScript?

- JavaScript finds out about a wrong type when the code runs: a missing prop renders as nothing, a string where a number was expected does string maths, and `undefined.name` crashes. TypeScript finds out while you type.
- TypeScript is JavaScript plus type annotations. Every `.js` file you have written this term is valid TypeScript. You add types where they help and let TypeScript work out the rest.
- Props become a contract. In Week 1 PropTypes warned in the console at runtime. A TypeScript interface does the same job in the editor, before the app runs, and React 19 dropped PropTypes anyway.
- The types never reach the phone. Metro strips them out and ships plain JavaScript, so a type error does not stop the app from running. That is why Snack runs an `App.tsx` with squiggles in it. The checker is a tool for you, not a gate for the device.
- Every new Expo project is TypeScript by default. The Expo projects later in the course are `.tsx` files, so this is the week to get comfortable.

## Example: Types

- The pieces from the slides, in a screen you can run.

```tsx
import { View, Text, StyleSheet } from "react-native";

let count: number = 0;
let name: string = "Nicholas";
let isActive: boolean = true;
let scores: number[] = [10, 20, 30];
let user: { name: string; age: number } = { name: "Ava", age: 21 };

export default function MyApp() {
  return (
    <View style={styles.container}>
      <Text style={styles.line}>count: {count}</Text>
      <Text style={styles.line}>name: {name}</Text>
      <Text style={styles.line}>isActive: {String(isActive)}</Text>
      <Text style={styles.line}>scores: {scores.join(", ")}</Text>
      <Text style={styles.line}>
        user: {user.name}, {user.age}
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, paddingHorizontal: 20 },
  line: { fontSize: 18, marginBottom: 8 },
});
```

### How It Works

- A type annotation is a colon and a type after the name: `count: number`. It says what the variable may hold, now and after every reassignment.
- `number[]` is an array of numbers. `{ name: string; age: number }` is the shape of an object: these properties, with these types, nothing missing.
- `String(isActive)` is a React Native thing, not a TypeScript thing: `Text` will not render a bare boolean.

### What to Show

1. Run it. Five lines of output, nothing special. TypeScript only shows up when something is wrong.
2. Change `let count: number = 0;` to `let count: number = "zero";`. A squiggle appears under `count`. Hover: `Type 'string' is not assignable to type 'number'`. Now look at the preview: the app still runs and prints `count: zero`. The checker and the runtime are separate.
3. Under the variables add `scores.push("forty");`. Same message. The array type applies to everything you put in later, not only the initial values.
4. Delete `age: 21` from `user`. Hover: `Property 'age' is missing`. An object type is a checklist.
5. Sometimes a property really is optional. Add a question mark to it in the type: `{ name: string; age?: number }`. The squiggle goes away. `age?` means the property may be left out, and inside the code its type is `number | undefined`. Run it: the line shows `user: Ava,` with nothing after the comma, because `user.age` is `undefined` and `Text` renders nothing for it. That is the trade: optional means every use has to cope with it missing.
6. Remove the annotation from `count` so it reads `let count = 0;`, then hover over `count`. The tooltip says `let count: number`. TypeScript inferred it from the `0`. Change the value to `"zero"` again: still an error. You get checking without writing a type, which is how most lines of TypeScript look.

## Example: Typed Props

- A component with props is the first place types earn their keep. The interface is a contract between the component and everyone who uses it, and TypeScript checks both sides.

### Starter Code

- Replace everything in `App.tsx`. Three stat cards, written the way we have written components all term.

```tsx
import { View, Text, StyleSheet } from "react-native";

function Stat({ label, value, unit }) {
  return (
    <View style={styles.card}>
      <Text style={styles.value}>
        {value} {unit}
      </Text>
      <Text style={styles.label}>{label}</Text>
    </View>
  );
}

export default function MyApp() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Today</Text>
      <Stat label="Steps" value={8412} />
      <Stat label="Distance" value={6.2} unit="km" />
      <Stat label="Calories" value={320} unit="kcal" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, paddingHorizontal: 20 },
  title: { fontSize: 28, fontWeight: "700", marginBottom: 16 },
  card: {
    padding: 16,
    marginBottom: 12,
    borderRadius: 8,
    backgroundColor: "#E3F2FD",
  },
  value: { fontSize: 24, fontWeight: "600" },
  label: { fontSize: 14, color: "#555" },
});
```

- Run it. It works, and the editor already has something to say: squiggles under `label`, `value` and `unit`, and one under the first `<Stat>` in `MyApp`. Hover over `label`: `Binding element 'label' implicitly has an 'any' type`. TypeScript was not told what a `Stat` expects, so each prop is `any`, which means unchecked.
- Hover over the first `<Stat>`: `Property 'unit' is missing`. With nothing else to go on, TypeScript guessed the props from the destructuring: three props, all required. A guessed contract is wrong in both directions, and that is the gap the next step closes.

### Step 1 - Describe the Props

- Above `Stat`, add an interface for its props.

```tsx
interface StatProps {
  label: string;
  value: number;
}
```

- Annotate the props parameter with it.

```tsx
function Stat({ label, value, unit }: StatProps) {
```

- The three squiggles inside `Stat` are gone, and two new ones appear: `unit` in the parameter and `unit="km"` in `MyApp`. Hover: `Property 'unit' does not exist on type 'StatProps'`. The interface is the whole truth about a `Stat`. Anything not in it is an error on both sides, where it is read and where it is passed.
- `interface` is the TypeScript keyword for naming an object shape. The naming convention is the component name plus `Props`.

### Step 2 - Add an Optional Prop

- Add `unit` to the interface with a question mark.

```tsx
interface StatProps {
  label: string;
  value: number;
  unit?: string;
}
```

- `?` means the prop may be left out. Inside the component its type is `string | undefined`, so give it a default where it is destructured.

```tsx
function Stat({ label, value, unit = "" }: StatProps) {
```

- Everything is clear. Run it: the same three cards, but now `Stat` has a contract.
- A required prop has no `?`. Leaving it out is an error. An optional prop has `?`, and the component has to cope with it missing, usually with a default.

### Step 3 - Break the Contract

- Change `value={320}` to `value="320"`. Hover: `Type 'string' is not assignable to type 'number'`. In JavaScript this would silently work until someone tried to add to it.
- Delete `label="Steps"` from the first card. Hover over `Stat`: `Property 'label' is missing`. This is the PropTypes `isRequired` warning from Week 1, except it is in the editor and it is red.
- Put them back. Then, in `MyApp`, start typing a fourth card: `<Stat ` and press Ctrl+Space. The editor lists `label`, `value` and `unit`, with their types, and marks which are optional. The interface is also documentation that cannot go stale.
- Delete the half-typed card before moving on.

### Finished Code

- Compare against this if something is not working.

```tsx
import { View, Text, StyleSheet } from "react-native";

interface StatProps {
  label: string;
  value: number;
  unit?: string;
}

function Stat({ label, value, unit = "" }: StatProps) {
  return (
    <View style={styles.card}>
      <Text style={styles.value}>
        {value} {unit}
      </Text>
      <Text style={styles.label}>{label}</Text>
    </View>
  );
}

export default function MyApp() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Today</Text>
      <Stat label="Steps" value={8412} />
      <Stat label="Distance" value={6.2} unit="km" />
      <Stat label="Calories" value={320} unit="kcal" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, paddingHorizontal: 20 },
  title: { fontSize: 28, fontWeight: "700", marginBottom: 16 },
  card: {
    padding: 16,
    marginBottom: 12,
    borderRadius: 8,
    backgroundColor: "#E3F2FD",
  },
  value: { fontSize: 24, fontWeight: "600" },
  label: { fontSize: 14, color: "#555" },
});
```

## Example: Typed State and Events

- `useState` usually needs no type: TypeScript reads it from the initial value. The exceptions are state that starts empty or starts as `null`, because the initial value does not say what comes later.

### Starter Code

- Replace everything in `App.tsx`. A guest list: type a name, add it, tap a guest to check them in.

```tsx
import { useState } from "react";
import {
  View,
  Text,
  TextInput,
  Button,
  Pressable,
  StyleSheet,
} from "react-native";

export default function MyApp() {
  const [name, setName] = useState("");
  const [guests, setGuests] = useState([]);
  const [selected, setSelected] = useState(null);

  function addGuest() {
    if (name.trim() === "") return;
    setGuests((prev) => [...prev, name.trim()]);
    setName("");
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Guest List</Text>
      <TextInput
        style={styles.input}
        placeholder="Name"
        value={name}
        onChangeText={(text) => setName(text)}
      />
      <Button title="Add" onPress={addGuest} />
      {guests.map((guest, index) => (
        <Pressable key={index} onPress={() => setSelected(guest)}>
          <Text style={styles.guest}>{guest}</Text>
        </Pressable>
      ))}
      <Text style={styles.message}>
        {selected ? `Checking in: ${selected}` : "Tap a guest to check them in"}
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, paddingHorizontal: 20 },
  title: { fontSize: 28, fontWeight: "700", marginBottom: 16 },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    borderRadius: 8,
    padding: 12,
    marginBottom: 8,
    fontSize: 16,
  },
  guest: { fontSize: 18, paddingVertical: 8 },
  message: { marginTop: 16, fontSize: 16, color: "#555" },
});
```

- Run it. Add a couple of names and tap one. It all works, and the editor has one squiggle, under the updater function inside `setGuests`. Hover: `Type 'string' is not assignable to type 'never'`.

### Step 1 - Let Inference Do Its Job, Then Help It

- Hover over `name` in `useState("")`. The tooltip says `const name: string`. The initial value was a string, so the state is a string, and `setName` only accepts strings. No annotation needed. The same goes for a counter that starts at `0` or a flag that starts `false`.
- Hover over `guests`. The tooltip says `never[]`: an array that can never hold anything. An empty array tells TypeScript nothing about what will go in it, so it assumes nothing may.
- Tell it. `useState` takes the state type in angle brackets.

```tsx
const [guests, setGuests] = useState<string[]>([]);
```

- The squiggle moves. It is now on `guest` in `setSelected(guest)`. Hover: `Argument of type 'string' is not assignable to parameter of type 'SetStateAction<null>'`. `selected` started as `null`, so TypeScript decided it is always `null`.

### Step 2 - State That Can Be Null

- `selected` is sometimes a string and sometimes `null`. Say both.

```tsx
const [selected, setSelected] = useState<string | null>(null);
```

- `string | null` is a union type: one of these. The `|` reads as "or".
- Everything is clear. Run it again. Nothing changed on screen, which is the point: the app was already correct, and the editor can now prove it.
- Rule of thumb: annotate `useState` when the initial value is `[]` or `null`. Otherwise leave it to inference.

### Step 3 - Type the Handlers

- Move the two handlers into named functions. Under `addGuest`, add one for the input.

```tsx
function handleChange(text: string) {
  setName(text);
}
```

- Use it: `onChangeText={handleChange}`.
- Give `addGuest` a return type. `void` means it returns nothing.

```tsx
function addGuest(): void {
```

- Now change `text: string` to `text: number`. The squiggle is not inside `handleChange`; it is on `onChangeText` in the JSX. `TextInput` declares that it calls `onChangeText` with a string, so a function that wants a number does not fit. The component tells you what it passes.
- Put `string` back. In practice you rarely write the parameter type for inline arrows like `(text) => setName(text)`, because TypeScript reads it from the prop. Named handlers need it written down.

### Finished Code

- Compare against this if something is not working.

```tsx
import { useState } from "react";
import {
  View,
  Text,
  TextInput,
  Button,
  Pressable,
  StyleSheet,
} from "react-native";

export default function MyApp() {
  const [name, setName] = useState("");
  const [guests, setGuests] = useState<string[]>([]);
  const [selected, setSelected] = useState<string | null>(null);

  function addGuest(): void {
    if (name.trim() === "") return;
    setGuests((prev) => [...prev, name.trim()]);
    setName("");
  }

  function handleChange(text: string) {
    setName(text);
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Guest List</Text>
      <TextInput
        style={styles.input}
        placeholder="Name"
        value={name}
        onChangeText={handleChange}
      />
      <Button title="Add" onPress={addGuest} />
      {guests.map((guest, index) => (
        <Pressable key={index} onPress={() => setSelected(guest)}>
          <Text style={styles.guest}>{guest}</Text>
        </Pressable>
      ))}
      <Text style={styles.message}>
        {selected ? `Checking in: ${selected}` : "Tap a guest to check them in"}
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, paddingHorizontal: 20 },
  title: { fontSize: 28, fontWeight: "700", marginBottom: 16 },
  input: {
    borderWidth: 1,
    borderColor: "#ccc",
    borderRadius: 8,
    padding: 12,
    marginBottom: 8,
    fontSize: 16,
  },
  guest: { fontSize: 18, paddingVertical: 8 },
  message: { marginTop: 16, fontSize: 16, color: "#555" },
});
```

## Example: Typed API Data

- This is where TypeScript pays for itself. Data from a server is the one thing in the app you did not write, and every `item.name` is a guess about its shape. An interface turns the guess into something the editor can check.

### Starter Code

- Replace everything in `App.tsx`. The Week 5 fetch pattern: load the users, show a spinner, then a list.

```tsx
import { useState, useEffect } from "react";
import {
  View,
  Text,
  FlatList,
  ActivityIndicator,
  StyleSheet,
} from "react-native";

const URL = "https://jsonplaceholder.typicode.com/users";

export default function MyApp() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(URL)
      .then((res) => res.json())
      .then((json) => setUsers(json))
      .finally(() => setLoading(false));
  }, []);

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={users}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.row}>
            <Text style={styles.name}>{item.name}</Text>
            <Text style={styles.detail}>{item.email}</Text>
            <Text style={styles.detail}>{item.company.name}</Text>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  row: { padding: 20, borderBottomWidth: 1, borderColor: "#ddd" },
  name: { fontSize: 18, fontWeight: "600" },
  detail: { fontSize: 14, color: "#666" },
});
```

- Run it. The list loads. The editor has two squiggles, both in the `FlatList`. Hover over `id` in `keyExtractor`: `Property 'id' does not exist on type 'never'`. Same cause as the guest list: `useState([])` is a `never[]`, so each `item` is a `never`, and a `never` has no properties.
- Hover over `item` in `renderItem`: `Binding element 'item' implicitly has an 'any' type`. The `Stat` starter all over again. TypeScript does not know what a row is, so `item.name`, `item.email` and `item.company.name` are not checked at all. Misspell one and nothing turns red.
- Notice there is no squiggle on `setUsers(json)` either. `res.json()` returns `any`, and `any` fits anywhere. That is the problem with `any`: it switches the checker off, and the error shows up somewhere else or not at all.

### Step 1 - Describe the Data

- Above `MyApp`, describe one user. Only the fields this screen uses; the API sends more, and extra fields are fine.

```tsx
interface User {
  id: number;
  name: string;
  email: string;
  company: { name: string };
}
```

- Type the state and the response.

```tsx
const [users, setUsers] = useState<User[]>([]);
```

```tsx
.then((json: User[]) => setUsers(json))
```

- The `keyExtractor` squiggle is gone. Hover over its `item`: it is a `User` now, worked out from the `data` prop. The `renderItem` squiggle stays, because the object `renderItem` receives does not pick up the row type on its own. Say what it is in the parameter.

```tsx
renderItem={({ item }: { item: User }) => (
```

- Every squiggle is gone. Type `item.` inside `renderItem` and the editor offers `id`, `name`, `email` and `company`.
- `json: User[]` is a promise you are making, not a check. TypeScript never sees the server. If the API changes, the interface is wrong and the editor cannot know. The interface documents what you expect; the error handling from Week 5 covers the day the server disagrees.

### Step 2 - Literal Types for Status

- The screen has three states: loading, ready, and the error case this starter skipped. A boolean cannot hold three. Above `MyApp`, under the interface, name them.

```tsx
type Status = "loading" | "ready" | "error";
```

- `type` is like `interface`, but it can name any type, not only object shapes. This one is a union of three string literals: the only values a `Status` may be.
- Replace the `loading` state.

```tsx
const [status, setStatus] = useState<Status>("loading");
```

- Replace the fetch chain so it records the outcome. The `res.ok` check is the one from Week 5: `fetch` only rejects when the network fails, so a 404 has to be turned into an error by hand.

```tsx
fetch(URL)
  .then((res) => {
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();
  })
  .then((json: User[]) => {
    setUsers(json);
    setStatus("ready");
  })
  .catch(() => setStatus("error"));
```

- Replace the `if (loading)` block with one return per state.

```tsx
if (status === "loading") {
  return (
    <View style={styles.center}>
      <ActivityIndicator size="large" />
    </View>
  );
}

if (status === "error") {
  return (
    <View style={styles.center}>
      <Text>Could not load the users. Check your connection.</Text>
    </View>
  );
}
```

- Run it. Same screen as before. Now change `setStatus("ready")` to `setStatus("done")`. Hover: `Argument of type '"done"' is not assignable to parameter of type 'Status'`. A typo in a status string is a compile error, not a screen that is stuck on the spinner.
- Change the URL to `https://jsonplaceholder.typicode.com/nothing` and run it. The server answers 404, the `res.ok` check throws, and the error screen shows. Without that check, the 404 body parses as an empty object and the list quietly renders nothing. Put both back.

### Step 3 - The Payoff

- In `renderItem`, change `item.email` to `item.emial`. Squiggle: `Property 'emial' does not exist on type 'User'`. In the JavaScript version that line renders an empty `Text` and nobody notices until a user does.
- Fix the typo. That one squiggle is the whole argument for typing API data.

### Finished Code

- Compare against this if something is not working.

```tsx
import { useState, useEffect } from "react";
import {
  View,
  Text,
  FlatList,
  ActivityIndicator,
  StyleSheet,
} from "react-native";

const URL = "https://jsonplaceholder.typicode.com/users";

interface User {
  id: number;
  name: string;
  email: string;
  company: { name: string };
}

type Status = "loading" | "ready" | "error";

export default function MyApp() {
  const [users, setUsers] = useState<User[]>([]);
  const [status, setStatus] = useState<Status>("loading");

  useEffect(() => {
    fetch(URL)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((json: User[]) => {
        setUsers(json);
        setStatus("ready");
      })
      .catch(() => setStatus("error"));
  }, []);

  if (status === "loading") {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  if (status === "error") {
    return (
      <View style={styles.center}>
        <Text>Could not load the users. Check your connection.</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={users}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }: { item: User }) => (
          <View style={styles.row}>
            <Text style={styles.name}>{item.name}</Text>
            <Text style={styles.detail}>{item.email}</Text>
            <Text style={styles.detail}>{item.company.name}</Text>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  row: { padding: 20, borderBottomWidth: 1, borderColor: "#ddd" },
  name: { fontSize: 18, fontWeight: "600" },
  detail: { fontSize: 14, color: "#666" },
});
```

## Example: any and unknown

- Paste these lines above `MyApp` in any of the examples.

```ts
let value: any = "hello";
value = 123;
value.toUpperCase();
```

### What to Show

1. No squiggles anywhere. `any` tells TypeScript to stop checking this variable, so reassigning a number is fine and calling a string method on it is fine.
2. Run it. The app crashes with `value.toUpperCase is not a function`. That is the runtime error TypeScript exists to prevent, and `any` let it through.
3. Change `any` to `unknown`. Now `value.toUpperCase()` has a squiggle: `'value' is of type 'unknown'`. `unknown` also accepts anything, but it will not let you use the value until you have checked what it is.

```ts
if (typeof value === "string") {
  value.toUpperCase();
}
```

4. Inside the `if`, TypeScript knows `value` is a string, so the call is allowed. The check narrows the type.
5. Delete the lines before moving on. The lesson: when you reach for `any` to make a squiggle go away, you have switched off the thing that found the bug.

## TypeScript Checklist

- Rename `App.js` to `App.tsx` and let the squiggles guide you. Red means TypeScript found something; the app will still run.
- Let TypeScript infer when the value is obvious: `useState(0)`, `useState("")`, a mapped array, a string.
- Annotate state that starts as `[]` or `null`: `useState<string[]>([])`, `useState<User | null>(null)`.
- Give every component with props an interface named after it, `StatProps`. Mark optional props with `?` and give them a default.
- Describe API data with an interface before you fetch it, and only the fields you use. Give `FlatList` rows the type too: `({ item }: { item: User })`.
- Use a union of string literals for a value with a fixed set of options, like a status.
- Named event handlers need parameter types: `(text: string)`. Inline arrows in JSX usually do not.
- Avoid `any`. If you really do not know the type yet, `unknown` keeps the checker on.
- Keep the Week 2 updater rule: `setCount((prev) => prev + 1)`. TypeScript checks the types, not the logic.
