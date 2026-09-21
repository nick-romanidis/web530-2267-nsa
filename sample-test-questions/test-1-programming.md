# WEB530 Test 1 – Sample Programming Question: Class Poll

This is a practice question in the same format, length and difficulty as the programming question on Test 1. The test uses a different scenario, so do not memorise this one. Practise the skills: a stateless child with props, a default value and PropTypes; a parent that owns state; a controlled `TextInput`; conditional rendering with a ternary; `map` with a `key`; the function form of a state setter; and a `useEffect` with a timer and a cleanup.

Total: 24 marks. Suggested time: 35 minutes.

## Before you start

- Open the React Native shell Snack at https://snack.expo.dev/@nick-romanidis/react-native-shell, delete everything in `App.js`, and paste in the starter below. The **Web** preview is enough. Nothing here needs `Alert` or a phone.
- Use the course notes. Every part below describes *behaviour*, not code. Code copied from the notes without adapting it will not meet the requirements.
- Do **not** use the Context API, Jotai, or class components. Do **not** add `StyleSheet` or any `style` prop other than the one on the outer `ScrollView`.
- On the test you will not be able to log in to Expo or save the Snack, so get used to working in an unsaved Snack and copying `App.js` somewhere safe every few minutes.

## Starter

The `ScrollView` lets the screen scroll if the content gets tall, and the one `style` on it pushes your content below the phone's clock. Leave both as they are and add your components above `MyApp`. Fill in the imports as you need them.

```jsx
//import {  } from 'react';
import { ScrollView } from 'react-native';

const questions = [
  'Should Lab 3 be due on a Friday?',
  'Do you want a review session before the test?',
];

export default function MyApp() {
  return (
    <ScrollView style={{ marginTop: 60 }}>
      {/* Part B: one Poll for every question (see below) */}
    </ScrollView>
  );
}
```

## Scenario

You are building a quick class poll screen. Each poll asks one question. A student types their name, then presses a button to vote Yes, No or Maybe. The screen shows how long each poll has been open. The instructor can close a poll, after which no more votes are accepted.

## Part A – `VoteButton` (5 marks)

Create a functional component called `VoteButton` with these props:

- `choice` – string, required
- `onVote` – function, required
- `label` – string, optional, defaults to `Vote`

It renders a single `Button`:

- The title is `LABEL CHOICE`, for example `Vote Yes`, or `Pick Yes` if `label="Pick"` was passed.
- When pressed, it calls `onVote` and passes `choice` as the argument.

`VoteButton` has **no state of its own**. Declare PropTypes for all three props that match the list above, including whether each is required.

## Part B – `Poll` display and `MyApp` (5 marks)

Create a functional component called `Poll` with one prop:

- `question` – string

For now, `Poll` renders:

- A `Text` showing the question.
- A `Text` reading `No votes yet.`
- Three `VoteButton`s with the choices `Yes`, `No` and `Maybe`, in that order.

In `MyApp`, replace the comment inside the `ScrollView` so that it renders one `Poll` for each element of the `questions` array from the starter. Do not write `<Poll>` twice by hand. React will warn about a missing `key` if you forget it. PropTypes are only required on `VoteButton`.

## Part C – Voting (8 marks)

Add voting to `Poll`. `Poll` owns all of the state. `VoteButton` still has none.

- Add a `TextInput` with the placeholder `Voter name`. It must be a controlled input backed by state.
- While the name is empty, the three `VoteButton`s are **not rendered**. In their place is a `Text` reading `Enter your name to vote.` As soon as a name is typed, the `Text` disappears and the three buttons appear.
- Pressing a `VoteButton` adds one to the vote count and records the last vote as `NAME voted CHOICE`, for example `Sam voted Maybe`.
- The `No votes yet.` `Text` from Part B now reads `Votes: N. Last: NAME voted CHOICE` once at least one vote has been cast. Before the first vote it still reads `No votes yet.`

## Part D – Open timer and closing (6 marks)

Add a timer to `Poll` that shows how long the poll has been open, and a way to close it.

- A `Text` reads `Open for N seconds`, starting at 0 and going up by one every second. Use `useEffect` with `setInterval`. The effect must return a cleanup function that cancels the interval.
- A `Button` titled `Close poll`. Pressing it **closes** the poll.
- Once closed, the timer stops and the `Text` reads `Closed after N seconds with M votes`, or `Closed after N seconds. No votes.` if nobody voted.
- Once closed, the three `VoteButton`s are not rendered even if a name is typed. In their place is a `Text` reading `Voting has ended.` The `Close poll` button is disabled.
- There are two polls on screen. Each must have its own independent timer and its own votes.

## Checking your work

- With no name typed, you see `Enter your name to vote.` and no vote buttons. Type a name and the three buttons replace it. Delete the name and the message comes back.
- Vote `Yes` then `Maybe`. The text reads `Votes: 2. Last: NAME voted Maybe`.
- Press `Close poll` on the first poll. Its timer freezes, the vote buttons are replaced by `Voting has ended.`, and `Close poll` greys out. The second poll keeps counting and still accepts votes.
- Try `<VoteButton choice="Yes" onVote={handleVote} label="Pick" />` once to confirm the default works, then put it back.
- Check the browser console. There should be no red errors and no warning about a missing `key`.
