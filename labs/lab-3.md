# Lab 3 - Layout and Styling

## A Note on AI

AI assistants are welcome in this course, but use them to get unstuck, not to do the work. Attempt every question yourself first. If you are stuck, paste **your** code and ask why it does not work or what concept you are missing, rather than asking for the answer. Never keep code you could not explain line by line to a classmate. If you cannot explain it, you cannot debug it or build on it, and it is not yours yet. The same goes for the game: looking up the answer to a level skips the practice the level was built to give you. The point of this lab is not the finished Snack or the finished game. It is the ability to lay out a screen yourself.

## Overview

This lab has two questions. Plan for about 60 minutes.

- Question 1 is a browser game that teaches flexbox. You submit a screenshot.
- Question 2 is a React Native component you build in Expo Snack. You submit the Snack link.

## Submission

- Submit to the `Lab 3` assessment in Blackboard:
  - One screenshot for Question 1.
  - One Snack link for Question 2. After you save, the link looks like `https://snack.expo.dev/@yourname/web530-lab-3-...`.
- The lab is due Monday at 11:59 PM. You have the weekend to complete it.
- Only students who participated in the week 3 Tuesday/Wednesday class, and handed in a lab slip can receive marks for this lab. If you did not, you are still encouraged to complete it as practice.
- Before submitting, save the Snack one last time and open your link in a private browser window to confirm it loads without signing in.

## Question 1 - Flexbox Froggy (1 point)

React Native lays out every screen with flexbox, so this is the layout system you will use for the rest of the course.

Complete all 24 levels of Flexbox Froggy: (https://flexboxfroggy.com/)

- The game uses CSS names with dashes (`justify-content`, `flex-direction`). In React Native the same properties are camelCase (`justifyContent`, `flexDirection`) and the values are strings (`"center"`).
- One difference to remember: in the game, and on the web, `flex-direction` starts as `row`. In React Native, `flexDirection` starts as `"column"`.
- Your progress is saved in your browser. If you finish over more than one sitting, use the same browser each time.

To submit:

- Click **Level 24 of 24** at the top left to open the level menu. Every level from 1 to 24 must show as solved.
- Take a screenshot of the whole browser window with that menu open.

## Question 2 - Profile Card (4 points)

### Setup

1. Open the React Native Playground (Snack) at https://snack.expo.dev and sign in.
2. Rename the Snack to `WEB530 Lab 3 - Your Name` (click the name at the top left).
3. In the file list on the left, add a new file named `styles.js` in the same folder as `App.js`.
4. Paste in the two starter files below.
5. Click **Save** so the Snack gets a permanent link.

Starter for `App.js`:

```jsx
import { ScrollView, Text } from "react-native";
import styles from "./styles";

export default function MyApp() {
  return (
    <ScrollView style={styles.screen}>
      <Text>Question 2</Text>
      {/* Your ProfileCards go here */}
    </ScrollView>
  );
}
```

Starter for `styles.js`:

```js
import { StyleSheet } from "react-native";

export default StyleSheet.create({
  screen: {
    marginTop: 60,
    backgroundColor: "#f0f0f0",
  },
});
```

Check your work in more than one preview. The shadow in Part D looks different on the Web, Android, and iOS tabs, and that is the point.

### The Component

Create a component called `ProfileCard` in `App.js`. It displays a social media style profile and accepts these props:

- `name` (string)
- `handle` (string), for example `"@jordan"`
- `imageUri` (string), a link to an image. `https://picsum.photos/200` works, or use any image link you like.
- `topic` (string), default `"React Native"`
- `posts`, `followers`, `following` (numbers)

In `MyApp`, render `ProfileCard` **twice** under the `Question 2` heading, with different values for every prop. Leave `topic` out of one of them so the default shows.

The card is built in four parts, each worth 1 point.

### Part A - Header Row (1 point)

The top of the card is a row with the photo on the left and the name and handle on the right.

- The photo is an `Image` that is 80 wide and 80 high, shown as a circle.
- To the right of the photo, the name sits above the handle. The name is size 20 and bold. The handle is grey.
- There is a gap between the photo and the text.
- The name and handle are centred vertically against the photo, not stuck to the top of it.

Hint: A remote image shows nothing unless it has a width and height. A `borderRadius` of half the width turns a square into a circle.

### Part B - Bio Line (1 point)

Under the header row, show one line of text:

`Currently learning TOPIC.`

- `TOPIC` is the value of the `topic` prop.
- Only the topic is bold and coloured (pick any colour). The rest of the sentence is not.
- The sentence has a font size of 16, and the topic must be the same size without you setting `fontSize` on it a second time.

Hint: A `Text` inside a `Text` inherits the outer `Text`'s styles.

### Part C - Stats Row (1 point)

At the bottom of the card, show three stats side by side: **Posts**, **Followers**, and **Following**.

- Each stat is a number from the props above a small label, for example `128` over `Posts`.
- The number is large and bold. The label is smaller and grey.
- The three stats have equal widths and are centred in their space.
- A thin line separates the stats row from the bio above it.

Requirements:

- The equal widths must come from flexbox. Do not give the stats a fixed `width`.
- Write a small `Stat` component that takes `value` and `label` props, and use it three times inside `ProfileCard`.

### Part D - The Card and Styles File (1 point)

- The card has a white background, rounded corners, padding inside, and a margin of 16 around it so it does not touch the screen edges or the other card.
- The card has a shadow, set with `Platform.select()` inside the StyleSheet. Use these values:
  - `ios`: `shadowColor: "#000"`, `shadowOffset: { width: 0, height: 2 }`, `shadowOpacity: 0.2`, `shadowRadius: 4`
  - `android`: `elevation: 4`
  - `default`: `borderWidth: 1` and `borderColor: "#ddd"`
- Add a comment above the `Platform.select()` explaining, in your own words, why the web preview gets the `default` style.

Requirements:

- Every style is in `styles.js`, created with `StyleSheet.create()`, and imported into `App.js`. There are **no** inline `style={{ ... }}` objects anywhere in `App.js`.
- Style names describe what they are for (`card`, `headerRow`, `statLabel`), not what they look like (`bold`, `grey`, `box1`).

Hint: `Platform.select()` returns an object here, so spread it into the card style with `...Platform.select({ ... })`. Remember to import `Platform` into `styles.js`.
