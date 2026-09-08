# Week 1.1 - Notes and Examples

## Playground

React Playground (Professor's CodeSpace Template)
(https://codesandbox.io/p/sandbox/react-basic-3rqll4)

## Example: Simple Components

- MyApp is the most parent component.
- It is a JavaScript function
- Tags are case sensitive
  - HTML should remain in lower case.
  - Our custom components will have an uppercase letter at the start.
- Show that after return there is ( ) surrounding the HTML, it is not required. `return <MyButton />;`

```jsx
function MyButton() {
  return <button>I'm a button</button>;
}

function MyApp() {
  return <MyButton />;
}
```

## Example: Multiple Tags

- If you need multiple tags, they cannot be side by side.
- Create a parent tag then add the children.
- Show how to add the H1 tag

```jsx
function MyApp() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <MyButton />
    </div>
  );
}
```

- If you don't want a parent tag, you can use `<>...</>`

```jsx
function MyApp() {
  return (
    <>
      <h1>Welcome to my app</h1>
      <MyButton />
    </>
  );
}
```

## Example: JS-Expressions

- Show that we can define JS constants
- Show that we can use those constants in a tag argument using { }

```jsx
function MyButton() {
  const buttonText = "I'm a button";

  return <button>{buttonText}</button>;
}
```

- Show that we can use these constants as attribute values
- Show that we can negate a Boolean {!varname}

```jsx
function MyButton() {
  const buttonText = "I'm a button";
  const enable = false;

  return <button disabled={!enable}>{buttonText}</button>;
}
```

- Show that html tokens such as &copy; can be used

```jsx
function MyButton() {
  const buttonText = "I'm a button";
  const enable = false;

  return <button disabled={!enable}>{buttonText} &copy;</button>;
}
```

## Example: Passing props

- Pass props to our button component and read information from there instead of constants.
- Show what happens if `buttonText` is omitted.

```jsx
function MyButton(props) {
  return <button disabled={!props.isEnabled}>{props.buttonText}</button>;
}

function MyApp() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <MyButton buttonText="My Button" isEnabled={true} />
    </div>
  );
}
```

- Let's set a default value so when removing `buttonText` the control doesn't look broken.

```jsx
function MyButton(props) {
  return (
    <button disabled={!props.isEnabled}>
      {props.buttonText || "My Button"}
    </button>
  );
}

function MyApp() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <MyButton isEnabled={true} />
    </div>
  );
}
```

- Another way to create default values is by using `destructuring`.

```jsx
function MyButton({ buttonText = "My Button", isEnabled = false }) {
  return <button disabled={!isEnabled}>{buttonText}</button>;
}

function MyApp() {
  return (
    <div>
      <h1>Welcome to my app</h1>
      <MyButton isEnabled={true} />
    </div>
  );
}
```
