# Week 10.1 - Notes and Examples

## Firebase and Cloud Firestore

- So far our data has lived in the app (a hard-coded array) or come from someone else's API (jsonplaceholder in Week 5). Today the app gets a database of its own that it can read **and** write.
- **Firebase** is Google's Backend-as-a-Service: a database, user accounts, file storage and hosting that Google runs for you. There is no server to write or deploy.
- **Cloud Firestore** is Firebase's database. It stores **documents** (one record, a set of named fields, like one book) inside **collections** (a group of documents, like `books`). Each document has an id.
- Firestore has a **REST API**: every collection and every document has its own URL, and you use it with `fetch`, exactly as you used jsonplaceholder. That is how we use it in this course. Google also publishes a `firebase` npm package that wraps these requests in its own functions. We skip it so you can see the real requests, and so nothing in the app is new except the URLs.
- The HTTP method says what you want to do with a URL. Together the four are called **CRUD**:

| CRUD   | HTTP method | URL                    | Example                     |
| :----- | :---------- | :--------------------- | :-------------------------- |
| Create | `POST`      | the collection         | add a book                  |
| Read   | `GET`       | the collection, or one document | list the books, or open one |
| Update | `PATCH`     | one document           | change a book's page count  |
| Delete | `DELETE`    | one document           | remove a book               |

- Today's app is a **Reading List**: a list of books from Firestore, a details screen for each one, sign-in with Firebase Authentication, and screens to add and delete books. We build it in a **new project**, not in last week's shop. The Docusaurus [Getting Started with Firebase](https://seneca-ictoer.github.io/WEB530/I-Firebase/fb-getting-started) page does the same steps on the Product Shop, if you want a second example.

## Firebase Project Setup

- Do this part in the browser, in the Firebase console. You need a Google account. The console changes its menus from time to time, so a label may differ slightly from these notes.

### Create the Project

1. Go to [console.firebase.google.com](https://console.firebase.google.com/) and sign in.
2. Click **Get started by setting up a Firebase project**. If you already have a project, the button says **Create a new Firebase project** instead. Enter a name such as `reading-list`, accept the terms and click **Continue**.
3. Turn AI assistance on or off (either is fine) and click **Continue**.
4. Turn **Google Analytics off**. We do not need it. Click **Create project**, then **Continue** when it is ready.

### Register a Web App

- The app talks to Firebase over the web, so Firebase needs to know it as a **web** app, even though it runs on a phone.

1. In the left menu, under **Project Overview**, click **Settings**, then **General**.
2. Under **Your apps**, click the **Web** icon (`</>`).
3. Enter a nickname such as `Reading List App` and click **Register app**. Do not set up Firebase Hosting.
4. Firebase shows a `firebaseConfig` object. You need two values from it: `projectId` and `apiKey`. The project ID is usually the name you typed plus a few extra characters, like `reading-list-1a2b3`, so copy it rather than typing the name. You can come back to **Settings**, **General** to see them again at any time.
5. Click **Continue to console**.

- The API key is not a password. It only tells Google which project a request is for, and it ends up inside every copy of your app. What protects your data is the **security rules**, which you set at the end of today.

### Create the Database

1. In the left menu, open **Databases & Storage**, then **Firestore**.
2. Click **Create database**.
3. Choose **Standard edition** and click **Next**.
4. Leave the database ID as `(default)`: the app's URLs use that name. Choose a location. There are no Canadian locations, so pick one in the United States. Click **Next**.
5. Choose **Start in test mode** and click **Next**.
6. Leave **Scheduled backups** off. Backups need the paid Blaze plan, so ignore the **Upgrade project** link. Click **Create**.

- **Test mode** lets anyone who knows your project ID read and write the database for the next 30 days. That makes development easy, and it is why you never put real or personal data in a test-mode database. At the end of today we replace it with proper rules.

## Expo Project Setup

- Create a new project, the same way as last week. If anything here is unfamiliar, the Week 9 notes explain each step.

```bash
npx create-expo-app@latest books --template default@sdk-58
cd books
npm run reset-project
npm install jotai
```

- Answer **n** when `reset-project` asks about moving the demo files.
- Open `app.json` and change `"output": "server"` to `"output": "single"` in the `"web"` section.
- Start the server with `npx expo start` and press **w**. On a phone on the college Wi-Fi, use `npx expo start --tunnel`.
- `fetch` to Firebase works the same in the browser and on a phone. Everything today can be shown in the browser.
- If a red screen says `[Refresh] Expected to find the updated module.`, your code is fine: the live reload lost track of a file, often the first time a file imports a package. Reload the page (or press **r** in the terminal). If it comes back, stop the server and start it with `npx expo start -c`, which clears its cache.

## Example: Reading Books

- Connect the app to your project, add some books in the console, and show them in a list.

### Starter Code

- Your project's settings go in one file, so no other file has to know them. In `src`, create a folder `firebase` and a file `src/firebase/config.ts`. Replace both placeholder values with your own `projectId` and `apiKey` from **Settings**, **General**. `your-project-id` is not a real project.

```ts
// From the Firebase console: Settings, General, Your apps.
export const PROJECT_ID = "your-project-id";
export const API_KEY = "your-web-api-key";

// Every Firestore request starts here, then adds a collection and maybe a document id.
export const FIRESTORE_URL = `https://firestore.googleapis.com/v1/projects/${PROJECT_ID}/databases/(default)/documents`;

// Sign up and sign in go to Firebase Authentication, a different Google server.
export const AUTH_URL = "https://identitytoolkit.googleapis.com/v1/accounts";
```

- `FIRESTORE_URL` is the address of your database. A collection's URL is `FIRESTORE_URL` plus `/books`, and a document's URL adds its id: `/books/abc123`.
- `config.ts` is not a screen, so it goes in `src/firebase/`, next to `app/`, not inside it.

### Step 1 - Add Books and See Them

- In the console, open **Firestore**, click **Start collection** and name it `books`. Click **Next**.
- Click **Auto-ID** for the document ID, then add three fields:
  - `title`, type **string**, for example `The Hobbit`
  - `author`, type **string**, for example `J.R.R. Tolkien`
  - `pages`, type **int64** (a whole number), for example `310`
- Click **Save**. Then click **Add document** and add two more books the same way, with the same three field names, spelled exactly the same:
  - `Dune`, `Frank Herbert`, `412`
  - `Station Eleven`, `Emily St. John Mandel`, `333`
- Before writing any code, look at what Firestore sends. Copy this URL into a new browser tab, then **replace `your-project-id` with your own project ID** (**Settings**, **General**, **Project ID**):

```
https://firestore.googleapis.com/v1/projects/your-project-id/databases/(default)/documents/books
```

- If the page shows a `403` error that says `Cloud Firestore API has not been used in project your-project-id`, the placeholder is still in the URL: Firestore went looking for a project whose ID is literally `your-project-id`. The same mistake in `config.ts` makes the app show its error screen.
- The browser does a `GET` and shows the JSON. One document looks like this:

```json
{
  "name": "projects/reading-list-1a2b3/databases/(default)/documents/books/EDt3mPzkcwlGMDYGaaro",
  "fields": {
    "title": { "stringValue": "The Hobbit" },
    "author": { "stringValue": "J.R.R. Tolkien" },
    "pages": { "integerValue": "310" }
  },
  "createTime": "2026-11-19T19:17:02.471219Z",
  "updateTime": "2026-11-19T19:17:02.471219Z"
}
```

- Three things to notice:
  - The id is not a field of its own. It is the last part of `name`, after the final `/`.
  - Every value is wrapped in an object that says its type: `stringValue`, `integerValue`, `doubleValue` (a number with decimals), `booleanValue`.
  - The whole number `310` arrives as the **string** `"310"`. An int64 can be bigger than a JavaScript number can hold exactly, so Firestore sends it as text. The app has to turn it back into a number.
- The list is in the `documents` array of the response. A collection with no documents comes back as `{}`, with no `documents` at all.

### Step 2 - Fetch and Unwrap the Books

- The screens should work with plain books (`{ id, title, author, pages }`), not Firestore's wrapped shape. One file does the unwrapping, so no screen ever sees it. Create `src/firebase/books.ts`.

```ts
import { FIRESTORE_URL } from "./config";

export interface Book {
  id: string;
  title: string;
  author: string;
  pages: number;
}

// What Firestore sends for one document. Every field is wrapped in an
// object that names its type, and whole numbers arrive as strings.
interface FirestoreDocument {
  name: string;
  fields?: {
    title?: { stringValue: string };
    author?: { stringValue: string };
    pages?: { integerValue: string };
  };
}

// Unwrap one Firestore document into a plain Book.
function toBook(doc: FirestoreDocument): Book {
  const fields = doc.fields ?? {};
  return {
    id: doc.name.split("/").pop()!,
    title: fields.title?.stringValue ?? "Untitled",
    author: fields.author?.stringValue ?? "Unknown author",
    pages: Number(fields.pages?.integerValue ?? 0),
  };
}

// GET the whole collection.
export async function getBooks(): Promise<Book[]> {
  const res = await fetch(`${FIRESTORE_URL}/books`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const data = await res.json();
  // An empty collection comes back as {}, with no documents array.
  const docs: FirestoreDocument[] = data.documents ?? [];
  return docs.map(toBook).sort((a, b) => a.title.localeCompare(b.title));
}

// GET one document. Firestore answers 404 when there is no such id.
export async function getBook(id: string): Promise<Book | null> {
  const res = await fetch(`${FIRESTORE_URL}/books/${id}`);
  if (res.status === 404) return null;
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return toBook(await res.json());
}
```

- `Book` is the plain shape the screens use. `FirestoreDocument` describes what Firestore sends, so TypeScript can check the unwrapping. Its fields are optional (`?`) because a document made by hand in the console might be missing one.
- `toBook` does the unwrapping: the id from the end of `name`, each value out of its wrapper, and `Number(...)` for the page count. `??` supplies a default when a field is missing, so one bad document does not crash the list.
- `getBooks` is the Week 5 fetch: check `res.ok`, throw so the screen can show an error, then turn the JSON into books. `data.documents ?? []` handles the empty collection. Firestore returns documents in id order, which looks random, so it sorts by title.
- `getBook` fetches one document by id. Firestore answers **404** for an id that does not exist, so it returns `null` instead of throwing, and the screen can say so.

### Step 3 - Show the List

- More than one screen needs the books (the list now, the add screen later), so they go in a Jotai atom. Create a folder `store` in `src` and a file `src/store/books.ts`.

```ts
import { atom } from "jotai";
import { Book, getBooks } from "@/firebase/books";

// The books every screen shows.
export const booksAtom = atom<Book[]>([]);

// A write-only atom. Calling its setter fetches the books and stores them in booksAtom.
export const loadBooksAtom = atom(null, async (_get, set) => {
  set(booksAtom, await getBooks());
});
```

- `booksAtom` is an ordinary atom holding the list.
- `loadBooksAtom` is new: a **write-only atom**. Its first argument is `null` because it holds no value of its own. The second is a function that runs whenever someone calls its setter: it fetches the books and `set`s them into `booksAtom`. Any screen can now load the books with one call, and every screen that reads `booksAtom` updates.
- The `_get` parameter is the atom's way to read other atoms. We do not need it, and the underscore says so.
- Replace `src/app/index.tsx` with the list.

```tsx
import { useEffect, useState } from "react";
import {
  ActivityIndicator,
  Button,
  FlatList,
  Pressable,
  StyleSheet,
  Text,
  View,
} from "react-native";
import { Link } from "expo-router";
import { useAtomValue, useSetAtom } from "jotai";
import { booksAtom, loadBooksAtom } from "@/store/books";

export default function BooksScreen() {
  const books = useAtomValue(booksAtom);
  const loadBooks = useSetAtom(loadBooksAtom);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  async function load() {
    try {
      await loadBooks();
      setError(null);
    } catch {
      setError("Could not load the books. Check your connection and try again.");
    }
  }

  useEffect(() => {
    load().finally(() => setLoading(false));
  }, []);

  function retry() {
    setLoading(true);
    load().finally(() => setLoading(false));
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{error}</Text>
        <Button title="Try Again" onPress={retry} />
      </View>
    );
  }

  return (
    <FlatList
      contentContainerStyle={styles.list}
      data={books}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <Link href={`/books/${item.id}`} asChild>
          <Pressable style={styles.card}>
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.author}>{item.author}</Text>
          </Pressable>
        </Link>
      )}
      ListEmptyComponent={<Text style={styles.empty}>No books yet.</Text>}
    />
  );
}

const styles = StyleSheet.create({
  center: { flex: 1, alignItems: "center", justifyContent: "center", gap: 12 },
  error: { color: "red", textAlign: "center", padding: 20 },
  list: { padding: 16 },
  card: { backgroundColor: "#fff", padding: 16, borderRadius: 12, marginBottom: 12 },
  title: { fontSize: 16, fontWeight: "600" },
  author: { fontSize: 14, color: "#8e8e93", marginTop: 4 },
  empty: { textAlign: "center", color: "#8e8e93", marginTop: 40 },
});
```

- `useSetAtom(loadBooksAtom)` gives the setter. Calling `loadBooks()` runs the fetch, and because the function inside the atom is `async`, it returns a promise you can `await`.
- The loading, error and retry code is the Week 5 pattern, unchanged. Only what `load` calls is new.
- Replace `src/app/_layout.tsx` to give the screens titles.

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: "Reading List" }} />
      <Stack.Screen name="books/[id]" options={{ title: "Book" }} />
    </Stack>
  );
}
```

- The list shows your books, sorted by title. Add a book in the console and press **r**: it appears.

### Step 4 - One Book

- Create a folder `books` in `src/app`, and in it a file `[id].tsx`.

```tsx
import { useEffect, useState } from "react";
import { ActivityIndicator, StyleSheet, Text, View } from "react-native";
import { Stack, useLocalSearchParams } from "expo-router";
import { Book, getBook } from "@/firebase/books";

export default function BookScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const [book, setBook] = useState<Book | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function load() {
      try {
        const found = await getBook(id);
        if (found) {
          setBook(found);
        } else {
          setError(`No book with id ${id}.`);
        }
      } catch {
        setError("Could not load the book.");
      }
    }
    load().finally(() => setLoading(false));
  }, [id]);

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  if (error || !book) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{error}</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Stack.Screen options={{ title: book.title }} />
      <Text style={styles.title}>{book.title}</Text>
      <Text style={styles.author}>by {book.author}</Text>
      <Text style={styles.pages}>{book.pages} pages</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  center: { flex: 1, alignItems: "center", justifyContent: "center" },
  error: { color: "red", textAlign: "center", padding: 20 },
  container: { flex: 1, padding: 24, gap: 8 },
  title: { fontSize: 24, fontWeight: "700" },
  author: { fontSize: 18, color: "#48484a" },
  pages: { fontSize: 16, color: "#8e8e93" },
});
```

- This screen does not look the book up in the atom. It asks Firestore for that one document with `getBook(id)`, which is a `GET` on the document's URL. That way it also works when it is opened straight from a link, before the list has loaded.
- The effect depends on `[id]`, so it fetches again if the screen is reused for another book.

### What to Show

1. Tap a book. The details load from Firestore, and the header shows the title.
2. In the browser, change the address to `/books/nope`. Firestore answers 404 and the screen says `No book with id nope.`
3. In the console, change a book's `pages` and press **r** in the terminal. The app shows the new number. The database is the single source of truth.
4. In `src/firebase/books.ts`, misspell the collection as `/bookz` in `getBooks`. The list says `No books yet.`, **not** an error: in test mode, a collection that does not exist is just an empty one. Then change `PROJECT_ID` to a project that does not exist: now Firestore refuses, `res.ok` is false, and the error screen appears. Put both back.

## Example: Signing In

- Firebase Authentication stores the accounts and checks the passwords. The app sends an email and password and gets back an **ID token**, a long string that proves who the user is.

### Turn On Email Sign-In

1. In the console's left menu, open **Security**, then **Authentication**, and click **Get started**.
2. Under **Sign-in method**, choose **Email/Password**, turn on the first switch (**Email/Password**) and click **Save**. Leave **Email link (passwordless sign-in)** off.

### Step 1 - Sign Up and Sign In Requests

- Create `src/firebase/auth.ts`.

```ts
import { API_KEY, AUTH_URL } from "./config";

export interface User {
  email: string;
  // Proof of who the user is. It is good for one hour.
  idToken: string;
}

// Sign up and sign in send the same body to two different addresses.
async function sendCredentials(
  action: "signUp" | "signInWithPassword",
  email: string,
  password: string
): Promise<User> {
  const res = await fetch(`${AUTH_URL}:${action}?key=${API_KEY}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password, returnSecureToken: true }),
  });
  const data = await res.json();
  // On failure Firebase says why, for example EMAIL_EXISTS.
  if (!res.ok) throw new Error(data.error.message);
  return { email: data.email, idToken: data.idToken };
}

export function signUp(email: string, password: string) {
  return sendCredentials("signUp", email, password);
}

export function signIn(email: string, password: string) {
  return sendCredentials("signInWithPassword", email, password);
}
```

- Signing up and signing in are both a `POST` with the same body. Only the end of the URL differs: `accounts:signUp` or `accounts:signInWithPassword`. `?key=` adds your API key, which Authentication requires.
- `returnSecureToken: true` asks for the ID token in the response. The response also has a refresh token and a user id, which we do not use today.
- When Firebase says no, the reason is in `data.error.message`: `EMAIL_EXISTS`, `INVALID_EMAIL`, `WEAK_PASSWORD : Password should be at least 6 characters`, or `INVALID_LOGIN_CREDENTIALS` for a wrong email or password. Firebase does not say which one was wrong, so nobody can use the form to find out who has an account.
- The ID token expires after one hour. Today we simply sign in again. A finished app would use the refresh token to get a new one.
- Create `src/store/user.ts`, the atom that says who is signed in.

```ts
import { atom } from "jotai";
import { User } from "@/firebase/auth";

// The signed-in user, or null when nobody is signed in.
export const userAtom = atom<User | null>(null);
```

### Step 2 - The Sign-In Screen

- Create `src/app/sign-in.tsx`.

```tsx
import { useState } from "react";
import { Button, StyleSheet, Text, TextInput, View } from "react-native";
import { useSetAtom } from "jotai";
import { User, signIn, signUp } from "@/firebase/auth";
import { userAtom } from "@/store/user";

export default function SignInScreen() {
  const setUser = useSetAtom(userAtom);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [busy, setBusy] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // send is signIn or signUp. Both take the same values and return a User.
  async function submit(send: (email: string, password: string) => Promise<User>) {
    setBusy(true);
    setError(null);
    try {
      setUser(await send(email.trim(), password));
    } catch (e) {
      setError((e as Error).message);
      setBusy(false);
    }
  }

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        autoCapitalize="none"
        keyboardType="email-address"
      />
      <TextInput
        style={styles.input}
        placeholder="Password (6 characters or more)"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      {error && <Text style={styles.error}>{error}</Text>}
      <Button title="Sign In" onPress={() => submit(signIn)} disabled={busy} />
      <Button title="Create Account" onPress={() => submit(signUp)} disabled={busy} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24, gap: 12, justifyContent: "center" },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 8, padding: 12, backgroundColor: "#fff" },
  error: { color: "red" },
});
```

- `submit` takes the request function as a parameter, so both buttons share one function: `submit(signIn)` and `submit(signUp)`. A function is a value like any other.
- On success it stores the user in the atom. It does not navigate anywhere. The layout takes care of that in the next step.
- Errors go in a `Text`, not an `Alert`, because `Alert.alert` does nothing in the browser.
- `(e as Error).message` tells TypeScript the caught value is an `Error`. A `catch` can receive anything, so TypeScript will not assume.

### Step 3 - Protect the Screens

- Signed-out users should only see the sign-in screen, and signed-in users should never see it. Replace `src/app/_layout.tsx`.

```tsx
import { Stack } from "expo-router";
import { useAtomValue } from "jotai";
import { userAtom } from "@/store/user";

export default function RootLayout() {
  const user = useAtomValue(userAtom);

  return (
    <Stack>
      <Stack.Protected guard={user !== null}>
        <Stack.Screen name="index" options={{ title: "Reading List" }} />
        <Stack.Screen name="books/[id]" options={{ title: "Book" }} />
      </Stack.Protected>
      <Stack.Protected guard={user === null}>
        <Stack.Screen name="sign-in" options={{ title: "Sign In" }} />
      </Stack.Protected>
    </Stack>
  );
}
```

- `Stack.Protected` hides the screens inside it while its `guard` is `false`. Going to a hidden screen, by a link or by typing its address, sends you to the first screen you are allowed to see instead.
- The layout reads `userAtom`, so it re-renders when the user signs in or out. Signing in makes the first guard `true` and the second `false`, so the app moves from the sign-in screen to the list by itself. Signing out does the reverse.
- The Docusaurus Firebase page does the same job with `useSegments`, `router.replace` and an effect. `Stack.Protected` is the newer, shorter way, built into Expo Router.
- This is the first time the layout imports `jotai`, so you may get the red `[Refresh] Expected to find the updated module.` screen here. Reload the page, or see the tip in Expo Project Setup. A reload signs you out, so sign in again.

### Step 4 - Sign Out

- In `src/app/index.tsx`, replace **all** the import lines at the top with these. Two lines change: the `jotai` import gains `useAtom`, and the last line is new.

```tsx
import { useEffect, useState } from "react";
import {
  ActivityIndicator,
  Button,
  FlatList,
  Pressable,
  StyleSheet,
  Text,
  View,
} from "react-native";
import { Link } from "expo-router";
import { useAtom, useAtomValue, useSetAtom } from "jotai";
import { booksAtom, loadBooksAtom } from "@/store/books";
import { userAtom } from "@/store/user";
```

- Inside `BooksScreen`, under the `loadBooks` line, read the user and get its setter.

```tsx
const [user, setUser] = useAtom(userAtom);
```

- Add a footer to the `FlatList`, under `ListEmptyComponent`.

```tsx
ListFooterComponent={
  <View style={styles.footer}>
    <Text style={styles.signedIn}>Signed in as {user?.email}</Text>
    <Button title="Sign Out" onPress={() => setUser(null)} />
  </View>
}
```

- Add the two styles.

```tsx
footer: { marginTop: 24, gap: 8 },
signedIn: { textAlign: "center", color: "#8e8e93" },
```

- Signing out is only `setUser(null)`. There is no request to Firebase: the app forgets the token, and the layout's guards do the rest.

### What to Show

1. Open the app: it starts on **Sign In**, even though `index` is the first file.
2. Press **Sign In** with an email that has no account: `INVALID_LOGIN_CREDENTIALS`. Press **Create Account** with a 3-character password: `WEAK_PASSWORD`.
3. Create an account with a real password. The app goes to the list on its own.
4. In the console, open **Authentication**, **Users**: the new account is there. The password is not shown anywhere, not even to you.
5. Press **Sign Out**, then sign in again with **Sign In**.
6. Sign in, then reload the browser page. You are signed out: the user lives in an atom, in memory, and a reload starts the app fresh.

## Example: Adding and Deleting

- Writing to Firestore is the same `fetch` with a different method, a body, and the user's token.

### Step 1 - Add a Book

- In `src/firebase/books.ts`, add the type for a book that has no id yet, under `Book`.

```ts
// A book before Firestore has given it an id.
export interface NewBook {
  title: string;
  author: string;
  pages: number;
}
```

- Add the reverse of `toBook`, under `toBook`. Firestore wants the same wrapped shape back.

```ts
// Wrap a book's values the way Firestore expects them.
function toFields(book: NewBook) {
  return {
    title: { stringValue: book.title },
    author: { stringValue: book.author },
    pages: { integerValue: String(book.pages) },
  };
}
```

- Add `addBook` at the bottom of the file.

```ts
// POST to the collection. Firestore picks the id and sends the new document back.
export async function addBook(book: NewBook, idToken: string): Promise<Book> {
  const res = await fetch(`${FIRESTORE_URL}/books`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${idToken}`,
    },
    body: JSON.stringify({ fields: toFields(book) }),
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return toBook(await res.json());
}
```

- `POST` to the **collection** URL creates a document with a new id. The response is the new document, in the same wrapped shape as a `GET`, so `toBook` unwraps it and we learn its id.
- `Authorization: Bearer ...` sends the user's ID token with the request. Test mode does not check it yet. The last step of today makes Firestore require it.
- Create `src/app/add.tsx`.

```tsx
import { useState } from "react";
import { Button, StyleSheet, Text, TextInput, View } from "react-native";
import { router } from "expo-router";
import { useAtomValue, useSetAtom } from "jotai";
import { addBook } from "@/firebase/books";
import { booksAtom } from "@/store/books";
import { userAtom } from "@/store/user";

export default function AddBookScreen() {
  const user = useAtomValue(userAtom);
  const setBooks = useSetAtom(booksAtom);
  const [title, setTitle] = useState("");
  const [author, setAuthor] = useState("");
  const [pages, setPages] = useState("");
  const [saving, setSaving] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function save() {
    const pageCount = Number(pages);
    if (title.trim() === "" || author.trim() === "" || !Number.isInteger(pageCount) || pageCount < 1) {
      setError("Enter a title, an author and a whole number of pages.");
      return;
    }
    setSaving(true);
    try {
      const book = await addBook(
        { title: title.trim(), author: author.trim(), pages: pageCount },
        user!.idToken
      );
      setBooks((prev) => [...prev, book].sort((a, b) => a.title.localeCompare(b.title)));
      router.back();
    } catch {
      setError("Could not save the book. Try again.");
      setSaving(false);
    }
  }

  return (
    <View style={styles.container}>
      <TextInput style={styles.input} placeholder="Title" value={title} onChangeText={setTitle} />
      <TextInput style={styles.input} placeholder="Author" value={author} onChangeText={setAuthor} />
      <TextInput
        style={styles.input}
        placeholder="Pages"
        value={pages}
        onChangeText={setPages}
        keyboardType="number-pad"
      />
      {error && <Text style={styles.error}>{error}</Text>}
      <Button title={saving ? "Saving..." : "Save"} onPress={save} disabled={saving} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24, gap: 12 },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 8, padding: 12, backgroundColor: "#fff" },
  error: { color: "red" },
});
```

- `user!.idToken`: the `!` tells TypeScript that `user` is not `null`. That is safe here, because `Stack.Protected` only lets signed-in users reach this screen.
- After a successful save, the new book goes into `booksAtom` with the updater form, and the list re-sorts. There is no need to fetch the whole list again: we already have the book Firestore sent back.
- A `TextInput` always gives a string, so `pages` is converted with `Number` and checked before anything is sent.
- In `src/app/_layout.tsx`, replace **all** the import lines at the top with these. The first line is new and `router` is added to the second.

```tsx
import { Button } from "react-native";
import { Stack, router } from "expo-router";
import { useAtomValue } from "jotai";
import { userAtom } from "@/store/user";
```

- Then give the list an **Add** button in its header, and add the new screen to the first `Stack.Protected`.

```tsx
<Stack.Screen
  name="index"
  options={{
    title: "Reading List",
    headerRight: () => <Button title="Add" onPress={() => router.push("/add")} />,
  }}
/>
<Stack.Screen name="books/[id]" options={{ title: "Book" }} />
<Stack.Screen name="add" options={{ title: "Add a Book" }} />
```

- Add a book. It appears in the list, and in the console under `books`, with an id Firestore chose.

### Step 2 - Delete a Book

- Add `deleteBook` at the bottom of `src/firebase/books.ts`.

```ts
// DELETE one document.
export async function deleteBook(id: string, idToken: string): Promise<void> {
  const res = await fetch(`${FIRESTORE_URL}/books/${id}`, {
    method: "DELETE",
    headers: { Authorization: `Bearer ${idToken}` },
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
}
```

- `DELETE` goes to the **document** URL and has no body.
- In `src/app/books/[id].tsx`, replace **all** the import lines at the top with these. Keep the first line, the `react` import: the screen still uses `useEffect` and `useState`.

```tsx
import { useEffect, useState } from "react";
import { ActivityIndicator, Button, StyleSheet, Text, View } from "react-native";
import { Stack, router, useLocalSearchParams } from "expo-router";
import { useAtomValue, useSetAtom } from "jotai";
import { Book, deleteBook, getBook } from "@/firebase/books";
import { booksAtom } from "@/store/books";
import { userAtom } from "@/store/user";
```

- Inside `BookScreen`, under the `id` line, read the user and get the books setter.

```tsx
const user = useAtomValue(userAtom);
const setBooks = useSetAtom(booksAtom);
```

- Add a `remove` function above the `if (loading)` check.

```tsx
async function remove() {
  try {
    await deleteBook(id, user!.idToken);
    setBooks((prev) => prev.filter((b) => b.id !== id));
    router.back();
  } catch {
    setError("Could not delete the book.");
  }
}
```

- Add the button under the page count, and give the `pages` style some room under it.

```tsx
<Text style={styles.pages}>{book.pages} pages</Text>
<Button title="Delete" color="red" onPress={remove} />
```

```tsx
pages: { fontSize: 16, color: "#8e8e93", marginBottom: 16 },
```

- The book disappears from Firestore, then from the atom, and the list updates as the screen goes back.

### Step 3 - Only Signed-In Users Can Write

- Test mode lets anyone write. Replace it with rules that let anyone read the books, but only a signed-in user change them. In the console, open **Firestore**, then the **Rules** tab, replace the rules with these, and click **Publish**.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /books/{bookId} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

- `match /books/{bookId}` applies to every document in `books`. `write` covers create, update and delete.
- `request.auth` is filled in from the `Authorization: Bearer` token. A request without a valid token has `request.auth == null`, and Firestore answers **403 Forbidden**.
- Rules run on Google's servers. Code in the app can be changed by anyone who has a copy of it. The rules cannot, which is why they, and not the API key, are what protect the data.
- The rules also replace test mode's 30-day limit, so the database keeps working after the test period.

### What to Show

1. Add a book and delete it. Both still work: the app sends the token.
2. In `addBook`, comment out the `Authorization` line and add a book. Firestore answers 403, `res.ok` is false, and the screen says `Could not save the book.` Put the line back.
3. Open the collection URL from the start of today in a browser tab. Reading still works without signing in, because the rules allow it.

### Finished Code

- Compare with these if something does not work.
- `src/firebase/config.ts`

```ts
// From the Firebase console: Settings, General, Your apps.
export const PROJECT_ID = "your-project-id";
export const API_KEY = "your-web-api-key";

// Every Firestore request starts here, then adds a collection and maybe a document id.
export const FIRESTORE_URL = `https://firestore.googleapis.com/v1/projects/${PROJECT_ID}/databases/(default)/documents`;

// Sign up and sign in go to Firebase Authentication, a different Google server.
export const AUTH_URL = "https://identitytoolkit.googleapis.com/v1/accounts";
```

- `src/firebase/books.ts`

```ts
import { FIRESTORE_URL } from "./config";

export interface Book {
  id: string;
  title: string;
  author: string;
  pages: number;
}

// A book before Firestore has given it an id.
export interface NewBook {
  title: string;
  author: string;
  pages: number;
}

// What Firestore sends for one document. Every field is wrapped in an
// object that names its type, and whole numbers arrive as strings.
interface FirestoreDocument {
  name: string;
  fields?: {
    title?: { stringValue: string };
    author?: { stringValue: string };
    pages?: { integerValue: string };
  };
}

// Unwrap one Firestore document into a plain Book.
function toBook(doc: FirestoreDocument): Book {
  const fields = doc.fields ?? {};
  return {
    id: doc.name.split("/").pop()!,
    title: fields.title?.stringValue ?? "Untitled",
    author: fields.author?.stringValue ?? "Unknown author",
    pages: Number(fields.pages?.integerValue ?? 0),
  };
}

// Wrap a book's values the way Firestore expects them.
function toFields(book: NewBook) {
  return {
    title: { stringValue: book.title },
    author: { stringValue: book.author },
    pages: { integerValue: String(book.pages) },
  };
}

// GET the whole collection.
export async function getBooks(): Promise<Book[]> {
  const res = await fetch(`${FIRESTORE_URL}/books`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const data = await res.json();
  // An empty collection comes back as {}, with no documents array.
  const docs: FirestoreDocument[] = data.documents ?? [];
  return docs.map(toBook).sort((a, b) => a.title.localeCompare(b.title));
}

// GET one document. Firestore answers 404 when there is no such id.
export async function getBook(id: string): Promise<Book | null> {
  const res = await fetch(`${FIRESTORE_URL}/books/${id}`);
  if (res.status === 404) return null;
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return toBook(await res.json());
}

// POST to the collection. Firestore picks the id and sends the new document back.
export async function addBook(book: NewBook, idToken: string): Promise<Book> {
  const res = await fetch(`${FIRESTORE_URL}/books`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${idToken}`,
    },
    body: JSON.stringify({ fields: toFields(book) }),
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return toBook(await res.json());
}

// DELETE one document.
export async function deleteBook(id: string, idToken: string): Promise<void> {
  const res = await fetch(`${FIRESTORE_URL}/books/${id}`, {
    method: "DELETE",
    headers: { Authorization: `Bearer ${idToken}` },
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
}
```

- `src/firebase/auth.ts`

```ts
import { API_KEY, AUTH_URL } from "./config";

export interface User {
  email: string;
  // Proof of who the user is. It is good for one hour.
  idToken: string;
}

// Sign up and sign in send the same body to two different addresses.
async function sendCredentials(
  action: "signUp" | "signInWithPassword",
  email: string,
  password: string
): Promise<User> {
  const res = await fetch(`${AUTH_URL}:${action}?key=${API_KEY}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password, returnSecureToken: true }),
  });
  const data = await res.json();
  // On failure Firebase says why, for example EMAIL_EXISTS.
  if (!res.ok) throw new Error(data.error.message);
  return { email: data.email, idToken: data.idToken };
}

export function signUp(email: string, password: string) {
  return sendCredentials("signUp", email, password);
}

export function signIn(email: string, password: string) {
  return sendCredentials("signInWithPassword", email, password);
}
```

- `src/store/books.ts`

```ts
import { atom } from "jotai";
import { Book, getBooks } from "@/firebase/books";

// The books every screen shows.
export const booksAtom = atom<Book[]>([]);

// A write-only atom. Calling its setter fetches the books and stores them in booksAtom.
export const loadBooksAtom = atom(null, async (_get, set) => {
  set(booksAtom, await getBooks());
});
```

- `src/store/user.ts`

```ts
import { atom } from "jotai";
import { User } from "@/firebase/auth";

// The signed-in user, or null when nobody is signed in.
export const userAtom = atom<User | null>(null);
```

- `src/app/_layout.tsx`

```tsx
import { Button } from "react-native";
import { Stack, router } from "expo-router";
import { useAtomValue } from "jotai";
import { userAtom } from "@/store/user";

export default function RootLayout() {
  const user = useAtomValue(userAtom);

  return (
    <Stack>
      <Stack.Protected guard={user !== null}>
        <Stack.Screen
          name="index"
          options={{
            title: "Reading List",
            headerRight: () => <Button title="Add" onPress={() => router.push("/add")} />,
          }}
        />
        <Stack.Screen name="books/[id]" options={{ title: "Book" }} />
        <Stack.Screen name="add" options={{ title: "Add a Book" }} />
      </Stack.Protected>
      <Stack.Protected guard={user === null}>
        <Stack.Screen name="sign-in" options={{ title: "Sign In" }} />
      </Stack.Protected>
    </Stack>
  );
}
```

- `src/app/index.tsx`

```tsx
import { useEffect, useState } from "react";
import {
  ActivityIndicator,
  Button,
  FlatList,
  Pressable,
  StyleSheet,
  Text,
  View,
} from "react-native";
import { Link } from "expo-router";
import { useAtom, useAtomValue, useSetAtom } from "jotai";
import { booksAtom, loadBooksAtom } from "@/store/books";
import { userAtom } from "@/store/user";

export default function BooksScreen() {
  const books = useAtomValue(booksAtom);
  const loadBooks = useSetAtom(loadBooksAtom);
  const [user, setUser] = useAtom(userAtom);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  async function load() {
    try {
      await loadBooks();
      setError(null);
    } catch {
      setError("Could not load the books. Check your connection and try again.");
    }
  }

  useEffect(() => {
    load().finally(() => setLoading(false));
  }, []);

  function retry() {
    setLoading(true);
    load().finally(() => setLoading(false));
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{error}</Text>
        <Button title="Try Again" onPress={retry} />
      </View>
    );
  }

  return (
    <FlatList
      contentContainerStyle={styles.list}
      data={books}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <Link href={`/books/${item.id}`} asChild>
          <Pressable style={styles.card}>
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.author}>{item.author}</Text>
          </Pressable>
        </Link>
      )}
      ListEmptyComponent={<Text style={styles.empty}>No books yet.</Text>}
      ListFooterComponent={
        <View style={styles.footer}>
          <Text style={styles.signedIn}>Signed in as {user?.email}</Text>
          <Button title="Sign Out" onPress={() => setUser(null)} />
        </View>
      }
    />
  );
}

const styles = StyleSheet.create({
  center: { flex: 1, alignItems: "center", justifyContent: "center", gap: 12 },
  error: { color: "red", textAlign: "center", padding: 20 },
  list: { padding: 16 },
  card: { backgroundColor: "#fff", padding: 16, borderRadius: 12, marginBottom: 12 },
  title: { fontSize: 16, fontWeight: "600" },
  author: { fontSize: 14, color: "#8e8e93", marginTop: 4 },
  empty: { textAlign: "center", color: "#8e8e93", marginTop: 40 },
  footer: { marginTop: 24, gap: 8 },
  signedIn: { textAlign: "center", color: "#8e8e93" },
});
```

- `src/app/books/[id].tsx`

```tsx
import { useEffect, useState } from "react";
import { ActivityIndicator, Button, StyleSheet, Text, View } from "react-native";
import { Stack, router, useLocalSearchParams } from "expo-router";
import { useAtomValue, useSetAtom } from "jotai";
import { Book, deleteBook, getBook } from "@/firebase/books";
import { booksAtom } from "@/store/books";
import { userAtom } from "@/store/user";

export default function BookScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const user = useAtomValue(userAtom);
  const setBooks = useSetAtom(booksAtom);
  const [book, setBook] = useState<Book | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function load() {
      try {
        const found = await getBook(id);
        if (found) {
          setBook(found);
        } else {
          setError(`No book with id ${id}.`);
        }
      } catch {
        setError("Could not load the book.");
      }
    }
    load().finally(() => setLoading(false));
  }, [id]);

  async function remove() {
    try {
      await deleteBook(id, user!.idToken);
      setBooks((prev) => prev.filter((b) => b.id !== id));
      router.back();
    } catch {
      setError("Could not delete the book.");
    }
  }

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  if (error || !book) {
    return (
      <View style={styles.center}>
        <Text style={styles.error}>{error}</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Stack.Screen options={{ title: book.title }} />
      <Text style={styles.title}>{book.title}</Text>
      <Text style={styles.author}>by {book.author}</Text>
      <Text style={styles.pages}>{book.pages} pages</Text>
      <Button title="Delete" color="red" onPress={remove} />
    </View>
  );
}

const styles = StyleSheet.create({
  center: { flex: 1, alignItems: "center", justifyContent: "center" },
  error: { color: "red", textAlign: "center", padding: 20 },
  container: { flex: 1, padding: 24, gap: 8 },
  title: { fontSize: 24, fontWeight: "700" },
  author: { fontSize: 18, color: "#48484a" },
  pages: { fontSize: 16, color: "#8e8e93", marginBottom: 16 },
});
```

- `src/app/add.tsx`

```tsx
import { useState } from "react";
import { Button, StyleSheet, Text, TextInput, View } from "react-native";
import { router } from "expo-router";
import { useAtomValue, useSetAtom } from "jotai";
import { addBook } from "@/firebase/books";
import { booksAtom } from "@/store/books";
import { userAtom } from "@/store/user";

export default function AddBookScreen() {
  const user = useAtomValue(userAtom);
  const setBooks = useSetAtom(booksAtom);
  const [title, setTitle] = useState("");
  const [author, setAuthor] = useState("");
  const [pages, setPages] = useState("");
  const [saving, setSaving] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function save() {
    const pageCount = Number(pages);
    if (title.trim() === "" || author.trim() === "" || !Number.isInteger(pageCount) || pageCount < 1) {
      setError("Enter a title, an author and a whole number of pages.");
      return;
    }
    setSaving(true);
    try {
      const book = await addBook(
        { title: title.trim(), author: author.trim(), pages: pageCount },
        user!.idToken
      );
      setBooks((prev) => [...prev, book].sort((a, b) => a.title.localeCompare(b.title)));
      router.back();
    } catch {
      setError("Could not save the book. Try again.");
      setSaving(false);
    }
  }

  return (
    <View style={styles.container}>
      <TextInput style={styles.input} placeholder="Title" value={title} onChangeText={setTitle} />
      <TextInput style={styles.input} placeholder="Author" value={author} onChangeText={setAuthor} />
      <TextInput
        style={styles.input}
        placeholder="Pages"
        value={pages}
        onChangeText={setPages}
        keyboardType="number-pad"
      />
      {error && <Text style={styles.error}>{error}</Text>}
      <Button title={saving ? "Saving..." : "Save"} onPress={save} disabled={saving} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24, gap: 12 },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 8, padding: 12, backgroundColor: "#fff" },
  error: { color: "red" },
});
```

- `src/app/sign-in.tsx`

```tsx
import { useState } from "react";
import { Button, StyleSheet, Text, TextInput, View } from "react-native";
import { useSetAtom } from "jotai";
import { User, signIn, signUp } from "@/firebase/auth";
import { userAtom } from "@/store/user";

export default function SignInScreen() {
  const setUser = useSetAtom(userAtom);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [busy, setBusy] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // send is signIn or signUp. Both take the same values and return a User.
  async function submit(send: (email: string, password: string) => Promise<User>) {
    setBusy(true);
    setError(null);
    try {
      setUser(await send(email.trim(), password));
    } catch (e) {
      setError((e as Error).message);
      setBusy(false);
    }
  }

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        autoCapitalize="none"
        keyboardType="email-address"
      />
      <TextInput
        style={styles.input}
        placeholder="Password (6 characters or more)"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      {error && <Text style={styles.error}>{error}</Text>}
      <Button title="Sign In" onPress={() => submit(signIn)} disabled={busy} />
      <Button title="Create Account" onPress={() => submit(signUp)} disabled={busy} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24, gap: 12, justifyContent: "center" },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 8, padding: 12, backgroundColor: "#fff" },
  error: { color: "red" },
});
```

## Firebase Checklist

- Firestore stores documents (records with named fields) in collections. Every collection and document has a URL.
- `GET` reads, `POST` to a collection creates, `PATCH` updates, `DELETE` deletes. All of it is `fetch`.
- Firestore wraps every value in its type (`stringValue`, `integerValue`, ...). Unwrap documents in one place so screens only see plain objects.
- A document's id is the end of its `name`. A whole number arrives as a string. An empty collection has no `documents` array.
- Keep the project ID and API key in one config file. The API key is not a secret.
- Sign up and sign in are `POST`s to Firebase Authentication. The ID token they return proves who the user is, for one hour.
- `Stack.Protected` with a `guard` shows each screen only to the right users.
- Send the token as `Authorization: Bearer ...`, and use security rules, not the app, to decide who may read and write.
- Never put real or personal data in a test-mode database.
