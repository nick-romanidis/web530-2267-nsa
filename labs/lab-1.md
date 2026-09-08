# Lab 1 - Components and Props

## A Note on AI

AI assistants are welcome in this course, but use them to get unstuck, not to do the work. Attempt every question yourself first. If you are stuck, paste **your** code and ask why it does not work or what concept you are missing, rather than asking for the answer. Never keep code you could not explain line by line to a classmate. If you cannot explain it, you cannot debug it or build on it, and it is not yours yet. The point of this lab is not the finished sandbox. It is the ability to write these components yourself, and having an assistant write them for you skips the only part that builds that ability.

## Setup

1. You need a free CodeSandbox account. Sign in with your GitHub account, or create one at https://codesandbox.io.
2. Open the React Playground (Professor's CodeSandbox Template) and fork it:
   (https://codesandbox.io/p/sandbox/react-basic-3rqll4)
3. Rename the sandbox to `WEB530 Lab 1 - Your Name`.
4. Open `package.json` and verify it matches the file below. It must use React 18 and include `prop-types`. If it does not, replace the contents with the file below and restart the sandbox.

```json
{
  "name": "react",
  "version": "1.0.0",
  "description": "",
  "keywords": [],
  "main": "src/index.js",
  "dependencies": {
    "prop-types": "^15.8.1",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-scripts": "^5.0.0"
  },
  "devDependencies": {
    "loader-utils": "3.2.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "browserslist": [">0.2%", "not dead", "not ie <= 11", "not op_mini all"]
}
```

## Submission

- Submit **one** CodeSandbox link to the `Lab 1` assessment in Blackboard.
- The lab is due before the end of Friday's class.
- All questions go in the same sandbox. Write every component in `src/index.js`.
- `MyApp` must render the answer to each question in order, under a heading, like this:

```jsx
function MyApp() {
  return (
    <div>
      <h2>Question 1</h2>
      <HelloMessage />

      <h2>Question 3</h2>
      <MyButton buttonText="Save" isEnabled={true} />
      <MyButton buttonText="Delete" isEnabled={false} />

      {/* and so on */}
    </div>
  );
}
```

- Before submitting, make sure the sandbox is public. Open your link in a private browser window to confirm it loads without signing in.

## Question 1 - Your First Component (1 point)

Create a component called `HelloMessage` that displays the text: "Hello from my component!"

Starter:

```jsx
import { createRoot } from "react-dom/client";

function MyApp() {
  return <div>{/* Render your HelloMessage component here */}</div>;
}

createRoot(document.getElementById("root")).render(<MyApp />);
```

## Question 2 - Add a Button (1 point)

Add a button under your text in `HelloMessage` that logs "Button pressed!" to the console.

Hint:

```jsx
<button onClick={() => console.log("Button pressed!")}>Press me</button>
```

## Question 3 - Create a Reusable Button Component (1 point)

Create a component called `MyButton` that accepts:

- `buttonText` (string)
- `isEnabled` (boolean)

The button must display `buttonText` and be disabled when `isEnabled` is false.

Render it twice with different props.

## Question 4 - Display Dynamic Text (1 point)

Create a component called `WelcomeMessage` that accepts a `name` prop and displays:

```
Welcome, NAME!
```

Render it with your own name.

## Question 5 - Using JavaScript Inside Components (1 point)

Create a component called `ProfileCard` that displays:

- A name
- A program
- A fun fact
- A button that does nothing yet (we'll use it later)

You must use JavaScript variables inside the component to store the values.

Requirements:

- Inside `ProfileCard`, create three variables:

```jsx
const name = "Your Name";
const program = "Your Program";
const funFact = "Something interesting about you";
```

- Display each variable inside a `<p>` element.
- Add a `<button>` at the bottom with the text "Learn More".
- Render `<ProfileCard />` inside `MyApp`.

## Question 6 - Prop Validation (1 point)

Add PropTypes to two of your components:

- `MyButton`: `buttonText` is a required string, `isEnabled` is an optional boolean.
- `WelcomeMessage`: `name` is a required string.

Then prove the validation works:

- In `MyApp`, add one **commented-out** line that would break one of these rules, for example passing a number as `name`.
- Above it, add a comment containing the warning React printed in the console when you tried it.

Example of what this looks like:

```jsx
{
  /* Warning: Failed prop type: Invalid prop `name` of type `number` supplied to `WelcomeMessage`, expected `string`. */
}
{
  /* <WelcomeMessage name={42} /> */
}
```

Remember: PropTypes only warn in the console. The page still renders.
