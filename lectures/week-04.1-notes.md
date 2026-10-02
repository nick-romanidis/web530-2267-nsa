# Week 4.1 - Notes and Examples

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.js_
(https://snack.expo.dev)

- Keep Snack on SDK 54 (the default). SDK 55 currently fails to load `react-native-screens`, so no navigation example runs on it.
- Replace the contents of `package.json` before starting. Every example today uses it.

_package.json_

```json
{
  "dependencies": {
    "@react-navigation/native": "^7.0.0",
    "@react-navigation/native-stack": "^7.0.0",
    "@react-navigation/bottom-tabs": "^7.0.0",
    "@react-navigation/drawer": "^7.0.0",
    "react-native-screens": "~4.16.0",
    "react-native-safe-area-context": "~5.6.0",
    "react-native-gesture-handler": "~2.28.0",
    "react-native-reanimated": "~4.1.1",
    "react-native-worklets": "0.5.1",
    "@expo/vector-icons": "^15.0.3",
    "jotai": "*",
    "@types/react": "*"
  }
}
```

- The `@react-navigation` packages are pinned to version 7. Version 7 changed how `navigate()` behaves, so older tutorials can disagree with what students see.
- The other versions match SDK 54. If you switch SDKs, Snack flags the ones that don't match.
- `jotai` is for the last example. Snack asks for `@types/react` alongside it, even though Jotai doesn't need it.

## Why Navigation?

- A web browser gives you navigation for free: every page has a URL, and the back button walks through the history.
- React Native has none of that. Screens, transitions and the history of where the user has been are all managed in JavaScript.
- Real apps also want deep linking: opening the app directly on a screen, for example from a notification.
- React Navigation is the community standard. It provides stack, tab and drawer navigators, deep linking, and typed params for TypeScript.
- Once it is set up, it also handles the Android back button for you.

## Example: The NavigationContainer

- Every navigation app starts with a `NavigationContainer`. It holds the navigation state for the whole app.
- Show the empty shell. The navigators go inside it.

```jsx
import { NavigationContainer } from "@react-navigation/native";

export default function MyApp() {
  return <NavigationContainer>{/* Navigators go here */}</NavigationContainer>;
}
```

- It must be the outermost navigation component.
- Only one per app. A nested navigator never needs its own container.
- Think of it like the router in a React web app.

## Example: Stack Navigator

- Screens stack on top of each other like a deck of cards. Navigating forward **pushes** a screen, going back **pops** it.
- The most common navigation pattern in mobile apps.

### Starter Code

- Paste this into `App.js`. The screens, buttons and styles are done. Nothing is connected yet.
- It runs as is: `MyApp` shows `HomeScreen` directly, and the button does nothing.

```jsx
import { View, Text, Button, StyleSheet } from "react-native";

function HomeScreen() {
  return (
    <View style={styles.screen}>
      <Text>Home Screen</Text>
      <Button title="Go to Settings" />
    </View>
  );
}

function SettingsScreen() {
  return (
    <View style={styles.screen}>
      <Text>Settings Screen</Text>
      <Button title="Go Back" />
    </View>
  );
}

export default function MyApp() {
  return <HomeScreen />;
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```

### Step 1 - Import the Navigation Pieces

- Add two imports under the `react-native` import.

```jsx
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
```

- `NavigationContainer` comes from the core package. Every navigator type needs it.
- `createNativeStackNavigator` comes from the stack package. Tabs and drawers have their own packages later today.

### Step 2 - Create the Stack

- Add this line under the imports, outside every component.

```jsx
const Stack = createNativeStackNavigator();
```

- `createNativeStackNavigator()` returns an object with a `Navigator` and a `Screen`, used as `Stack.Navigator` and `Stack.Screen`.
- Create it once, at the top level. Calling it inside a component would build a new navigator on every render.

### Step 3 - Register the Screens

- Replace the `return` in `MyApp` with the container, the navigator, and one `Stack.Screen` per screen.

```jsx
export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Settings" component={SettingsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

- Each `Stack.Screen` maps a route name to a component. The `name` is what you navigate to.
- Pass the component itself, `component={HomeScreen}`, not `<HomeScreen />`. The navigator decides when to render it.
- `initialRouteName` picks the first screen. Without it, the first screen listed is used.
- Run it. Home now has a header titled "Home", taken from the route name. The button still does nothing, because nothing tells it where to go.

### Step 4 - Navigate Forward

- Every screen registered in Step 3 receives a `navigation` prop automatically. Destructure it in `HomeScreen`.

```jsx
function HomeScreen({ navigation }) {
```

- Give the button an `onPress` that navigates to the `Settings` route.

```jsx
<Button
  title="Go to Settings"
  onPress={() => navigation.navigate("Settings")}
/>
```

- The string must match a `name` from Step 3 exactly. A typo does nothing on screen, and the console logs that the `NAVIGATE` action "was not handled by any navigator".
- Run it. Pressing the button pushes Settings onto the stack, and a back arrow appears in its header on its own.

### Step 5 - Go Back

- Do the same in `SettingsScreen`: destructure `navigation`, then add an `onPress` to the button.

```jsx
function SettingsScreen({ navigation }) {
```

```jsx
<Button title="Go Back" onPress={() => navigation.goBack()} />
```

- `goBack()` pops the current screen off the stack.
- Show that the button and the header's back arrow do the same job. You only need your own back button when the design calls for one.

### Finished Code

- This is the completed example. Compare it against yours if something doesn't work and you can't find the typo.

```jsx
import { View, Text, Button, StyleSheet } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";

const Stack = createNativeStackNavigator();

function HomeScreen({ navigation }) {
  return (
    <View style={styles.screen}>
      <Text>Home Screen</Text>
      <Button
        title="Go to Settings"
        onPress={() => navigation.navigate("Settings")}
      />
    </View>
  );
}

function SettingsScreen({ navigation }) {
  return (
    <View style={styles.screen}>
      <Text>Settings Screen</Text>
      <Button title="Go Back" onPress={() => navigation.goBack()} />
    </View>
  );
}

export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Settings" component={SettingsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```

## Example: Navigate vs Push

- `navigate()` and `push()` look the same until you call them for a screen that is already open.
- Start from the finished Stack Navigator code. Every change in this example goes in `SettingsScreen`, plus one line in the styles.

### Step 1 - Make Room for More Buttons

- Add `gap: 8` to `styles.screen` so the buttons don't touch.

```jsx
screen: {
  flex: 1,
  alignItems: "center",
  justifyContent: "center",
  gap: 8,
},
```

### Step 2 - Navigate to the Screen You Are On

- Add this button to `SettingsScreen`, under the **Go Back** button.

```jsx
<Button
  title="Navigate to Settings"
  onPress={() => navigation.navigate("Settings")}
/>
```

- Run it, open Settings and press it. Nothing happens. You are already on Settings, so `navigate()` stays put.

### Step 3 - Push the Screen You Are On

- Add a second button under it. The only difference is `push` instead of `navigate`.

```jsx
<Button title="Push Settings" onPress={() => navigation.push("Settings")} />
```

- `push()` always adds another copy. Press it three times, then count the back presses to reach Home. It takes four, one per copy of Settings.
- Show the header after each press. Every copy has its own back arrow.

### Step 4 - Navigate to a Screen Below You

- Add a third button that navigates to Home.

```jsx
<Button
  title="Navigate to Home"
  onPress={() => navigation.navigate("Home")}
/>
```

- This is the trap. Home is below you in the stack, but `navigate()` does not go back to it. It pushes a second Home on top.
- Show that this Home has a back arrow in its header. The first Home never does.
- To go back to a screen, use `goBack()` or `popTo()` from the next example.
- `push()` suits drill-down flows, like opening a related product from a product page.

### Finished SettingsScreen

- Only `SettingsScreen` and the `gap` line changed. Compare against this if a button doesn't work.

```jsx
function SettingsScreen({ navigation }) {
  return (
    <View style={styles.screen}>
      <Text>Settings Screen</Text>
      <Button title="Go Back" onPress={() => navigation.goBack()} />
      <Button
        title="Navigate to Settings"
        onPress={() => navigation.navigate("Settings")}
      />
      <Button title="Push Settings" onPress={() => navigation.push("Settings")} />
      <Button
        title="Navigate to Home"
        onPress={() => navigation.navigate("Home")}
      />
    </View>
  );
}
```

## Example: Going Back

- A stack has three ways to go back.
- Start from the finished Navigate vs Push code. Every change goes in `SettingsScreen` again.

### Step 1 - Go Back One Screen

- You already have this one. The **Go Back** button calls `goBack()`.
- Run it, open Settings, and press **Push Settings** three times. Now press **Go Back** once.
- `goBack()` pops one screen, like the back button in a browser. It does the same job as the arrow in the header.
- You can't tell the copies of Settings apart yet. Params, two examples from now, are how each screen gets its own data.

### Step 2 - Fix the Navigate to Home Trap

- In the last example, **Navigate to Home** put a second Home on top. Change that button to use `popTo()`, and rename it.

```jsx
<Button title="Pop to Home" onPress={() => navigation.popTo("Home")} />
```

- Push a few copies of Settings, then press **Pop to Home**. One press takes you all the way back.
- Show that Home has no back arrow this time. It is the original Home, not a new copy.
- `popTo("Home")` pops screens until the named one is on top. Use it to jump several levels back to a specific screen.

### Step 3 - Pop to the Top

- Add one more button under **Pop to Home**.

```jsx
<Button title="Pop to Top" onPress={() => navigation.popToTop()} />
```

- `popToTop()` pops everything and returns to the first screen in the stack. Useful after logging out.
- Here it lands on Home, the same as **Pop to Home**, because Home is the first screen. `popTo()` is for a named screen anywhere in the stack, and `popToTop()` always goes to the first one.

### Finished SettingsScreen

- Compare against this if a button doesn't work.

```jsx
function SettingsScreen({ navigation }) {
  return (
    <View style={styles.screen}>
      <Text>Settings Screen</Text>
      <Button title="Go Back" onPress={() => navigation.goBack()} />
      <Button
        title="Navigate to Settings"
        onPress={() => navigation.navigate("Settings")}
      />
      <Button title="Push Settings" onPress={() => navigation.push("Settings")} />
      <Button title="Pop to Home" onPress={() => navigation.popTo("Home")} />
      <Button title="Pop to Top" onPress={() => navigation.popToTop()} />
    </View>
  );
}
```

## Example: The useNavigation Hook

- Only screens get the `navigation` prop.
- Show a reusable button that is not a screen. It gets the same object from the `useNavigation()` hook.
- Start from the finished Going Back code. This time the changes go in `HomeScreen`, a new component, and the imports.

### Step 1 - Move the Button into Its Own Component

- Add a new component above `HomeScreen`, and move Home's **Go to Settings** button into it.

```jsx
// Not a screen, so it doesn't receive a navigation prop.
function SettingsButton() {
  return (
    <Button
      title="Go to Settings"
      onPress={() => navigation.navigate("Settings")}
    />
  );
}
```

- In `HomeScreen`, put the new component where the button was.

```jsx
<SettingsButton />
```

- Run it and press the button. It crashes with an error saying `navigation` is not defined.
- `SettingsButton` isn't registered as a `Stack.Screen`, so nothing gives it a `navigation` prop.

### Step 2 - Import the Hook

- Add `useNavigation` to the existing `@react-navigation/native` import.

```jsx
import { NavigationContainer, useNavigation } from "@react-navigation/native";
```

### Step 3 - Call the Hook

- Add this as the first line inside `SettingsButton`.

```jsx
const navigation = useNavigation();
```

- Run it. The button works again.
- `useNavigation()` returns the same object a screen gets as its prop, from whichever screen the component is inside.
- Hooks can only be called inside components, so this is the same rule as `useState`.

### Step 4 - Clean Up HomeScreen

- `HomeScreen` no longer uses `navigation`, so remove it from the parameters.

```jsx
function HomeScreen() {
```

- Without the hook, `HomeScreen` would have to pass `navigation` down as a prop, through every component in between.

### Finished Code

- Only the import, `SettingsButton` and `HomeScreen` changed. Compare against this if the button doesn't work.

```jsx
import { NavigationContainer, useNavigation } from "@react-navigation/native";
```

```jsx
// Not a screen, so it doesn't receive a navigation prop.
function SettingsButton() {
  const navigation = useNavigation();

  return (
    <Button
      title="Go to Settings"
      onPress={() => navigation.navigate("Settings")}
    />
  );
}

function HomeScreen() {
  return (
    <View style={styles.screen}>
      <Text>Home Screen</Text>
      <SettingsButton />
    </View>
  );
}
```

## Example: Passing Params

- Screens often need data from the screen before, like the item a user tapped.
- Pass an object as the second argument to `navigate()` or `push()`. The next screen reads it from `route.params`.
- Start from the finished useNavigation code. Each copy of Settings gets a `level` param, so you can finally tell the copies apart.

### Step 1 - Remove the Navigate to Settings Button

- Delete the **Navigate to Settings** button from `SettingsScreen`. It was only there for the Navigate vs Push example.

### Step 2 - Send a Param

- In `SettingsButton`, add a second argument to `navigate()`.

```jsx
onPress={() => navigation.navigate("Settings", { level: 1 })}
```

- Params can be any JavaScript object: strings, numbers, arrays, other objects.

### Step 3 - Read the Param

- Every screen gets a `route` prop alongside `navigation`. Add it to `SettingsScreen`, and read `level` from `route.params`.

```jsx
function SettingsScreen({ navigation, route }) {
  const { level } = route.params;
```

- Change the text to show the level.

```jsx
<Text>Settings Level {level}</Text>
```

- Run it. Settings now says **Settings Level 1**.

### Step 4 - Push the Next Level

- Give **Push Settings** a param too, one higher than the current level.

```jsx
<Button
  title="Push Settings"
  onPress={() => navigation.push("Settings", { level: level + 1 })}
/>
```

- Push a few times, then press **Go Back**. The levels count down, one copy at a time. These are the copies the Going Back example couldn't tell apart.

### Step 5 - Show the Crash, Then Fix It

- Always treat params as optional. A screen opened without them (as the first screen, or from a deep link) gets `undefined`, and destructuring `undefined` crashes the app.
- Show the crash. In `MyApp`, change `initialRouteName` to `"Settings"`.

```jsx
<Stack.Navigator initialRouteName="Settings">
```

- Run it. The app crashes, because nothing passed params to the first screen.
- Fix the line in `SettingsScreen`.

```jsx
const { level = 1 } = route.params ?? {};
```

- `route.params ?? {}` falls back to an empty object, and `level = 1` gives a default.
- Run it. Settings opens as **Settings Level 1**. Change `initialRouteName` back to `"Home"`.

### Step 6 - Update the Current Screen's Params

- Add a button under **Push Settings** that resets the level without navigating.

```jsx
<Button
  title="Reset to Level 1"
  onPress={() => navigation.setParams({ level: 1 })}
/>
```

- Push to level 3 or 4, then press **Reset to Level 1**. The text changes, but you are still on the same screen. **Go Back** still goes back the same number of times.
- `setParams()` updates the current screen's params, and the screen re-renders. It merges, so params you don't mention keep their values.

### Finished Code

- Only `SettingsButton` and `SettingsScreen` changed, and `initialRouteName` should be back to `"Home"`. Compare against this if something doesn't work.

```jsx
// Not a screen, so it doesn't receive a navigation prop.
function SettingsButton() {
  const navigation = useNavigation();

  return (
    <Button
      title="Go to Settings"
      onPress={() => navigation.navigate("Settings", { level: 1 })}
    />
  );
}
```

```jsx
function SettingsScreen({ navigation, route }) {
  const { level = 1 } = route.params ?? {};

  return (
    <View style={styles.screen}>
      <Text>Settings Level {level}</Text>
      <Button title="Go Back" onPress={() => navigation.goBack()} />
      <Button
        title="Push Settings"
        onPress={() => navigation.push("Settings", { level: level + 1 })}
      />
      <Button
        title="Reset to Level 1"
        onPress={() => navigation.setParams({ level: 1 })}
      />
      <Button title="Pop to Home" onPress={() => navigation.popTo("Home")} />
      <Button title="Pop to Top" onPress={() => navigation.popToTop()} />
    </View>
  );
}
```

## Example: Customizing the Header

- Each screen can set header options.
- `options` can be an object, or a function that receives the `route`. Use the function to read params.
- Start from the finished Passing Params code. Every change goes in `MyApp`, on the navigator and its screens.

### Step 1 - Style Every Header

- Add `screenOptions` to `Stack.Navigator`.

```jsx
<Stack.Navigator
  initialRouteName="Home"
  screenOptions={{
    headerStyle: { backgroundColor: "#4f46e5" },
    headerTintColor: "#fff",
  }}
>
```

- Run it. Both headers turn purple with white text.
- `screenOptions` on the `Navigator` applies to every screen in it.
- `headerTintColor` colours the title and the back arrow.

### Step 2 - Give Home a Title

- Add `options` to the Home screen.

```jsx
<Stack.Screen
  name="Home"
  component={HomeScreen}
  options={{ title: "My App" }}
/>
```

- `title` replaces the route name in the header.
- `options` on one `Stack.Screen` applies to just that screen, and overrides `screenOptions`.

### Step 3 - Put the Level in the Header

- Settings needs a title that changes with its params, so its `options` is a function that receives the `route`.

```jsx
<Stack.Screen
  name="Settings"
  component={SettingsScreen}
  options={({ route }) => ({
    title: `Level ${route.params?.level ?? 1}`,
  })}
/>
```

- Run it and push a few levels. Each header shows its own level.
- Press **Reset to Level 1**. The header updates too, because `setParams()` re-runs the `options` function.
- `route.params?.level ?? 1` is the same rule as Step 5 of the last example: params might be missing, so give a default.

### Step 4 - Add a Header Button

- The `options` function also receives `navigation`. Add `headerRight` to put a **Reset** button in the header, greyed out when you are already on level 1.

```jsx
options={({ navigation, route }) => ({
  title: `Level ${route.params?.level ?? 1}`,
  headerRight: () => (
    <Button
      title="Reset"
      onPress={() => navigation.setParams({ level: 1 })}
      disabled={(route.params?.level ?? 1) === 1}
    />
  ),
})}
```

- Push to level 3, then press **Reset** in the header. It does the same job as the **Reset to Level 1** button on the screen, then greys itself out.
- `headerRight` puts any component on the right side of the header. `headerLeft` does the same on the left, but replaces the back arrow.
- `headerShown: false` hides the header completely. It comes back in the nesting example.

### Finished Code

- Only `MyApp` changed. Compare against this if the header doesn't look right.

```jsx
export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          headerStyle: { backgroundColor: "#4f46e5" },
          headerTintColor: "#fff",
        }}
      >
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{ title: "My App" }}
        />
        <Stack.Screen
          name="Settings"
          component={SettingsScreen}
          options={({ navigation, route }) => ({
            title: `Level ${route.params?.level ?? 1}`,
            headerRight: () => (
              <Button
                title="Reset"
                onPress={() => navigation.setParams({ level: 1 })}
                disabled={(route.params?.level ?? 1) === 1}
              />
            ),
          })}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

## Example: Tab Navigator

- Tabs are for the top-level sections of an app. The tab bar is always visible.
- Same `Navigator` and `Screen` pattern as the stack, with a different creation function.

### Starter Code

- This is a new app, so replace everything in `App.js`. The three screens and styles are done. Nothing is connected yet.
- It runs as is: `MyApp` shows `HomeScreen` directly. The counter on Home is there to show what happens to state when you switch tabs.

```jsx
import { useState } from "react";
import { View, Text, Button, StyleSheet } from "react-native";

function HomeScreen() {
  const [count, setCount] = useState(0);

  return (
    <View style={styles.screen}>
      <Text>Home Screen</Text>
      <Text>Count: {count}</Text>
      <Button title="Add One" onPress={() => setCount((prev) => prev + 1)} />
    </View>
  );
}

function ProfileScreen() {
  return (
    <View style={styles.screen}>
      <Text>Profile Screen</Text>
    </View>
  );
}

function SettingsScreen() {
  return (
    <View style={styles.screen}>
      <Text>Settings Screen</Text>
    </View>
  );
}

export default function MyApp() {
  return <HomeScreen />;
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
    gap: 8,
  },
});
```

### Step 1 - Import the Navigation Pieces

- Add two imports under the `react-native` import.

```jsx
import { NavigationContainer } from "@react-navigation/native";
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";
```

- `NavigationContainer` is the same as before. Every navigator needs it.
- `createBottomTabNavigator` comes from its own package, `@react-navigation/bottom-tabs`.

### Step 2 - Create the Tabs

- Add this line under the imports, outside every component.

```jsx
const Tab = createBottomTabNavigator();
```

- It returns the same shape as the stack: a `Tab.Navigator` and a `Tab.Screen`.

### Step 3 - Register the Screens

- Replace the `return` in `MyApp` with the container, the navigator, and one `Tab.Screen` per screen.

```jsx
export default function MyApp() {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen name="Home" component={HomeScreen} />
        <Tab.Screen name="Profile" component={ProfileScreen} />
        <Tab.Screen name="Settings" component={SettingsScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
```

- Run it. A tab bar appears at the bottom with one tab per screen, in the order you listed them.
- Show that no screen has a back arrow. Tabs switch between sections side by side, they don't stack.
- Each tab shows a placeholder symbol for now. Icons come in Step 5.

### Step 4 - Show That Tabs Keep Their State

- No code in this step. Press **Add One** a few times on Home, switch to Profile, then switch back.
- The count is still there. Tabs stay mounted when you switch away, so each tab keeps its state.
- Compare that to the stack: a screen you pop with `goBack()` is unmounted, and its state is gone.

### Step 5 - Add Icons

- Import an icon set under the other imports.

```jsx
import { Ionicons } from "@expo/vector-icons";
```

- Give the Home tab an `options` prop with a `tabBarIcon`.

```jsx
<Tab.Screen
  name="Home"
  component={HomeScreen}
  options={{
    tabBarIcon: ({ color, size }) => (
      <Ionicons name="home" color={color} size={size} />
    ),
  }}
/>
```

- Run it. Home has an icon, and the other two still show the placeholder.
- `tabBarIcon` is a function that receives the `color` and `size` the tab bar wants, so the icon changes colour when its tab is selected.
- Do the same for the other two tabs, with `name="person"` for Profile and `name="settings"` for Settings.
- Icons come from `@expo/vector-icons`, which is part of every Expo project. Search them at (https://icons.expo.fyi/).

### Step 6 - Colour the Selected Tab

- Add `screenOptions` to `Tab.Navigator`, the same prop the stack used for its headers.

```jsx
<Tab.Navigator screenOptions={{ tabBarActiveTintColor: "#4f46e5" }}>
```

- Run it. The selected tab's icon and label turn purple. That colour is what `tabBarIcon` receives as `color`.

### Finished Code

- This is the completed example. Compare it against yours if something doesn't work and you can't find the typo.

```jsx
import { useState } from "react";
import { View, Text, Button, StyleSheet } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";
import { Ionicons } from "@expo/vector-icons";

const Tab = createBottomTabNavigator();

function HomeScreen() {
  const [count, setCount] = useState(0);

  return (
    <View style={styles.screen}>
      <Text>Home Screen</Text>
      <Text>Count: {count}</Text>
      <Button title="Add One" onPress={() => setCount((prev) => prev + 1)} />
    </View>
  );
}

function ProfileScreen() {
  return (
    <View style={styles.screen}>
      <Text>Profile Screen</Text>
    </View>
  );
}

function SettingsScreen() {
  return (
    <View style={styles.screen}>
      <Text>Settings Screen</Text>
    </View>
  );
}

export default function MyApp() {
  return (
    <NavigationContainer>
      <Tab.Navigator screenOptions={{ tabBarActiveTintColor: "#4f46e5" }}>
        <Tab.Screen
          name="Home"
          component={HomeScreen}
          options={{
            tabBarIcon: ({ color, size }) => (
              <Ionicons name="home" color={color} size={size} />
            ),
          }}
        />
        <Tab.Screen
          name="Profile"
          component={ProfileScreen}
          options={{
            tabBarIcon: ({ color, size }) => (
              <Ionicons name="person" color={color} size={size} />
            ),
          }}
        />
        <Tab.Screen
          name="Settings"
          component={SettingsScreen}
          options={{
            tabBarIcon: ({ color, size }) => (
              <Ionicons name="settings" color={color} size={size} />
            ),
          }}
        />
      </Tab.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
    gap: 8,
  },
});
```

## Example: Drawer Navigator

- A drawer slides in from the side. Good for settings or secondary sections that don't need a permanent tab.

```jsx
import { View, Text, StyleSheet } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createDrawerNavigator } from "@react-navigation/drawer";

const Drawer = createDrawerNavigator();

function HomeScreen() {
  return (
    <View style={styles.screen}>
      <Text>Home Screen</Text>
    </View>
  );
}

function SettingsScreen() {
  return (
    <View style={styles.screen}>
      <Text>Settings Screen</Text>
    </View>
  );
}

export default function MyApp() {
  return (
    <NavigationContainer>
      <Drawer.Navigator>
        <Drawer.Screen name="Home" component={HomeScreen} />
        <Drawer.Screen name="Settings" component={SettingsScreen} />
      </Drawer.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```

- The hamburger icon appears in the header on its own. Every screen becomes a menu item.
- Compare the three navigators:
  - Each has its own creation function: `createNativeStackNavigator()`, `createBottomTabNavigator()`, `createDrawerNavigator()`.
  - All use the same `<X.Navigator>` and `<X.Screen>` JSX.
  - Screens get the same `navigation` and `route` props in all three.
  - They differ in how they look and behave, not in how you write screens.

## Example: Choosing a Navigator per Platform

- `Platform.select()` from week 3 returns the value for the current platform. It can return a whole navigator.
- The point to make: the same two screens end up in three different navigators. Only the `Platform.select()` block decides which.
- Replace everything in `App.js` with this. There's nothing to build, just run it on each platform.

```jsx
import { View, Text, Platform, StyleSheet } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";
import { createDrawerNavigator } from "@react-navigation/drawer";

const Stack = createNativeStackNavigator();
const Tab = createBottomTabNavigator();
const Drawer = createDrawerNavigator();

function HomeScreen() {
  return (
    <View style={styles.screen}>
      <Text>Home Content</Text>
    </View>
  );
}

function SettingsScreen() {
  return (
    <View style={styles.screen}>
      <Text>Settings Content</Text>
    </View>
  );
}

// Choose a navigator based on the platform.
const Navigator = Platform.select({
  ios: () => (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  ),
  android: () => (
    <Drawer.Navigator>
      <Drawer.Screen name="Home" component={HomeScreen} />
      <Drawer.Screen name="Settings" component={SettingsScreen} />
    </Drawer.Navigator>
  ),
  default: () => (
    <Stack.Navigator>
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="Settings" component={SettingsScreen} />
    </Stack.Navigator>
  ),
});

export default function MyApp() {
  return <NavigationContainer>{Navigator()}</NavigationContainer>;
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```

### How It Works

- `Platform.select()` looks at the keys `ios`, `android` and `default`. Web has no key of its own here, so it gets `default`.
- Each value is a function that returns a navigator. `Platform.select()` picks one function, and `MyApp` calls it with `Navigator()` inside the `NavigationContainer`.
- The screens are written once and reused by all three navigators. Only the wrapper changes.

### What to Show

1. **Web.** Run it in the Web preview. You see a header that says **Home** and nothing else: no tab bar, no menu. This is the stack from `default`.
2. **Point out the dead end.** There's no way to reach Settings on web. A stack only moves when something calls `navigate()`, and these screens have no buttons. That's one reason real apps combine navigators, which is the next example.
3. **iOS.** Switch the preview to iOS. A tab bar appears at the bottom with Home and Settings, and tapping a tab switches screens.
4. **Android.** Switch the preview to Android. The header has a menu icon on the left. Tap it, or swipe in from the left edge, to open the drawer with Home and Settings.
5. **Back to the code.** Point at the `Platform.select()` block. Three navigators, the same two screens, and nothing else in the app changed.

- The iOS and Android previews in Snack run on a remote simulator and can take a minute to start. Opening the Snack in Expo Go on a phone also works, but only shows your phone's platform.
- This is a demo, not a requirement. Tabs and drawers work on every platform.

## Example: Nesting Navigators

- Real apps almost always use more than one navigator: a stack for signing in, tabs for the main sections, a drawer for settings.
- They are combined by putting one navigator inside a screen of another.
- Common patterns:
  - Stack → Tabs: detail screens open over the tab bar.
  - Tabs → Stack: each tab has its own stack of screens.
  - Drawer → Tabs → Stack: very common in large apps.
  - Stack → Drawer: the drawer only appears after login.
- Show the first pattern: tabs inside a stack.
- Replace everything in `App.js` with this. There's nothing to build. The steps under **What to Show** walk through it.

```jsx
import { View, Text, Button, StyleSheet } from "react-native";
import { Ionicons } from "@expo/vector-icons";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";

const Stack = createNativeStackNavigator();
const Tab = createBottomTabNavigator();

function FeedScreen({ navigation }) {
  return (
    <View style={styles.screen}>
      <Text>Feed</Text>
      <Button title="Open Post" onPress={() => navigation.navigate("Post")} />
    </View>
  );
}

function AccountScreen() {
  return (
    <View style={styles.screen}>
      <Text>Account</Text>
    </View>
  );
}

function PostScreen({ navigation }) {
  return (
    <View style={styles.screen}>
      <Text>Post Details</Text>
      <Button
        title="Go to the Account tab"
        onPress={() => navigation.popTo("Main", { screen: "Account" })}
      />
    </View>
  );
}

// The tab navigator is itself a screen in the stack below.
function MainTabs() {
  return (
    <Tab.Navigator>
      <Tab.Screen
        name="Feed"
        component={FeedScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="newspaper" color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="Account"
        component={AccountScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person" color={color} size={size} />
          ),
        }}
      />
    </Tab.Navigator>
  );
}

export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen
          name="Main"
          component={MainTabs}
          options={{ headerShown: false }}
        />
        <Stack.Screen name="Post" component={PostScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```

### How It Works

- `MainTabs` is an ordinary component that returns a `Tab.Navigator`. The stack registers it as the `Main` screen, so the whole tab navigator is one screen in the stack.
- Still one `NavigationContainer`, at the very top. The nested navigator never gets its own.
- The stack holds two screens: `Main` (the tabs) and `Post`. `Post` is outside the tabs, which is why it covers the tab bar.
- Each navigator keeps its own history, and screens get the same props wherever they live.

### What to Show

1. **Run it.** You land on the Feed tab with a tab bar at the bottom and one header, titled **Feed**. That header belongs to the tab navigator.
2. **Show why `headerShown: false` is there.** Delete `options={{ headerShown: false }}` from the `Main` screen and run it. Now there are two headers: the stack's, titled **Main**, above the tab navigator's, titled **Feed**. Put it back.
3. **Up is automatic.** Press **Open Post**. Post slides in over the tabs, the tab bar disappears, and a back arrow appears.
   - `FeedScreen` calls `navigate("Post")`. The tab navigator has no `Post`, so the action passes up to the parent stack, which pushes `Post` over the tabs.
4. **Back to the tabs.** Press the back arrow. You return to Feed with the tab bar, exactly as you left it.
5. **Down needs the parent's name.** Open Post again and press **Go to the Account tab**. You land on the Account tab.
   - `PostScreen` can't name `Account` on its own, because `Account` is inside the tab navigator. Name the parent screen first, then the screen inside it: `popTo("Main", { screen: "Account" })`.
   - It uses `popTo()`, not `navigate()`, because `Main` is below Post in the stack. This is the same trap as Navigate vs Push.
6. **Show the failure.** Change the button to `navigation.navigate("Account")`. Press it and nothing happens. In development, React Navigation logs an error that no navigator handled the action.
7. **Show the trap (optional).** Change the button to `navigation.navigate("Main", { screen: "Account" })`. It looks like it works, but `navigate()` pushed a second copy of the tabs on top of Post. There's no back arrow, because `Main` hides its header. On Android, the back button takes you to Post instead of leaving the app. Change it back to `popTo()`.

## Example: Handling State Across Screens

- Params pass data one way, into the next screen. Data that several screens change needs shared state.
- React Navigation has no state API of its own. Use the same tools as any React app.
- Show Jotai from week 2 keeping stock counts in sync between the list and a header button.
- `jotai` is already in the `package.json` from the top of these notes.

### Starter Code

- This is a new app, so replace everything in `App.js`. The navigator, both screens, the header button and the styles are done.
- It runs as is, but the counts are a plain object, so nothing can change them. **Buy** does nothing, and it is greyed out for Banana because Banana starts at 0.

```jsx
import { View, Button, Text, StyleSheet } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";

const Stack = createNativeStackNavigator();

// Stock counts. A plain object for now, so nothing can change them.
const stock = { Apple: 5, Banana: 0, Cherry: 15 };

function HomeScreen({ navigation }) {
  return (
    <View style={styles.screen}>
      {Object.keys(stock).map((item) => (
        <Button
          key={item}
          title={`${item} (${stock[item]})`}
          onPress={() => navigation.navigate("Details", { item })}
        />
      ))}
    </View>
  );
}

function DetailsScreen({ route }) {
  const { item = "Apple" } = route.params ?? {};

  return (
    <View style={styles.screen}>
      <Text>
        {item}: {stock[item]} in stock
      </Text>
    </View>
  );
}

// Lives in the header, not inside a screen.
function BuyButton({ item }) {
  const qty = stock[item] ?? 0;

  return <Button title="Buy" disabled={qty === 0} />;
}

export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={({ route }) => ({
            title: route.params?.item ?? "Details",
            headerRight: () => <BuyButton item={route.params?.item} />,
          })}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```

- Before adding anything, point out the problem. `BuyButton` needs to change a count, and `HomeScreen` and `DetailsScreen` need to show it.
- `useState` in one of them can't reach the others. `BuyButton` is rendered by the header, and both screens are rendered by the navigator, so none of them is a parent you control.

### Step 1 - Import Jotai

- Add this import under the others.

```jsx
import { atom, useAtom } from "jotai";
```

### Step 2 - Turn the Object into an Atom

- Replace the plain `stock` object with an atom. Same values, wrapped in `atom()`, with a new name.

```jsx
// Global stock counts, shared by every screen.
const stockAtom = atom({ Apple: 5, Banana: 0, Cherry: 15 });
```

- Run it. It crashes, saying `stock` is not defined. Every component was reading the plain object, and it's gone.
- The next three steps fix them one at a time.

### Step 3 - Read the Atom in HomeScreen

- Add this as the first line inside `HomeScreen`.

```jsx
const [stock] = useAtom(stockAtom);
```

- Run it. The list is back. Don't open an item yet, because `DetailsScreen` and `BuyButton` still crash.
- Only the value is needed here, so only the first item of the array is kept.

### Step 4 - Read the Atom in DetailsScreen

- Add the same line as the first line inside `DetailsScreen`.

```jsx
const [stock] = useAtom(stockAtom);
```

### Step 5 - Read and Update the Atom in BuyButton

- `BuyButton` needs the setter too. Add this as its first line.

```jsx
const [stock, setStock] = useAtom(stockAtom);
```

- Give the button an `onPress` that takes one off this item's count.

```jsx
<Button
  title="Buy"
  disabled={qty === 0}
  onPress={() => setStock((prev) => ({ ...prev, [item]: prev[item] - 1 }))}
/>
```

- The updater form `(prev) => ...` is used because the new count depends on the old one.
- `{ ...prev, [item]: ... }` builds a new object with one count changed. Changing `prev` directly would not re-render anything.

### Step 6 - Show That Every Screen Stays in Sync

- No code in this step. Open Apple and press **Buy** a few times. The count on the Details screen goes down.
- Keep pressing until it reaches 0. **Buy** greys itself out.
- Go back. The count on the list has changed too.
- `HomeScreen`, `DetailsScreen` and `BuyButton` never pass the counts to each other. They all read the same atom.
- The alternative is lifting state into `MyApp` and passing it to every screen. It works, but every screen has to receive and pass it along, and screens are rendered by the navigator, not by you, which makes passing props awkward.

### Finished Code

- This is the completed example. Compare it against yours if something doesn't work and you can't find the typo.

```jsx
import { View, Button, Text, StyleSheet } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { atom, useAtom } from "jotai";

const Stack = createNativeStackNavigator();

// Global stock counts, shared by every screen.
const stockAtom = atom({ Apple: 5, Banana: 0, Cherry: 15 });

function HomeScreen({ navigation }) {
  const [stock] = useAtom(stockAtom);

  return (
    <View style={styles.screen}>
      {Object.keys(stock).map((item) => (
        <Button
          key={item}
          title={`${item} (${stock[item]})`}
          onPress={() => navigation.navigate("Details", { item })}
        />
      ))}
    </View>
  );
}

function DetailsScreen({ route }) {
  const [stock] = useAtom(stockAtom);
  const { item = "Apple" } = route.params ?? {};

  return (
    <View style={styles.screen}>
      <Text>
        {item}: {stock[item]} in stock
      </Text>
    </View>
  );
}

// Lives in the header, but reads and updates the same atom.
function BuyButton({ item }) {
  const [stock, setStock] = useAtom(stockAtom);
  const qty = stock[item] ?? 0;

  return (
    <Button
      title="Buy"
      disabled={qty === 0}
      onPress={() => setStock((prev) => ({ ...prev, [item]: prev[item] - 1 }))}
    />
  );
}

export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={({ route }) => ({
            title: route.params?.item ?? "Details",
            headerRight: () => <BuyButton item={route.params?.item} />,
          })}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  screen: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```
