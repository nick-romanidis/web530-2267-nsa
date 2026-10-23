# Week 7.1 - Notes and Examples

## React Native Playground

React Native Playground (Snack)
_Use this for the examples. Modify App.js_
(https://snack.expo.dev)

- Keep Snack on SDK 54 (the default).
- Every example this week needs two packages: `expo-location` for the device's position and `react-native-maps` for the map. Replace the contents of `package.json` before the first example.

_package.json_

```json
{
  "dependencies": {
    "expo-location": "~19.0.8",
    "react-native-maps": "1.20.1"
  }
}
```

- This week runs on a phone. Choose **My Device** and open the Snack in Expo Go. The location examples also run on **Web**, where the browser asks for your location, but a laptop's position comes from Wi-Fi and is not very accurate. The map examples do not run on Web at all: `react-native-maps` has no web version.
- The first time an example asks, Expo Go shows the phone's permission dialog. Your answer is remembered for Expo Go, not for each Snack. To see the dialog again after you allow or deny it, change the location permission for Expo Go in the phone's Settings.

## Why Location?

- Many apps start from where the user is: nearby restaurants, a ride pickup, the weather, a running route, a store's opening hours.
- The phone finds its position with GPS, Wi-Fi and cell towers. The app gets back coordinates: a latitude and a longitude.
- Location is private. It reveals where someone lives, works and goes every day, so the operating system makes the app ask, and the user can say no. The app has to keep working if they do.
- Location calls are asynchronous, like `fetch` in Week 5. They return promises, so they run inside `useEffect` and their results go into state.
- There are two ways to ask: once (where am I now?) or continuously (tell me every time I move). The first two examples show one each.

## Example: Where Am I?

- The simplest location feature: ask once, show the coordinates, and turn them into an address.

### Starter Code

- Replace everything in `App.js`. The screen says it is finding you and spins forever, because nothing asks for the location yet.

```jsx
import { View, Text, ActivityIndicator, StyleSheet } from "react-native";

export default function MyApp() {
  return (
    <View style={styles.center}>
      <Text style={styles.title}>Where Am I?</Text>
      <ActivityIndicator size="large" />
      <Text style={styles.message}>Finding you...</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  center: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    padding: 20,
  },
  title: { fontSize: 24, fontWeight: "600", marginBottom: 20 },
  message: { fontSize: 16, color: "#666", marginTop: 10 },
  error: { fontSize: 16, color: "red", textAlign: "center" },
  value: { fontSize: 18, marginBottom: 6 },
});
```

### Step 1 - Ask for Permission

- Add the hooks and `expo-location` above the `react-native` import.

```jsx
import { useState, useEffect } from "react";
import * as Location from "expo-location";
```

- `import * as Location` puts every export of the package on one `Location` object: `Location.requestForegroundPermissionsAsync`, `Location.getCurrentPositionAsync`, and so on.
- Add an `errorMsg` state at the top of `MyApp`. `null` means no error.

```jsx
const [errorMsg, setErrorMsg] = useState(null);
```

- Add an effect under it that asks for permission when the screen opens.

```jsx
useEffect(() => {
  async function getLocation() {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== "granted") {
      setErrorMsg("Permission to access location was denied.");
      return;
    }
  }

  getLocation();
}, []);
```

- Every function in `expo-location` returns a promise. `await` waits for a promise and gives back its result, and it can only be used inside an `async` function. It does the same job as `.then()` from Week 5, but the code reads top to bottom.
- The effect function itself cannot be `async`, because an effect may only return a clean-up function, and an `async` function always returns a promise. So the effect defines an `async` function and calls it.
- Foreground permission means location while the app is open, which is all this week needs. Background location, while the app is closed, needs extra setup and app store review.
- Above the main `return`, return an error screen when permission was denied.

```jsx
if (errorMsg) {
  return (
    <View style={styles.center}>
      <Text style={styles.error}>{errorMsg}</Text>
    </View>
  );
}
```

- Run it on your phone. The permission dialog appears. Deny it, and the error replaces the spinner. A real app would explain what it needs location for and keep the rest of the app working.
- Allow it (in Settings, if you denied it), and the screen still spins. Permission only opens the door. Nothing has asked for the position yet.

### Step 2 - Get the Position

- Add a `location` state under `errorMsg`.

```jsx
const [location, setLocation] = useState(null);
```

- Inside `getLocation`, after the permission check, ask for the position and store its coordinates. The `try` / `catch` handles a phone with location services turned off.

```jsx
try {
  const pos = await Location.getCurrentPositionAsync({});
  setLocation(pos.coords);
} catch {
  setErrorMsg("Could not get your location. Is location turned on?");
}
```

- `getCurrentPositionAsync` takes an options object. `{}` keeps the defaults.
- `pos` has a `timestamp` and a `coords` object with `latitude`, `longitude`, `accuracy`, `altitude`, `speed` and `heading`. Some of those can be `null`, depending on the device.
- `try` / `catch` is how `await` code handles a rejected promise. It is the `.catch()` from Week 5.
- Under the error screen, return the spinner screen until the location arrives.

```jsx
if (!location) {
  return (
    <View style={styles.center}>
      <ActivityIndicator size="large" />
      <Text style={styles.message}>Finding you...</Text>
    </View>
  );
}
```

- Replace the main `return` with the coordinates.

```jsx
return (
  <View style={styles.center}>
    <Text style={styles.title}>Where Am I?</Text>
    <Text style={styles.value}>Latitude: {location.latitude.toFixed(5)}</Text>
    <Text style={styles.value}>Longitude: {location.longitude.toFixed(5)}</Text>
    <Text style={styles.message}>
      Accurate to {Math.round(location.accuracy)} metres
    </Text>
  </View>
);
```

- Run it. After a moment the spinner is replaced by your coordinates. At Newnham Campus they are about 43.79 and -79.35.
- Latitude is north-south, from -90 at the South Pole to 90 at the North Pole. Longitude is east-west, from -180 to 180. West of Greenwich is negative, which is why Toronto's longitude is.
- `toFixed(5)` keeps five decimal places, about one metre. The raw numbers have more digits than the GPS can measure.
- `accuracy` is a radius in metres: the phone is somewhere inside that circle. Compare it indoors and next to a window. Run it on **Web** too: the laptop's Wi-Fi position is usually much less accurate.

### Step 3 - Find the Address

- Coordinates mean nothing to a user. `reverseGeocodeAsync` turns them into an address. Add an `address` state under `location`.

```jsx
const [address, setAddress] = useState(null);
```

- Inside the `try`, after `setLocation`, look up the address.

```jsx
const places = await Location.reverseGeocodeAsync(pos.coords);
setAddress(places[0]);
```

- It returns an array of possible matches, best first, so the code keeps the first one.
- Show it under the accuracy line. The address arrives after the coordinates, so check it first.

```jsx
{address && (
  <Text style={styles.value}>
    {address.city}, {address.region}, {address.country}
  </Text>
)}
```

- An address has `name`, `street`, `streetNumber`, `city`, `region`, `postalCode` and `country`. Any of them can be `null`, so pick the ones that make sense for the app.
- Run it on your phone. The city, province and country appear under the coordinates. Reverse geocoding uses the phone's built-in service and does not work on **Web**: there it fails and the `catch` shows the error.

### Finished Code

- Compare against this if something is not working.

```jsx
import { useState, useEffect } from "react";
import * as Location from "expo-location";
import { View, Text, ActivityIndicator, StyleSheet } from "react-native";

export default function MyApp() {
  const [errorMsg, setErrorMsg] = useState(null);
  const [location, setLocation] = useState(null);
  const [address, setAddress] = useState(null);

  useEffect(() => {
    async function getLocation() {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== "granted") {
        setErrorMsg("Permission to access location was denied.");
        return;
      }

      try {
        const pos = await Location.getCurrentPositionAsync({});
        setLocation(pos.coords);

        const places = await Location.reverseGeocodeAsync(pos.coords);
        setAddress(places[0]);
      } catch {
        setErrorMsg("Could not get your location. Is location turned on?");
      }
    }

    getLocation();
  }, []);

  if (errorMsg) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{errorMsg}</Text>
      </View>
    );
  }

  if (!location) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
        <Text style={styles.message}>Finding you...</Text>
      </View>
    );
  }

  return (
    <View style={styles.center}>
      <Text style={styles.title}>Where Am I?</Text>
      <Text style={styles.value}>Latitude: {location.latitude.toFixed(5)}</Text>
      <Text style={styles.value}>Longitude: {location.longitude.toFixed(5)}</Text>
      <Text style={styles.message}>
        Accurate to {Math.round(location.accuracy)} metres
      </Text>
      {address && (
        <Text style={styles.value}>
          {address.city}, {address.region}, {address.country}
        </Text>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  center: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    padding: 20,
  },
  title: { fontSize: 24, fontWeight: "600", marginBottom: 20 },
  message: { fontSize: 16, color: "#666", marginTop: 10 },
  error: { fontSize: 16, color: "red", textAlign: "center" },
  value: { fontSize: 18, marginBottom: 6 },
});
```

## Example: Live Location

- A running app, a delivery tracker or turn-by-turn directions need the position every time it changes, not once.
- `watchPositionAsync` keeps calling a function with each new position until the app tells it to stop. Stopping matters: a watcher left running keeps the GPS on and drains the battery.
- Start from the finished Where Am I? code.

### Step 1 - Watch Instead of Get

- An address per update would be too many lookups, so take the address out: delete the `address` state, the two `reverseGeocodeAsync` lines and the `{address && ...}` block.
- Change the title in the main `return` to `Live Location`.
- Replace the whole effect.

```jsx
useEffect(() => {
  let subscription;
  let cancelled = false;

  async function startWatching() {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== "granted") {
      setErrorMsg("Permission to access location was denied.");
      return;
    }

    try {
      subscription = await Location.watchPositionAsync(
        {
          accuracy: Location.Accuracy.High,
          timeInterval: 1000, // at most once a second
          distanceInterval: 1, // and only after moving a metre
        },
        (pos) => setLocation(pos.coords)
      );
    } catch {
      setErrorMsg("Could not get your location. Is location turned on?");
      return;
    }

    // The screen may have closed while we were waiting.
    if (cancelled) subscription.remove();
  }

  startWatching();

  return () => {
    cancelled = true;
    if (subscription) subscription.remove();
  };
}, []);
```

- The second argument to `watchPositionAsync` is the callback. It runs once for each new position, for as long as the watch lasts.
- The options trade accuracy for battery. `Accuracy.High` uses GPS. The two intervals stop the callback from running more than the app needs.
- `watchPositionAsync` resolves to a subscription, and `subscription.remove()` stops the watch. The clean-up function calls it when the screen closes, the same clean-up pattern as the timer in Week 2.
- The `cancelled` flag covers a race. The clean-up can run while the permission dialog is still open, before there is a subscription to remove. When the watch starts after that, the flag says the screen is gone, so it is removed straight away.
- Run it on your phone and walk around the room. The numbers change as you move. Standing still, they stop: `distanceInterval` skips updates under a metre.

### Step 2 - Count the Updates

- Add an `updates` state under `location`.

```jsx
const [updates, setUpdates] = useState(0);
```

- Make the callback count each update as well. The count depends on the previous count, so use the updater form.

```jsx
(pos) => {
  setLocation(pos.coords);
  setUpdates((prev) => prev + 1);
}
```

- Show the count under the accuracy line.

```jsx
<Text style={styles.message}>Updates: {updates}</Text>
```

- Run it. The count proves the watch is live. Walk, and it climbs about once a second. Stand still, and it stops.

### Step 3 - Pause and Resume

- Tracking should stop when the user does not need it. Add `Button` to the `react-native` import, and a `tracking` state under `updates`.

```jsx
const [tracking, setTracking] = useState(true);
```

- In the effect, add a check as the first line, and change the dependency array to `[tracking]`.

```jsx
useEffect(() => {
  if (!tracking) return;

  let subscription;
  // ...the rest of the effect is unchanged
}, [tracking]);
```

- Add the button under the updates count.

```jsx
<Button
  title={tracking ? "Pause" : "Resume"}
  onPress={() => setTracking((prev) => !prev)}
/>
```

- Pressing **Pause** changes `tracking`, so React runs the effect's clean-up, which removes the watch. Then it runs the effect again, which returns at the first line without starting a new one.
- Pressing **Resume** starts a new watch. The permission request runs again, but it answers straight away without a dialog, because the user already said yes.
- Run it. Pause, walk, and the count stays still. Resume, and it goes up by one straight away, because a new watch always reports the current position first. Then it climbs again as you move.

### Finished Code

```jsx
import { useState, useEffect } from "react";
import * as Location from "expo-location";
import {
  View,
  Text,
  Button,
  ActivityIndicator,
  StyleSheet,
} from "react-native";

export default function MyApp() {
  const [errorMsg, setErrorMsg] = useState(null);
  const [location, setLocation] = useState(null);
  const [updates, setUpdates] = useState(0);
  const [tracking, setTracking] = useState(true);

  useEffect(() => {
    if (!tracking) return;

    let subscription;
    let cancelled = false;

    async function startWatching() {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== "granted") {
        setErrorMsg("Permission to access location was denied.");
        return;
      }

      try {
        subscription = await Location.watchPositionAsync(
          {
            accuracy: Location.Accuracy.High,
            timeInterval: 1000, // at most once a second
            distanceInterval: 1, // and only after moving a metre
          },
          (pos) => {
            setLocation(pos.coords);
            setUpdates((prev) => prev + 1);
          }
        );
      } catch {
        setErrorMsg("Could not get your location. Is location turned on?");
        return;
      }

      // The screen may have closed while we were waiting.
      if (cancelled) subscription.remove();
    }

    startWatching();

    return () => {
      cancelled = true;
      if (subscription) subscription.remove();
    };
  }, [tracking]);

  if (errorMsg) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{errorMsg}</Text>
      </View>
    );
  }

  if (!location) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
        <Text style={styles.message}>Finding you...</Text>
      </View>
    );
  }

  return (
    <View style={styles.center}>
      <Text style={styles.title}>Live Location</Text>
      <Text style={styles.value}>Latitude: {location.latitude.toFixed(5)}</Text>
      <Text style={styles.value}>Longitude: {location.longitude.toFixed(5)}</Text>
      <Text style={styles.message}>
        Accurate to {Math.round(location.accuracy)} metres
      </Text>
      <Text style={styles.message}>Updates: {updates}</Text>
      <Button
        title={tracking ? "Pause" : "Resume"}
        onPress={() => setTracking((prev) => !prev)}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  center: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    padding: 20,
  },
  title: { fontSize: 24, fontWeight: "600", marginBottom: 20 },
  message: { fontSize: 16, color: "#666", marginTop: 10 },
  error: { fontSize: 16, color: "red", textAlign: "center" },
  value: { fontSize: 18, marginBottom: 6 },
});
```

## Maps

- Coordinates on a screen are hard to read. A map shows them: where the user is, where the places are, and how they relate.
- `react-native-maps` provides the `MapView` component. It is not a picture of a map. It wraps the phone's own map, so it pans, zooms and rotates like the Maps app.
- That means it is a different map on each platform. In Expo Go, iOS draws Apple Maps and Android draws Google Maps. Most props work on both, but some are one platform only. The examples point those out.
- Expo Go needs no API key for the map. A published Android app needs a Google Maps key, which is a build setting, not code.
- Choose **My Device** for every map example. The **Web** preview shows an error, because there is no web version.

## Example: Your First Map

### Starter Code

- Replace everything in `App.js`. It has a **Find Me** button over an empty screen, and the region for Newnham Campus, which the steps use.

```jsx
import { View, Text, Pressable, StyleSheet } from "react-native";

const NEWNHAM = {
  latitude: 43.79457,
  longitude: -79.35043,
  latitudeDelta: 0.01,
  longitudeDelta: 0.01,
};

export default function MyApp() {
  return (
    <View style={styles.container}>
      <Pressable
        style={({ pressed }) => [styles.button, pressed && styles.pressed]}
      >
        <Text style={styles.buttonText}>Find Me</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  map: { flex: 1 },
  button: {
    position: "absolute",
    bottom: 40,
    alignSelf: "center",
    padding: 16,
    borderRadius: 8,
    backgroundColor: "#2196F3",
  },
  pressed: { backgroundColor: "#1565C0" },
  buttonText: { color: "white", fontSize: 18 },
});
```

- `position: "absolute"` takes the button out of the flexbox layout and places it 40 points above the bottom edge, on top of whatever else is there. `alignSelf: "center"` centres it across.
- The button is the `Pressable` from Week 6, with its pressed style. It does nothing yet.

### Step 1 - Show a Map

- Import `MapView` under the `react-native` import. It is the package's default export, so it has no braces.

```jsx
import MapView from "react-native-maps";
```

- Add the map as the first child of the container, above the button.

```jsx
<MapView style={styles.map} />
```

- A map has no size of its own. `flex: 1` makes it fill the screen. Without a size it renders as nothing.
- The button comes after the map in the JSX, so it draws on top of it.
- Run it on your phone. Drag to pan, pinch to zoom, and twist with two fingers to rotate. All of that is built in. The starting view is wherever the platform decides, often a whole continent.

### Step 2 - Start on Campus

- Give the map a starting region.

```jsx
<MapView style={styles.map} initialRegion={NEWNHAM} />
```

- A region is a centre point and a span. `latitudeDelta` and `longitudeDelta` are how many degrees of latitude and longitude fit on the screen. 0.01 degrees of latitude is about a kilometre.
- A smaller delta zooms in. Change both deltas to `0.1` and run it: you see most of North York. Change them to `0.002`: you see one building. Change them back to `0.01`.
- `initialRegion` only sets where the map starts. After that the user moves the map, and changing the prop does nothing. To move the map later, the code has to tell it to, which is Step 4.

### Step 3 - Show the Blue Dot

- `showsUserLocation` draws the user as a blue dot, but on Android it fails silently unless the app already has location permission. So ask first with `expo-location`, as in the first example.
- Add the imports.

```jsx
import { useState, useEffect } from "react";
import * as Location from "expo-location";
```

- Add a state and an effect at the top of `MyApp`.

```jsx
const [canShowUser, setCanShowUser] = useState(false);

useEffect(() => {
  async function askPermission() {
    const { status } = await Location.requestForegroundPermissionsAsync();
    setCanShowUser(status === "granted");
  }

  askPermission();
}, []);
```

- Pass the result to the map.

```jsx
<MapView
  style={styles.map}
  initialRegion={NEWNHAM}
  showsUserLocation={canShowUser}
/>
```

- Run it. If you are on campus, the blue dot is on the map. If not, pan or zoom out to find it. The pale circle around it is the accuracy radius from the first example.
- `MapView` also has a `followsUserLocation` prop that keeps the map centred on the user, but it is Apple Maps only. It does nothing on Android. The next step works on both.

### Step 4 - Find Me

- To move the map from code, the code needs the map itself. Add `useRef` to the React import.

```jsx
import { useState, useEffect, useRef } from "react";
```

- Create a ref under the state.

```jsx
const mapRef = useRef(null);
```

- Give the map the ref.

```jsx
<MapView
  ref={mapRef}
  style={styles.map}
  initialRegion={NEWNHAM}
  showsUserLocation={canShowUser}
/>
```

- Week 6 used `useRef` to keep an animated value. A ref passed as the `ref` prop works the other way: React puts the component into `mapRef.current` after it renders. The map component has methods, and `animateToRegion` moves the map.
- Add a function under the effect that finds the user and moves the map there.

```jsx
async function findMe() {
  const pos = await Location.getCurrentPositionAsync({});
  mapRef.current.animateToRegion(
    {
      latitude: pos.coords.latitude,
      longitude: pos.coords.longitude,
      latitudeDelta: 0.005,
      longitudeDelta: 0.005,
    },
    1000
  );
}
```

- The second argument is how long the animation takes, in milliseconds. iOS ignores it and uses its own speed.
- Wire up the button. `disabled` stops it from working until permission is granted.

```jsx
<Pressable
  style={({ pressed }) => [styles.button, pressed && styles.pressed]}
  onPress={findMe}
  disabled={!canShowUser}
>
```

- Run it. Pan far away, press **Find Me**, and the map glides back to the blue dot and zooms in. That is the "recentre" button every map app has, on both platforms.

### Finished Code

```jsx
import { useState, useEffect, useRef } from "react";
import * as Location from "expo-location";
import { View, Text, Pressable, StyleSheet } from "react-native";
import MapView from "react-native-maps";

const NEWNHAM = {
  latitude: 43.79457,
  longitude: -79.35043,
  latitudeDelta: 0.01,
  longitudeDelta: 0.01,
};

export default function MyApp() {
  const [canShowUser, setCanShowUser] = useState(false);
  const mapRef = useRef(null);

  useEffect(() => {
    async function askPermission() {
      const { status } = await Location.requestForegroundPermissionsAsync();
      setCanShowUser(status === "granted");
    }

    askPermission();
  }, []);

  async function findMe() {
    const pos = await Location.getCurrentPositionAsync({});
    mapRef.current.animateToRegion(
      {
        latitude: pos.coords.latitude,
        longitude: pos.coords.longitude,
        latitudeDelta: 0.005,
        longitudeDelta: 0.005,
      },
      1000
    );
  }

  return (
    <View style={styles.container}>
      <MapView
        ref={mapRef}
        style={styles.map}
        initialRegion={NEWNHAM}
        showsUserLocation={canShowUser}
      />
      <Pressable
        style={({ pressed }) => [styles.button, pressed && styles.pressed]}
        onPress={findMe}
        disabled={!canShowUser}
      >
        <Text style={styles.buttonText}>Find Me</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  map: { flex: 1 },
  button: {
    position: "absolute",
    bottom: 40,
    alignSelf: "center",
    padding: 16,
    borderRadius: 8,
    backgroundColor: "#2196F3",
  },
  pressed: { backgroundColor: "#1565C0" },
  buttonText: { color: "white", fontSize: 18 },
});
```

## Example: Campus Markers

- A marker is a pin at a coordinate. It is how a map says "this place matters": a store, a pickup point, a search result.
- Markers are components inside the `MapView`, so a list of places becomes a list of markers with `.map()`, the same way a list became rows in Week 5.

### Starter Code

- Replace everything in `App.js`. It has the Seneca campuses as data, a map that fits them all, and an info card at the bottom that the steps fill in.

```jsx
import { View, Text, StyleSheet } from "react-native";
import MapView from "react-native-maps";

const CAMPUSES = [
  {
    id: "newnham",
    name: "Newnham Campus",
    address: "1750 Finch Ave E, Toronto",
    latitude: 43.79457,
    longitude: -79.35043,
  },
  {
    id: "york",
    name: "Seneca@York",
    address: "70 The Pond Rd, Toronto",
    latitude: 43.77151,
    longitude: -79.49892,
  },
  {
    id: "markham",
    name: "Markham Campus",
    address: "8 The Seneca Way, Markham",
    latitude: 43.8495,
    longitude: -79.36649,
  },
  {
    id: "king",
    name: "King Campus",
    address: "13990 Dufferin St, King City",
    latitude: 43.95489,
    longitude: -79.51858,
  },
];

const GTA_NORTH = {
  latitude: 43.865,
  longitude: -79.435,
  latitudeDelta: 0.3,
  longitudeDelta: 0.3,
};

export default function MyApp() {
  return (
    <View style={styles.container}>
      <MapView style={styles.map} initialRegion={GTA_NORTH} />
      <View style={styles.card}>
        <Text style={styles.cardHint}>Tap a campus.</Text>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
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
});
```

- The region's centre sits between the campuses, and a delta of 0.3 is wide enough to fit them all.
- The card uses `position: "absolute"` with `left` and `right`, so it stretches across the bottom of the map with a 20-point margin.

### Step 1 - Add One Marker

- `Marker` is a named export of `react-native-maps`. Change the import.

```jsx
import MapView, { Marker } from "react-native-maps";
```

- Give the map a child. `MapView` is no longer self-closing.

```jsx
<MapView style={styles.map} initialRegion={GTA_NORTH}>
  <Marker
    coordinate={{ latitude: 43.79457, longitude: -79.35043 }}
    title="Newnham Campus"
    description="1750 Finch Ave E, Toronto"
  />
</MapView>
```

- `coordinate` is the only required prop: an object with `latitude` and `longitude`.
- Run it, and tap the pin. A bubble (a callout) shows the `title` and `description`. On Android, Google Maps also shows two buttons for directions and opening the place in the Maps app.

### Step 2 - A Marker for Every Campus

- Replace the single marker with one marker per campus.

```jsx
<MapView style={styles.map} initialRegion={GTA_NORTH}>
  {CAMPUSES.map((campus) => (
    <Marker
      key={campus.id}
      coordinate={{ latitude: campus.latitude, longitude: campus.longitude }}
      title={campus.name}
      description={campus.address}
    />
  ))}
</MapView>
```

- Each marker needs a `key`, like every list in Week 5.
- Run it. Four pins, one per campus. Tap each one. Add a fifth object to `CAMPUSES` with any coordinates you like and it appears without touching the JSX.

### Step 3 - Show the Selected Campus

- A callout is small. Most apps show details in their own panel instead. Add `useState` above the `react-native` import, and a `selected` state at the top of `MyApp`.

```jsx
import { useState } from "react";
```

```jsx
const [selected, setSelected] = useState(null);
```

- Give each marker an `onPress` that selects its campus.

```jsx
<Marker
  key={campus.id}
  coordinate={{ latitude: campus.latitude, longitude: campus.longitude }}
  title={campus.name}
  description={campus.address}
  onPress={() => setSelected(campus)}
/>
```

- Replace the hint in the card with the selected campus, or the hint when nothing is selected.

```jsx
<View style={styles.card}>
  {selected ? (
    <>
      <Text style={styles.cardTitle}>{selected.name}</Text>
      <Text style={styles.cardText}>{selected.address}</Text>
    </>
  ) : (
    <Text style={styles.cardHint}>Tap a campus.</Text>
  )}
</View>
```

- `selected` holds the whole campus object, so the card can show any of its fields.
- Run it. Tap a pin and the card shows its name and address. Tap another and the card changes. On Android the map also slides to centre the pin you tapped.
- `Marker` has a `pinColor` prop, but set it once. Changing it on a marker that is already drawn does not update on Android, so a different colour for the selected pin is not reliable. The card is the selection feedback.

### Step 4 - Hide Other Places

- The base map is full of restaurants, shops and bus stops. They compete with your markers. Hiding them is different on each platform.
- Add a map style above `MyApp`. It is Google Maps' styling format: a list of rules, each picking a kind of feature and changing how it looks.

```jsx
const HIDE_PLACES = [{ featureType: "poi", stylers: [{ visibility: "off" }] }];
```

- Give the map both props.

```jsx
<MapView
  style={styles.map}
  initialRegion={GTA_NORTH}
  showsPointsOfInterest={false}
  customMapStyle={HIDE_PLACES}
>
```

- `poi` means point of interest: any business or landmark the map labels on its own.
- `showsPointsOfInterest={false}` hides them on iOS (Apple Maps) and is ignored on Android. `customMapStyle` hides them on Android (Google Maps) and is ignored by Apple Maps. Together they work on both.
- Run it and zoom in on a campus. Only roads, buildings and your markers are left.

### Finished Code

```jsx
import { useState } from "react";
import { View, Text, StyleSheet } from "react-native";
import MapView, { Marker } from "react-native-maps";

const CAMPUSES = [
  {
    id: "newnham",
    name: "Newnham Campus",
    address: "1750 Finch Ave E, Toronto",
    latitude: 43.79457,
    longitude: -79.35043,
  },
  {
    id: "york",
    name: "Seneca@York",
    address: "70 The Pond Rd, Toronto",
    latitude: 43.77151,
    longitude: -79.49892,
  },
  {
    id: "markham",
    name: "Markham Campus",
    address: "8 The Seneca Way, Markham",
    latitude: 43.8495,
    longitude: -79.36649,
  },
  {
    id: "king",
    name: "King Campus",
    address: "13990 Dufferin St, King City",
    latitude: 43.95489,
    longitude: -79.51858,
  },
];

const GTA_NORTH = {
  latitude: 43.865,
  longitude: -79.435,
  latitudeDelta: 0.3,
  longitudeDelta: 0.3,
};

const HIDE_PLACES = [{ featureType: "poi", stylers: [{ visibility: "off" }] }];

export default function MyApp() {
  const [selected, setSelected] = useState(null);

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={GTA_NORTH}
        showsPointsOfInterest={false}
        customMapStyle={HIDE_PLACES}
      >
        {CAMPUSES.map((campus) => (
          <Marker
            key={campus.id}
            coordinate={{ latitude: campus.latitude, longitude: campus.longitude }}
            title={campus.name}
            description={campus.address}
            onPress={() => setSelected(campus)}
          />
        ))}
      </MapView>
      <View style={styles.card}>
        {selected ? (
          <>
            <Text style={styles.cardTitle}>{selected.name}</Text>
            <Text style={styles.cardText}>{selected.address}</Text>
          </>
        ) : (
          <Text style={styles.cardHint}>Tap a campus.</Text>
        )}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
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
});
```

## Example: Drawing on the Map

- Markers show points. Shapes show areas: a campus boundary, a delivery zone, a neighbourhood. `Polygon` draws a closed shape through a list of coordinates.
- The map also reports where the user taps, which is how apps let users drop pins or draw their own areas.

### Starter Code

- Replace everything in `App.js`. It is a map of Newnham Campus with a **Clear** button that the last step uses.

```jsx
import { View, Text, Pressable, StyleSheet } from "react-native";
import MapView from "react-native-maps";

const CAMPUS = {
  latitude: 43.79607,
  longitude: -79.3487,
  latitudeDelta: 0.01,
  longitudeDelta: 0.01,
};

export default function MyApp() {
  return (
    <View style={styles.container}>
      <MapView style={styles.map} initialRegion={CAMPUS} />
      <Pressable
        style={({ pressed }) => [styles.button, pressed && styles.pressed]}
      >
        <Text style={styles.buttonText}>Clear</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  map: { flex: 1 },
  button: {
    position: "absolute",
    bottom: 40,
    alignSelf: "center",
    padding: 16,
    borderRadius: 8,
    backgroundColor: "#2196F3",
  },
  pressed: { backgroundColor: "#1565C0" },
  buttonText: { color: "white", fontSize: 18 },
});
```

### Step 1 - Outline the Campus

- Add `Polygon` to the map import.

```jsx
import MapView, { Polygon } from "react-native-maps";
```

- Add the campus corners above `MyApp`.

```jsx
const CAMPUS_OUTLINE = [
  { latitude: 43.79701, longitude: -79.35178 },
  { latitude: 43.79849, longitude: -79.3453 },
  { latitude: 43.79483, longitude: -79.34612 },
  { latitude: 43.79382, longitude: -79.35049 },
];
```

- Draw it inside the map.

```jsx
<MapView style={styles.map} initialRegion={CAMPUS}>
  <Polygon coordinates={CAMPUS_OUTLINE} strokeWidth={3} strokeColor="red" />
</MapView>
```

- The polygon joins the points in order and closes the shape back to the first one. Four points make a four-sided shape. The order matters: swap two of them and the lines cross.
- Run it. A red outline sits around the campus, and it stays in place as you pan and zoom, because it is drawn in map coordinates, not screen points.

### Step 2 - Fill It In

- Add a fill.

```jsx
<Polygon
  coordinates={CAMPUS_OUTLINE}
  strokeWidth={3}
  strokeColor="red"
  fillColor="rgba(255, 0, 0, 0.2)"
/>
```

- `rgba` is red, green, blue and alpha. The alpha, 0.2, makes the fill 20% opaque, so the map shows through. A solid fill would hide what is underneath.
- Run it. The campus is shaded, and the roads and buildings are still readable.

### Step 3 - Draw Your Own Shape

- Add `useState` above the `react-native` import, and `Marker` to the map import.

```jsx
import { useState } from "react";
```

```jsx
import MapView, { Marker, Polygon } from "react-native-maps";
```

- Add a `points` state at the top of `MyApp`.

```jsx
const [points, setPoints] = useState([]);
```

- Give the map an `onPress`. The tapped coordinate is in `e.nativeEvent.coordinate`. Adding it depends on the previous list, so use the updater form.

```jsx
<MapView
  style={styles.map}
  initialRegion={CAMPUS}
  onPress={(e) => {
    const point = e.nativeEvent.coordinate;
    setPoints((prev) => [...prev, point]);
  }}
>
```

- The coordinate is read into `point` first, so the updater only needs a plain value when React runs it later.
- Inside the map, under the campus polygon, show a marker for each point, and the shape once there are enough points for one.

```jsx
{points.map((point, index) => (
  <Marker key={index} coordinate={point} pinColor="blue" />
))}
{points.length >= 3 && (
  <Polygon
    coordinates={points}
    strokeWidth={3}
    strokeColor="blue"
    fillColor="rgba(0, 0, 255, 0.2)"
  />
)}
```

- The points have no ids, so the index is the key. That is safe here because points are only ever added to the end or cleared, never reordered.
- A polygon needs at least three points to have an area, so `&&` waits until there are three.
- Make the button clear the points.

```jsx
<Pressable
  style={({ pressed }) => [styles.button, pressed && styles.pressed]}
  onPress={() => setPoints([])}
>
```

- Run it. Tap three places: three blue pins, then a blue shape. Keep tapping and the shape grows. Press **Clear** to start again.
- A tap on one of the map's own labelled places, like a restaurant icon, is a tap on that place, not on the map, so it adds no point. Hiding the places, as in the previous example, fixes that.

### Finished Code

```jsx
import { useState } from "react";
import { View, Text, Pressable, StyleSheet } from "react-native";
import MapView, { Marker, Polygon } from "react-native-maps";

const CAMPUS = {
  latitude: 43.79607,
  longitude: -79.3487,
  latitudeDelta: 0.01,
  longitudeDelta: 0.01,
};

const CAMPUS_OUTLINE = [
  { latitude: 43.79701, longitude: -79.35178 },
  { latitude: 43.79849, longitude: -79.3453 },
  { latitude: 43.79483, longitude: -79.34612 },
  { latitude: 43.79382, longitude: -79.35049 },
];

export default function MyApp() {
  const [points, setPoints] = useState([]);

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        initialRegion={CAMPUS}
        onPress={(e) => {
          const point = e.nativeEvent.coordinate;
          setPoints((prev) => [...prev, point]);
        }}
      >
        <Polygon
          coordinates={CAMPUS_OUTLINE}
          strokeWidth={3}
          strokeColor="red"
          fillColor="rgba(255, 0, 0, 0.2)"
        />
        {points.map((point, index) => (
          <Marker key={index} coordinate={point} pinColor="blue" />
        ))}
        {points.length >= 3 && (
          <Polygon
            coordinates={points}
            strokeWidth={3}
            strokeColor="blue"
            fillColor="rgba(0, 0, 255, 0.2)"
          />
        )}
      </MapView>
      <Pressable
        style={({ pressed }) => [styles.button, pressed && styles.pressed]}
        onPress={() => setPoints([])}
      >
        <Text style={styles.buttonText}>Clear</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  map: { flex: 1 },
  button: {
    position: "absolute",
    bottom: 40,
    alignSelf: "center",
    padding: 16,
    borderRadius: 8,
    backgroundColor: "#2196F3",
  },
  pressed: { backgroundColor: "#1565C0" },
  buttonText: { color: "white", fontSize: 18 },
});
```

## Location and Maps Checklist

- Ask for location when a feature needs it, not when the app opens, and make it clear why.
- Handle "no": a denied permission and location services turned off both need a message, and the rest of the app should still work.
- Show progress while finding the user. A position can take a few seconds.
- Stop watching when the screen closes or the user pauses. A forgotten watcher drains the battery.
- Let the user recentre after they pan away.
- Keep markers few and meaningful, and hide the map's own places when they compete with yours.
- Check every map prop's platform. Some work only on Apple Maps or only on Google Maps.
- Test on a real phone, on both iOS and Android when you can. The web preview cannot show a map.
