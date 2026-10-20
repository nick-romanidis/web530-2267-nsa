# Lab 5 - Lists

## A Note on AI

AI assistants are welcome in this course, but use them to get unstuck, not to do the work. Attempt every question yourself first. If you are stuck, paste **your** code and ask why it does not work or what concept you are missing, rather than asking for the answer. Never keep code you could not explain line by line to a classmate. If you cannot explain it, you cannot debug it or build on it, and it is not yours yet. The point of this lab is not the finished Snack. It is being able to put any array on screen as a list, and to handle everything that can happen while that array is on its way from a server.

## Overview

You build one app, a contact list, and each question adds to it. Plan for about 60 minutes.

- Questions 1 to 3 use a small array of contacts that is already in the starter.
- Question 4 swaps that array for real data from an API.
- Questions 5 and 6 add a contact screen and pull to refresh.

Because every question builds on the one before, do them in order. Your submitted Snack shows the finished app, and it is marked against all six questions.

## Setup

1. Open the React Native Playground (Snack) at https://snack.expo.dev and sign in.
2. Rename the Snack to `WEB530 Lab 5 - Your Name` (click the name at the top left).
3. Keep Snack on SDK 54 (the default). SDK 55 does not load `react-native-screens`, so the navigation in this lab does not run on it.
4. In the file list on the left, add a new file named `styles.js` in the same folder as `App.js`.
5. Replace the contents of `package.json`, `App.js` and `styles.js` with the three starter files below.
6. Click **Save** so the Snack gets a permanent link.

Starter for `package.json`:

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

Starter for `App.js`:

```jsx
import { View, Text } from "react-native";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import styles from "./styles";

// Questions 1 to 3 use this array. Question 4 replaces it with the API.
const CONTACTS = [
  {
    id: 1,
    name: "Priya Shah",
    email: "priya@example.com",
    address: { city: "Toronto" },
  },
  {
    id: 2,
    name: "Marcus Lee",
    email: "marcus@example.com",
    address: { city: "Ottawa" },
  },
  {
    id: 3,
    name: "Ana Costa",
    email: "ana@example.com",
    address: { city: "Toronto" },
  },
  {
    id: 4,
    name: "Liam O'Brien",
    email: "liam@example.com",
    address: { city: "Halifax" },
  },
  {
    id: 5,
    name: "Sofia Rossi",
    email: "sofia@example.com",
    address: { city: "Vancouver" },
  },
  {
    id: 6,
    name: "Daniel Kim",
    email: "daniel@example.com",
    address: { city: "Montreal" },
  },
];

const Stack = createNativeStackNavigator();

function ContactsScreen({ navigation }) {
  return (
    <View style={styles.container}>
      <Text>Your list goes here</Text>
    </View>
  );
}

export default function MyApp() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Contacts" component={ContactsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

Starter for `styles.js`:

```js
import { StyleSheet } from "react-native";

export default StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
  },
});
```

The stack navigator is already there, so the app has a **Contacts** header from the start and you do not need `marginTop` to get below the status bar. Look at the shape of each contact: the city is inside an `address` object. That is on purpose. It matches the API you switch to in Question 4.

## Styling Rules

- Every style is in `styles.js`, created with `StyleSheet.create()`, as in Lab 3. There are **no** inline `style={{ ... }}` objects anywhere in `App.js`.
- Style names describe what they are for (`row`, `contactName`, `searchInput`), not what they look like (`bold`, `grey`, `box1`).
- Keep the styling simple. This lab is about lists, not looks.

## Running Your App

The **Web** preview is enough for Questions 1 to 5. Pull to refresh in Question 6 needs a touch screen, so check it on the **Android** or **iOS** tab, or on your phone with **My Device** and Expo Go.

## Submission

- Submit **one** Snack link to the `Lab 5` assessment in Blackboard. After you save, the link looks like `https://snack.expo.dev/@yourname/web530-lab-5-...`.
- The lab is due at the end of class. Submit before you leave.
- Only students who participate in class and hand in a lab slip can receive marks for this lab. If you miss the class, you are still encouraged to complete it as practice.
- Before submitting, save one last time and open your link in a private browser window to confirm it loads without signing in.

## Question 1 - FlatList (1 point)

Replace the placeholder `Text` in `ContactsScreen` with a `FlatList` that shows every contact in `CONTACTS`.

Each row shows:

- The contact's **name**, large and bold, with the **email** under it in a smaller grey font.
- The contact's **city** on the right-hand side of the row, in a colour of your choice.
- A thin line under each row to separate it from the next.

Requirements:

- Use `FlatList` with `data`, `renderItem` and `keyExtractor`. Do not use `ScrollView` or `.map()` for the rows.
- The `id`s are numbers. Your `keyExtractor` must return a string.
- The name and email sit on the left, the city on the right, and a long name must not push the city off the screen. Use flexbox from week 3, not a fixed `width`.

Hint: Make the row a `View` with `flexDirection: "row"`. Wrap the name and email in their own `View` and give it `flex: 1`, so it takes all the space the city does not need. The city is at `item.address.city`.

## Question 2 - Sorting (1 point)

Add two buttons above the list: `Sort by name` and `Sort by city`.

- The list starts sorted by name, A to Z.
- `Sort by city` sorts the list by city, A to Z. Contacts in the **same city** are sorted by name. With the starter data, Ana Costa must come before Priya Shah, even though Priya is first in the array.
- The button for the order that is already showing is **disabled**. When the app starts, `Sort by name` is disabled.

Requirements:

- One piece of state holds which sort is active, for example `"name"` or `"city"`.
- The sorted array is **not** stored in state. Build it from `CONTACTS` and the sort state on every render, above the `return`.
- The original `CONTACTS` array must never be changed. Add a comment above your sort explaining, in your own words, why you copy the array before calling `.sort()` instead of sorting `CONTACTS` itself.
- Lay out the two buttons side by side in a row.

Hint: `localeCompare` returns `0` when two strings are equal, and `0` is falsy. So `a.address.city.localeCompare(b.address.city) || a.name.localeCompare(b.name)` compares cities first, and only compares names when the cities are the same.

## Question 3 - Search, Count and Empty State (1 point)

Add a controlled `TextInput` between the buttons and the list, with the placeholder `Search by name or city`.

- Typing filters the list. A contact stays in the list if the text appears in its **name or its city**. Upper and lower case do not matter: `tor`, `TOR` and `Tor` all find both Toronto contacts.
- The search and the sort work together. Search `o` and then switch the sort, and the filtered list re-sorts.
- Above the first row, show `Showing N of M contacts`, where `N` is the number of rows on screen and `M` is the total number of contacts. This text must scroll with the list.
- When nothing matches, the list shows `No contacts match "TEXT".` with the search text in it, for example `No contacts match "zz".`

Requirements:

- Filter first, then sort, in the same derived array as Question 2. There is still only the sort state and the search text in state.
- `.filter()` returns a new array, so the copy from Question 2 is no longer needed. Keep your Question 2 comment and update it to say what keeps the original array safe now.
- Use `ListHeaderComponent` for the count and `ListEmptyComponent` for the empty message. A `Text` placed above the `FlatList` does not scroll with it, so it does not count.

## Question 4 - Fetching the Contacts (1 point)

Replace the starter data with real data from this API, which returns ten users in the same shape as the starter array (plus a few extra fields):

`https://jsonplaceholder.typicode.com/users`

Open the link in a browser tab first and find `name`, `email` and `address.city` in the first user.

- **Delete** the `CONTACTS` array. The contacts now live in state, start as an empty array, and are loaded once when the screen first renders.
- While the first load is running, the whole screen shows an `ActivityIndicator` and the text `Loading contacts...` instead of the list.
- If the load fails, the whole screen shows a message for the user and a `Try Again` button that shows the spinner and loads again. The message must say what happened and what to do. No status codes.
- If the load succeeds but the API returns no contacts, the list shows `No contacts yet.` instead of `No contacts match "".` The match message from Question 3 is only for when the user has typed something.
- `Showing N of M contacts` still works, now out of 10.

Requirements:

- Use `useEffect` with an empty dependency array, and `fetch`.
- Check `res.ok` before calling `res.json()`. Add a comment above that check explaining, in your own words, why `.catch()` alone is not enough.
- Test all three cases before you move on, and put the URL back after each one:
  - The error screen: change the end of the URL to `/userz`. The API answers 404.
  - `Try Again`: with the URL still broken, press it. The spinner shows, then the error comes back.
  - The empty list: change the end of the URL to `/users?id=99`. No user has that id, so the API returns `[]`.

Hint: This is the Fetching List Data example from the week 5 notes. Use the same three pieces of state and the same early `return`s, then check that your Question 1 to 3 code still works with `contacts` in place of `CONTACTS`.

## Question 5 - Contact Screen (1 point)

Pressing a row opens a second screen in the stack that shows that contact's full details.

- Register a second screen in the stack named `Contact`. Its header title is the contact's name.
- Each row is a `Pressable`. Pressing it navigates to `Contact` with **two params only**: the contact's `id` and `name`. Do **not** pass the whole contact object.
- The contact screen fetches that one contact itself from `https://jsonplaceholder.typicode.com/users/ID`, where `ID` is the `id` param. While it loads, the screen shows an `ActivityIndicator`.
- When it has loaded, the screen shows the contact's name as a large heading, then their email, phone, website, street and city, and company name, each with a label (`Email:`, `Phone:` and so on). The company catch phrase goes last, in italics.
- If the request fails, the screen shows `Could not load this contact.` To check it, temporarily change the URL on this screen to end in `/users/99`.
- Going back keeps the list as it was: the same sort, the same search text.

Requirements:

- The contact screen's `useEffect` has `[id]` as its dependency array.
- Add a comment above your `navigate` call explaining, in your own words, why this app passes only the `id` instead of the whole contact, and what it costs.

Hint: `options` can be a function that receives `route`, as in week 4. The phone and website are at the top level of the user, the street is `address.street`, and the company fields are `company.name` and `company.catchPhrase`. A `Text` inside a `Text`, from Lab 3, lets you make only the label bold.

## Question 6 - Pull to Refresh (1 point)

Let the user reload the contacts by pulling down on the list.

- Pulling down on the list shows the refresh spinner at the top, reloads the contacts from the API, and hides the spinner when it is done.
- During a refresh the list **stays on screen**. The full-screen `Loading contacts...` spinner from Question 4 is only for the first load and for `Try Again`.
- The search text and the sort survive a refresh.
- Under `Showing N of M contacts`, show `Updated` and the time the contacts last arrived, for example `Updated 10:42:07 AM`. Every successful load or refresh updates it.

Requirements:

- Use the `refreshing` and `onRefresh` props of `FlatList`. Do not write any gesture code.
- The refresh has its own piece of state, separate from the loading state. Add a comment above it explaining, in your own words, why one `loading` state cannot do both jobs.
- Store the time in state where the contacts are stored, and show it with `toLocaleTimeString()`.

Check this one on the **Android** or **iOS** tab, or on your phone. Pulling does nothing in the web preview. The API returns the same ten users every time, so the updated time is how you know the refresh worked.
