# Lab 6 - Location and Maps

## A Note on AI

AI assistants are welcome in this course, but use them to get unstuck, not to do the work. Attempt every part yourself first. If you are stuck, paste **your** code and ask why it does not work or what concept you are missing, rather than asking for the answer. Never keep code you could not explain line by line to a classmate. If you cannot explain it, you cannot debug it or build on it, and it is not yours yet. The point of this lab is not the finished Snack. It is being able to ask for someone's location at the right moment, put it on a map, and stop using it when you are done.

## Overview

You build one app, **Park It**, that remembers where you left your car. Each part adds to it. Plan for about 45 minutes.

- Part 1 puts a map behind the starter's info card.
- Parts 2 and 3 save your spot with the GPS and look up its address.
- Part 4 tells you how far away the car is as you walk.
- Part 5 lets you fix the spot by hand and clear it when you are back.

Because every part builds on the one before, do them in order. Your submitted Snack shows the finished app, and it is marked against all five parts.

## Setup

1. Open the React Native Playground (Snack) at https://snack.expo.dev and sign in.
2. Rename the Snack to `WEB530 Lab 6 - Your Name` (click the name at the top left).
3. Keep Snack on SDK 54 (the default).
4. In the file list on the left, add a new file named `styles.js` in the same folder as `App.js`.
5. Replace the contents of `package.json`, `App.js` and `styles.js` with the three starter files below.
6. Click **Save** so the Snack gets a permanent link.

Starter for `package.json`:

```json
{
  "dependencies": {
    "react-native-paper": "4.9.2",
    "@expo/vector-icons": "^15.0.3",
    "expo-location": "~19.0.8",
    "react-native-maps": "1.20.1"
  }
}
```

Starter for `App.js`:

```jsx
import { View, Text, Pressable } from "react-native";
import styles from "./styles";

// Where the map starts: Newnham Campus.
const START = {
  latitude: 43.79457,
  longitude: -79.35043,
  latitudeDelta: 0.01,
  longitudeDelta: 0.01,
};

// Straight-line distance between two { latitude, longitude } points, in metres.
// You do not need to change this function. Part 4 uses it.
function distanceInMetres(a, b) {
  const R = 6371000;
  const toRad = (deg) => (deg * Math.PI) / 180;
  const dLat = toRad(b.latitude - a.latitude);
  const dLon = toRad(b.longitude - a.longitude);
  const h =
    Math.sin(dLat / 2) ** 2 +
    Math.cos(toRad(a.latitude)) *
      Math.cos(toRad(b.latitude)) *
      Math.sin(dLon / 2) ** 2;
  return 2 * R * Math.asin(Math.sqrt(h));
}

export default function MyApp() {
  return (
    <View style={styles.container}>
      <View style={styles.card}>
        <Text style={styles.cardHint}>
          Press Park Here when you leave your car.
        </Text>
        <View style={styles.buttonRow}>
          <Pressable
            style={({ pressed }) => [styles.button, pressed && styles.pressed]}
          >
            <Text style={styles.buttonText}>Park Here</Text>
          </Pressable>
          <Pressable
            style={({ pressed }) => [styles.button, pressed && styles.pressed]}
          >
            <Text style={styles.buttonText}>I'm Back</Text>
          </Pressable>
        </View>
      </View>
    </View>
  );
}
```

Starter for `styles.js`:

```js
import { StyleSheet } from "react-native";

export default StyleSheet.create({
  container: { flex: 1 },
  map: { flex: 1 },
  card: {
    position: "absolute",
    bottom: 40,
    left: 20,
    right: 20,
    padding: 16,
    borderRadius: 8,
    backgroundColor: "white",
  },
  cardTitle: { fontSize: 18, fontWeight: "600" },
  cardText: { fontSize: 14, color: "#666", marginTop: 4 },
  cardHint: { fontSize: 16, color: "#666", textAlign: "center" },
  error: { fontSize: 14, color: "red", marginBottom: 8 },
  buttonRow: { flexDirection: "row", gap: 10, marginTop: 12 },
  button: {
    flex: 1,
    alignItems: "center",
    padding: 12,
    borderRadius: 8,
    backgroundColor: "#2196F3",
  },
  pressed: { backgroundColor: "#1565C0" },
  disabled: { backgroundColor: "#BDBDBD" },
  buttonText: { color: "white", fontSize: 16 },
});
```

The card already floats over the bottom of the screen with `position: "absolute"`, as in the week 7 map examples. Every style you need is already in `styles.js`, so you can spend the lab on location and maps.

## Styling Rules

- Every style is in `styles.js`. There are **no** inline `style={{ ... }}` objects anywhere in `App.js`. A `Pressable` style function that returns an array of styles, like the starter's, is fine.
- You may add styles if you want to, but only in `styles.js`, and with names that describe what they are for.

## Running Your App

- This lab runs on a phone. Choose **My Device** and open the Snack in Expo Go. The **Web** preview cannot show a map: `react-native-maps` has no web version.
- If your phone will not connect, Snack's **Android** tab runs an emulator in the browser. It works for every part, but its location is fixed in Oshawa and never moves, so Part 4 can only show a change when you move the car.
- Expo Go remembers your answer to the permission dialog for Expo Go, not for each Snack. If you allowed location for the week 7 examples, you will not see the dialog again. Part 2 tells you how to test the "no" answer anyway.

## Submission

- Submit **one** Snack link to the `Lab 6` assessment in Blackboard. After you save, the link looks like `https://snack.expo.dev/@yourname/web530-lab-6-...`.
- The lab is due at the end of class. Submit before you leave.
- Only students who participate in class and hand in a lab slip can receive marks for this lab. If you miss the class, you are still encouraged to complete it as practice.
- Before submitting, save one last time and open your link in a private browser window to confirm it loads without signing in.

## Part 1 - The Map (1 point)

Put a map behind the card.

- The map fills the whole screen and starts at `START`.
- The card and its buttons sit on top of the map.
- The map's own places (restaurants, shops, schools) are hidden on **both** iOS and Android, so the only pin on the map will be your car.

Requirements:

- Add a comment above the props that hide the places explaining, in your own words, why it takes two props to do it.

Hint: The map needs a size, or it renders as nothing. Whatever comes later in the JSX draws on top.

## Part 2 - Park Here (1 point)

Pressing `Park Here` saves where you are standing.

- The app asks for location permission when the user presses `Park Here`, **not** when the app opens. Add a comment above your function explaining, in your own words, why that is the better moment to ask.
- If the user says no, the card shows a message in the `error` style that says what the app needs and what the user can do instead. Suggest long-pressing the map, which you add in Part 5. The map keeps working.
- If the user says yes:
  - The blue dot for the user appears on the map.
  - The app gets the current position and drops a `Marker` there with the title `My car` and the description `Saved at` and the time, for example `Saved at 7:44:50 PM`.
  - The map glides to the car and zooms in, with both deltas at `0.005`.
- While the app is finding you, the button reads `Finding you...`, is disabled, and uses the `disabled` style. It goes back to `Park Here` when it is done, whether it worked or not.
- If getting the position fails, for example because location is turned off on the phone, the card shows `Could not get your location. Is location turned on?`.
- Pressing `Park Here` again moves the car to where you are now. There is only ever one car marker, and an old error message disappears.

Requirements:

- One piece of state, `spot`, holds the car as `{ latitude, longitude, time }`, where `time` is `new Date()`. It starts as `null`.
- Use `try` / `catch` around the position request, and `animateToRegion` on a ref to move the map.
- Test the "no" answer. On Android, choose **Don't allow** the first time the dialog appears. If you already allowed location, temporarily change `"granted"` in your permission check to `"nope"`, so the app acts as though the user said no. Put it back afterwards.

Hint: This is the Your First Map example from the week 7 notes, with the permission request moved into the button. `showsUserLocation` only works on Android once permission has been granted, so it needs its own state.

## Part 3 - The Address (1 point)

Coordinates mean nothing to someone looking for their car. Show the address in the card.

- When a spot is saved, the card's first line, in the `cardTitle` style, is the street number, street and city, for example `915 Grandview Street North, Oshawa`. The second line, in the `cardText` style, is `Saved at` and the time.
- While the address is being looked up, the first line reads `Looking up the address...`. It must **never** show the address of the previous spot.
- If the lookup fails, or the place it finds has no street, show the coordinates to five decimal places instead, for example `43.92280, -78.86930`.
- When there is no spot, the card shows the starter's hint, as it does now.

Requirements:

- Look up the address in a `useEffect` with `[spot]` as its dependency array, **not** inside your `Park Here` function. Part 5 changes the spot a second way, and the effect covers both.
- Use a `cancelled` flag, as in the Live Location example, so a lookup that finishes after the spot has moved is ignored. Add a comment above it explaining, in your own words, what would go wrong without it.

Hint: `reverseGeocodeAsync` returns an array, best match first. `streetNumber` can be `null` even when `street` is not.

## Part 4 - How Far Is the Car? (1 point)

While a spot is saved, the card tells you how far away the car is, and keeps telling you as you walk.

- The card's third line reads `Your car is 230 m away.`, with the distance rounded to whole metres. Under 20 metres, it reads `Your car is right here.` instead.
- Until the first position arrives, there is no third line.
- The app only watches your position while a spot is saved. With no spot, it does not watch at all.

Requirements:

- Use `watchPositionAsync` in a `useEffect`, with a clean-up function that removes the subscription and the `cancelled` flag from the Live Location example.
- Keep the latest position from the watch in state. Work out the distance with the starter's `distanceInMetres` above the `return`, on every render. Do **not** store the distance in state.
- Make a boolean, `const parked = spot !== null;`, and use `[parked]` as the dependency array. Add a comment above the effect explaining, in your own words, why `[spot]` would be the wrong choice here.

Walk around the room with your phone and watch the number change. GPS indoors is only accurate to a few metres, so it will not count every step.

## Part 5 - Fix the Spot and Go Home (1 point)

The GPS is not always right, and you will not always have it. Let the user fix the spot by hand, and clear it when they are back.

- Long-pressing the map moves the car there. The marker moves, the saved time updates, and the address and distance update to match.
- Long-pressing works even when location permission was denied, so the advice in your Part 2 message is true. Without permission the address may fall back to coordinates, and there is no distance, but the marker still moves.
- Pressing `I'm Back` removes the marker, puts the hint back in the card, and stops watching the position.
- `I'm Back` is disabled, and uses the `disabled` style, whenever no spot is saved.

Requirements:

- Use the map's `onLongPress` prop. The pressed coordinate is in `e.nativeEvent.coordinate`, the same as `onPress` in the Drawing on the Map example.
- If your Parts 3 and 4 effects are right, the address, distance and watch need no new code for this part.
