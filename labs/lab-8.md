# Lab 8 - Product Shop with Firebase

## A Note on AI

AI assistants are welcome in this course, but use them to get unstuck, not to do the work. Most of this lab follows a guide, and Parts 4 and 5 ask you to change code you did not write. That is exactly where pasting an AI answer goes wrong: it will happily rewrite files the rest of your app depends on. Read the guide's code before you paste it, and when something breaks, paste **your** file and the error and ask why. Never keep code you could not explain line by line to a classmate.

## Overview

You connect last week's **Product Shop** (Lab 7) to Firebase. The products move out of the app and into a Cloud Firestore database, and the shop gets a sign-in screen. You follow the [Getting Started with Firebase](https://seneca-ictoer.github.io/WEB530/I-Firebase/fb-getting-started) page in the course notes, then add two things it leaves out. Plan for about 60 to 75 minutes.

- Part 1 sets up Firebase in the console and puts your products in it.
- Parts 2 and 3 follow the guide: read the products from Firestore, then add sign-in.
- Part 4 adds a way to sign out.
- Part 5 shows an error when the products cannot load, instead of an empty shop.

**Start from your Lab 7 project.** Open the same folder in VS Code. If you no longer have it, finish Lab 7 first: this lab changes its files by path (`store/productsAtom.ts`, `app/(tabs)/products/index.tsx`, `app/_layout.tsx`).

The Week 10 notes built a different app (a Reading List) with the same ideas. Use them to understand what the guide's code is doing, not as code to copy: the file names, data and screens are different.

## Rules

- Keep the guide's file and folder names: `config/firebaseConfig.js`, `store/productsAtom.ts`, `store/authAtom.ts`, `app/auth.tsx`.
- The products live in Firestore and in `productsAtom` only. Delete the hard-coded product array from Lab 7. No screen keeps its own copy of the products.
- Keep your Lab 7 Settings screen (your name and the product count). The count must still come from `productsAtom`.
- Styles go in each file's `StyleSheet.create`. No inline `style={{ ... }}` objects.
- Never put real or personal data in your database. Test mode lets anyone who knows your project ID read and change it.

## Submission

- Submit to the `Lab 8` assessment in Blackboard:
  1. A zip of your code. Open your project folder, select the `app`, `store` and `config` folders and `app.json`, and compress them into one zip. Do **not** zip `node_modules`.
  2. Four screenshots, from your phone or the browser:
     - The Firestore console showing your `products` collection, with one document open so its fields show.
     - The Products tab listing your four products from Firestore.
     - The Settings tab from Part 4, showing who is signed in and the count of 4.
     - The error screen from Part 5.
- The lab is due Friday at 11:59 PM.
- Only students who participate in class and hand in a lab slip can receive marks for this lab, even though it is due after class.

## Part 1 - Firebase and Your Products (1 point)

Follow sections **1. Create a Firebase Account and Project** and **2. Set Up Cloud Firestore** in the guide.

- Create a Firebase project (Analytics off) and register a **Web** app. Keep the `projectId` and `apiKey` for Part 2.
- Create the Firestore database in **test mode**, with the database ID left as `(default)`.
- Add a `products` collection with **four** documents: your own two products from Lab 7 Part 5, and any two of the four products from the Lab 7 guide. Each document has an **Auto-ID** and three fields:
  - `name`, type **string**
  - `price`, type **double**
  - `description`, type **string**
- Check your data before writing any code. Open this URL in a browser tab, with your own project ID in place of `your-project-id`. You should see your four products in a `documents` array.

```
https://firestore.googleapis.com/v1/projects/your-project-id/databases/(default)/documents/products
```

Hint: A `403` error that mentions `your-project-id` means the placeholder is still in the URL. The project ID is usually your project's name plus a few extra characters: copy it from **Settings**, **General**.

## Part 2 - Products from Firestore (1 point)

Follow sections **3. Integrate Firebase into your Expo Router App** and **4. Modify the Product Shop to Read from Firestore**.

- `config/firebaseConfig.js` holds your own `projectId` and `apiKey`.
- `productsAtom` starts empty, and `fetchProductsAtom` fills it from Firestore. The hard-coded list is gone.
- The Products tab shows a spinner, then your four products with their prices.
- Tapping a product still opens its details: name, price and description.
- The Settings count reads `Products in the shop: 4`, without any change to `settings.tsx`.

Hint: Change a price in the Firestore console, then reload the app (press **r**). The new price appears: the database is now the only source of the products.

## Part 3 - Sign In (1 point)

Follow section **5. Add User Authentication**.

- Turn on **Email/Password** sign-in in the console.
- The app opens on the **Account Access** screen. The shop's tabs cannot be reached without signing in.
- **Create Account** with a new email and password takes you to the Products tab. **Sign In** does the same for an existing account.
- Your account appears in the console under **Authentication**, **Users**.

Hint: The guide's sign-in errors use `Alert.alert`, which does nothing in the browser. If a sign-in seems to do nothing on the web, the password may be under 6 characters or the email may already have an account. Test errors on your phone, or add a `console.log` next to each `Alert.alert`.

## Part 4 - Sign Out (1 point)

The guide never lets you sign out. Add it to the Settings tab.

- Under your Lab 7 lines, Settings shows `Signed in as` and the email the user signed in with, for example `Signed in as sam@example.com`.
- A **Sign Out** button returns the user to the **Account Access** screen. Back must not return to the shop.
- Signing in again, with the same or another account, works, and Settings shows the right email.

Requirements:

- The email comes from the sign-in response, not from the email text box: Firebase sends it back as `email`, next to `idToken`.
- Store it in Jotai. You can change `authAtom` to hold an object with both the email and the token, or add a second atom. Either way, the redirect in `app/_layout.tsx` must keep working.
- Signing out is a change to the atom only. There is no request to Firebase.

Hint: Look at what the redirect in `app/_layout.tsx` does when the token becomes `null`. You do not need to navigate anywhere yourself.

## Part 5 - Show When the Products Cannot Load (1 point)

Right now, if Firestore refuses the request (a wrong project ID, for example), `fetchProductsAtom` logs the problem to the console and the shop says `No hardware options available.` A user would think the shop is empty. Fix it.

- When the request fails, the Products tab shows `Could not load the products. Check your connection and try again.` and a **Try Again** button, instead of the list.
- **Try Again** shows the spinner and tries again. Once the problem is fixed, it shows the products.
- An empty `products` collection (no documents) still shows `No hardware options available.`: that is not an error.

Requirements:

- In `fetchProductsAtom`, check `response.ok` and `throw` when it is false, so the screen can catch the error. Remove the `try` / `catch` that hides it.
- Keep the error message in state on the Products screen, as in the Week 5 error and retry example.

Test it: change `projectId` in `config/firebaseConfig.js` to a project that does not exist, and reload the app. Take your screenshot of the error screen, put your project ID back, and press **Try Again**.

Hint: `fetch` only rejects when the network fails. A `403` or `404` is a successful response as far as `fetch` is concerned, which is why the guide's code never reaches its `catch`.
