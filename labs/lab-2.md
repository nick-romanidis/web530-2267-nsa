# Lab 2 - React Native, Events, and State

## A Note on AI

AI assistants are welcome in this course, but use them to get unstuck, not to do the work. Attempt every question yourself first. If you are stuck, paste **your** code and ask why it does not work or what concept you are missing, rather than asking for the answer. Never keep code you could not explain line by line to a classmate. If you cannot explain it, you cannot debug it or build on it, and it is not yours yet. The point of this lab is not the finished Snack. It is the ability to write these components yourself, and having an assistant write them for you skips the only part that builds that ability.

## Setup

This lab uses React Native only. There is no React DOM, no `div`, and no `createRoot`. Plan for about 60 minutes.

1. You need a free Expo account. Create one at https://expo.dev/signup, or sign in with your GitHub account.
2. Open the React Native Playground (Snack) at https://snack.expo.dev and sign in.
3. Rename the Snack to `WEB530 Lab 2 - Your Name` (click the name at the top left).
4. Open `App.js`, delete everything in it, and paste in the starter below.
5. Click **Save** so the Snack gets a permanent link.

Starter for `App.js`:

```jsx
import { ScrollView, Text } from "react-native";

export default function MyApp() {
  return (
    <ScrollView style={{ marginTop: 60 }}>
      <Text>Question 1</Text>
      {/* Your components go here */}
    </ScrollView>
  );
}
```

About that one line of style: the `style={{ marginTop: 60 }}` on the outer `ScrollView` pushes your content below the phone's clock and status bar so you can see it in the preview. It stays on the outer `ScrollView` in `MyApp` and nowhere else. **It is the only style allowed in this lab.** Do not add `style` props or `StyleSheet` anywhere else. Styling is covered in a later week.

## Running Your App

Snack shows a preview on the right. There are three ways to run your code:

- **Web** tab: fastest, but `Alert.alert` does nothing here.
- **Android** or **iOS** tab: a real emulator in your browser. It can take a minute to start and you may be placed in a queue.
- **My Device**: install the Expo Go app on your phone, then scan the QR code.

Questions 1 and 3 use `Alert.alert`, so check them on the Android or iOS tab or on your phone.

## Submission

- Submit **one** Snack link to the `Lab 2` assessment in Blackboard. After you save, the link looks like `https://snack.expo.dev/@yourname/web530-lab-2-...`.
- The lab is due Monday at 11:59 PM. You have the weekend to complete it.
- Only students who participated in the week 2 Tuesday/Wednesday class, and handed in a lab slip can receive marks for this lab. If you did not, you are still encouraged to complete it as preparation for your first test, even though no marks will be awarded.
- All questions go in the same Snack. Write every component in `App.js`.
- `MyApp` must render the answer to each question in order, under a `Text` heading, like this:

```jsx
export default function MyApp() {
  return (
    <ScrollView style={{ marginTop: 60 }}>
      <Text>Question 1</Text>
      <ReminderButton />
      <ReminderButton task="Submit Lab 2" minutes={30} />

      <Text>Question 2</Text>
      <NameTag />

      {/* and so on */}
    </ScrollView>
  );
}
```

- Before submitting, save one last time and open your link in a private browser window to confirm it loads without signing in.

## Question 1 - Events and Props (1 point)

Create a component called `ReminderButton` that renders a `Button`. It accepts two props with default values:

- `task` (string), default `"Drink water"`
- `minutes` (number), default `10`

When pressed, the button shows an alert with the title `Reminder` and the message `TASK in MINUTES minutes`, for example `Drink water in 10 minutes`.

Requirements:

- The button's `title` must be the value of `task`.
- Write the press handler as a named function inside the component (for example `handlePress`), not inline in the JSX.
- Render `ReminderButton` twice in `MyApp`: once with no props, and once with both props set to values of your choice.

Hint: `Alert.alert("Title", "Message")` takes the title first. Import `Alert` from `react-native`.

## Question 2 - Text Input and State (1 point)

Create a component called `NameTag` that renders a `TextInput` and a `Text`.

- The `TextInput` has the placeholder `Type your name`.
- The `Text` below it reads `Hello, NAME` and updates on every keystroke.
- When the input is empty, the `Text` reads `Hello, stranger`.

Requirements:

- Store the typed value in state with `useState`.
- The `TextInput` must be controlled: pass the state to `value` and update it with `onChangeText`.

Hint: `onChangeText` gives you the new string directly, so `onChangeText={setName}` works. There is no event object to dig through.

## Question 3 - Toggling State and Conditional Rendering (1 point)

Create a component called `NotificationSettings` with three parts:

- A `Text` that reads `Notifications are on` or `Notifications are off`. Notifications are off when the app starts.
- A `Button` that toggles notifications. Its title reads `Turn notifications on` when they are off, and `Turn notifications off` when they are on.
- A `Button` titled `Send test notification` that is **disabled** while notifications are off. When pressed, it shows an alert saying `Test notification sent`.

Requirements:

- One `useState` holds the on/off value as a boolean.
- The toggle button flips the boolean. Use the function form: `setEnabled((prev) => !prev)`.
- The send button uses the `disabled` prop. Do not hide the button; it must stay visible and greyed out.

Hint: A ternary inside JSX chooses between two strings: `{enabled ? "on" : "off"}`. The same trick works inside the `title` prop.

## Question 4 - Lifting State Up (1 point)

Build a basketball scoreboard from two components.

`ScoreButton` is a child component with three props:

- `label` (string) - the button title
- `points` (number) - how much to add to the score, can be negative
- `onScore` (function) - called with `points` when the button is pressed

`Scoreboard` is the parent. It owns the score in state and renders:

- A `Text` that reads `Score: N`
- Three `ScoreButton`s: `Three pointer` (+3), `Free throw` (+1), and `Technical foul` (-2)
- A `Button` titled `New game` that sets the score back to 0

Requirements:

- `ScoreButton` must **not** have any state of its own. It only reports the press to the parent.
- When adding points, update state with the function form: `setScore((prev) => prev + points)`.
- Render `<Scoreboard />` in `MyApp`.

## Question 5 - useEffect and Cleanup (1 point)

Create a component called `Stopwatch` that counts seconds.

- A `Text` reads `Elapsed: N seconds`.
- A `Button` toggles between `Start` and `Stop`. Its title changes to match what it will do next.
- A `Button` titled `Reset` sets the seconds back to 0. It must work whether the stopwatch is running or stopped.

Requirements:

- Two pieces of state: the elapsed seconds and whether the stopwatch is running.
- Use `useEffect` with `[running]` as its dependency array.
- Inside the effect, if the stopwatch is not running, do nothing. If it is running, start a `setInterval` that adds one second every 1000 ms, and return a cleanup function that calls `clearInterval`.
- Add `console.log("Stopwatch cleanup")` inside the cleanup function. Press Start, then Stop, and confirm the message appears in the console.
- Inside the interval, update the seconds with the function form `setSeconds((prev) => prev + 1)`. Add a comment above that line explaining, in your own words, why `setSeconds(seconds + 1)` would not work here.

Hint: `setInterval` returns an id. Keep it in a `const` and pass it to `clearInterval` in the cleanup.

## Question 6 - Shared State Without Props (1 point)

Build a tiny shopping cart where the count is shared between components that are **not** parent and child.

- `CartBadge` renders a `Text` that reads `Cart: N items`.
- `AddToCartButton` accepts an `item` prop (string) and renders a `Button` titled `Add ITEM`. Pressing it adds 1 to the cart count.
- In `MyApp`, render one `CartBadge` followed by two `AddToCartButton`s with different `item` values, for example `Coffee` and `Bagel`. The badge must update when either button is pressed.

Requirements:

- `MyApp` must **not** hold the cart count in state, and no cart count or handler may be passed as a prop. `CartBadge` and `AddToCartButton` read and update the shared value themselves.
- Use **one** of the two approaches from class:
  - **React Context**: create a context with `createContext`, write a `CartProvider` component that owns the state, wrap the `ScrollView` in `MyApp` with it, and read the value with `useContext` in both components.
  - **Jotai**: create one `atom` for the count and use `useAtom` in both components. When you import from `jotai`, Snack offers to add it to `package.json`. Accept that. If it does not, add `"jotai": "*"` to the `dependencies` in `package.json` yourself.
- Add a comment above `CartBadge` naming which approach you used and one sentence on why you picked it.

You only need one approach for marks, but it is recommended that you code this question both ways as preparation for your test. Both techniques are testable. If you do both, keep your submitted `App.js` to one approach and put the other version in a comment block or a second file in the Snack.
