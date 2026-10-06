# Lab 4 - Navigation

## A Note on AI

AI assistants are welcome in this course, but use them to get unstuck, not to do the work. Attempt every question yourself first. If you are stuck, paste **your** code and ask why it does not work or what concept you are missing, rather than asking for the answer. Never keep code you could not explain line by line to a classmate. If you cannot explain it, you cannot debug it or build on it, and it is not yours yet. This lab has a built-in check on that: a teammate is going to test your app while you watch, and you will test theirs. The point of this lab is not the finished Snack. It is being able to wire up navigation yourself, and to read someone else's app well enough to find what is wrong with it.

## Overview

This is a group lab with individual submissions, done entirely in class. You plan the app with your project group, build it on your own, then test each other's work.

- **Part A - Plan together.** Your group agrees on an app and draws its navigation map.
- **Part B - Build it yourself.** Every member builds the group's map in their own Snack. Your code is your own.
- **Part C - Test each other.** A teammate runs a set of checks on your app and records the results in your Snack. You do the same for them.

Your group is your project group. The list is in `project/project-groups.md` in the course repo. Because everyone in the group builds the same map, a teammate can test your app without reading your code.

This lab is also the first time you work with your project group, and getting to know each other is part of the point. **The app you build here is not your project.** Project brainstorming happens in formal sessions after the study break, so do not spend this lab pitching ideas or trying to pick the perfect topic.

### The Clock

The period runs from 11:40 to 1:25. These times are not suggestions:

| Time  | What                                                                 |
| ----- | -------------------------------------------------------------------- |
| 11:40 | Find your group, sit together, fork the starter Snack.               |
| 11:45 | Part A. Theme chosen by 11:47. Map drawn and photographed by 12:00.   |
| 12:00 | Part B. Build in order: B1, B2, B3, B4, then the bonus if time allows. |
| 12:55 | Part C. Stop building, whatever state your app is in, and start testing. |
| 1:20  | Save, check your link in a private window, submit.                   |

Part C needs two people. If you keep building past 12:55, your tester cannot finish either, and the lab is incomplete for both of you.

Checkpoints for Part B: B1 working by 12:15, B2 by 12:30, B3 by 12:40. If you are behind a checkpoint by more than five minutes, ask for help instead of pushing through alone.

## Submission

- Submit to the `Lab 4` assessment in Blackboard:
  - One photo or screenshot of your group's navigation map for Part A.
  - One Snack link for Parts B and C. After you save, the link looks like `https://snack.expo.dev/@yourname/web530-lab-4-...`.
- The lab is due at the end of class. Submit before you leave.
- This lab is marked **complete** or **incomplete**. Complete means all of the following: the map photo, a Snack where B1 to B4 work, a Part C block filled in by a teammate, and your name as the tester in a teammate's Snack. The bonus is optional.
- Only students who participate in class and hand in a lab slip can receive credit for this lab. If you miss the class, you are still encouraged to complete it as practice.
- Every member submits their own Snack. Two Snacks in the same group with the same code are both incomplete. Same map, same route names, same screens: yes. Same code: no.
- Before submitting, save the Snack one last time and open your link in a private browser window to confirm it loads without signing in.

## Setup

1. Sign in to Snack at https://snack.expo.dev.
2. Open the starter Snack: https://snack.expo.dev/@nick-romanidis/lab4-starter
3. Click **Fork** (top right) so you get your own copy. If you cannot see a Fork button, you are not signed in.
4. Rename your copy to `WEB530 Lab 4 - Your Name` (click the name at the top left).
5. Click **Save** so the Snack gets a permanent link.

The starter already has three files. Do not replace them.

- `package.json` has the week 4 dependencies. Leave it alone, and keep Snack on SDK 54 (the default). SDK 55 does not load `react-native-screens`, so nothing in this lab runs on it.
- `styles.js` has a centred `screen` style. Add to it if you want, but keep styling minimal. This lab is about navigation, not looks.
- `App.js` has the imports, the Part C comment block, a placeholder item array, empty screen components, and a tab navigator with one tab. Each placeholder has a comment saying which part fills it in.

The Web preview is enough for everything in this lab. Check the Android or iOS tab once at the end to see the native tab bar and header.

## Part A - Plan Together

With your group, pick a small app and draw its navigation map. Every member builds this same map in Part B, so agree on the details now. Do not use the fruit stand, the feed and account app, or anything else from the week 4 notes.

The theme does not matter and it is not your project. Give yourselves two minutes to choose, and if you cannot agree, take the first one on this list that nobody objects to:

- a campus food truck finder
- a board game collection
- a hiking trail guide
- a plant shop
- a podcast library

Spend the rest of Part A on the map, which is what you need for Part B, and on the names on it, which is who you are working with for the rest of the term.

The app has **three bottom tabs**. Your group names them, but they play these roles:

1. A **browse** tab. It contains a stack with two screens: a list of items and a details screen for one item.
2. A **saved** tab. It shows the items the user has saved from the details screen.
3. An **about** tab. It shows the app name, your group number and the names of every member.

Your map must show:

- The app name and the theme.
- The three tab names and the icon name for each, chosen from https://icons.expo.fyi/ (the Ionicons set).
- The route name of every screen, including the two inside the browse tab's stack.
- The item data: at least **four** items, each with an `id`, a `name`, and two more fields of your choice (for example `price` and `region`, or `players` and `minutes`).
- Which navigator holds which. A box inside a box is fine.

Draw it on paper, a whiteboard, or any drawing tool. Write the group number and the names of the members present on it. Everyone takes a photo, and everyone submits it.

## Part B - Build It Yourself

Build the group's map in your own Snack, in order. The code is yours: write it, do not receive it. Replace the placeholder items in the `ITEMS` array with your group's data.

### Part B1 - Tabs and a Nested Stack

- The tab navigator has the three tabs from your map, in that order. The app opens on the browse tab.
- Every tab has a `tabBarIcon` from `Ionicons`, using the icon names on your map. The icon takes the `color` and `size` the tab bar gives it.
- The browse tab's component, `BrowseStack` in the starter, is **not** a plain screen. It returns a stack navigator with the list screen and the details screen. Use the route names from your map.
- The **tab bar stays visible on the details screen**. If it disappears when you open an item, you nested the navigators the other way around.
- **Exactly one header is visible on every screen.** If the browse tab shows two stacked headers, hide one of them.

Hint: The course notes show both directions under **Nesting navigators**. The second example there, Messages and Contacts, is this exact pattern, including the double-header fix. Read it before you start B1.

### Part B2 - Params and the Header

- The list screen renders one `Button` per item in your array, using `.map()`. Pressing it opens the details screen with **two params**: the item's `id` and its `name`.
- The details screen looks the item up in the array by `id` and shows all of its fields.
- The details screen must **not crash** when it is opened with no params. Show the text `Item not found` instead. A quick way to check, if you have a minute: temporarily set `initialRouteName` on the stack to the details screen, confirm the message shows, then put it back.
- The header title of the details screen is the item's `name`, taken from the params. When there is no name, the title is `Details`.
- The list screen's header title is the app name, not the route name.

Hint: `options` can be a function that receives `route`. `route.params?.name ?? "Details"` handles the missing case.

### Part B3 - Push and Pop

Add two buttons to the details screen:

- `Next item` opens the details of the **next** item in your array (after the last item, wrap around to the first). It must open **on top of** the current details screen, so the header back arrow returns to the item you came from.
- `Back to list` returns to the list screen in **one press**, no matter how many `Next item` presses came before it.

Requirements:

- `Next item` uses `push`. Add a comment above it explaining, in your own words, what would happen if it used `navigate` instead.
- `Back to list` uses `popToTop`.

Hint: Find the current item's position in the array with `findIndex`, then use `%` to wrap.

### Part B4 - Saved Items and Shared State

The saved tab and the details screen share one list of saved ids through a Jotai atom, as in week 2. No saved data is passed as a prop or a param.

- One atom holds an array of saved `id`s. It starts empty.
- The details screen has a `headerRight` button. Its title is `Save` when the item is not saved and `Unsave` when it is. Pressing it toggles the item in the atom. Update the atom with the function form, `setSaved((prev) => ...)`.
- The saved tab lists the saved items by name, one `Text` or `Button` each.
- When nothing is saved, the saved tab shows the text `Nothing saved yet` and a `Browse` button that switches to the browse tab.
- The about tab shows the app name, your group number, and every member's name.

Requirements:

- The `Browse` button is its own component, not a screen. It gets the navigation object from the `useNavigation` hook.
- The `Save` and `Unsave` button is its own component that reads and updates the atom itself. The header only renders it and passes the item's `id`.

Hint: This is the Buy button from the notes, with an array instead of an object. `saved.includes(id)` tells you whether the item is saved. To toggle it: `prev.includes(id) ? prev.filter((s) => s !== id) : [...prev, id]`. For the saved list, `ITEMS.filter((item) => saved.includes(item.id))` gives you the items to show.

### Bonus - Open a Saved Item

Only start this if B1 to B4 work and it is not yet 12:55. It is not needed to complete the lab.

- Pressing a saved item on the saved tab opens **its details screen inside the browse tab**, with the same two params as the list screen. The browse tab becomes selected and the tab bar stays visible.

Hint: The saved tab is outside the browse tab's stack, so `navigate("Details", ...)` cannot find it. Name the tab first, then the screen inside it: `navigate("Browse", { screen: "Details", params: { id, name } })`. Use your own route names. The **Message Sam** button in the course notes' Messages and Contacts example does exactly this.

## Part C - Test Each Other

At 12:55, stop building. Pair up inside your group. If your group has an odd number, form a chain so everyone tests one app and everyone is tested once. If your tester is still building, test someone else in the group who is ready, and do not wait.

The tester opens the author's Snack, follows the checks in the comment block at the top of `App.js` **exactly as written**, and writes one word after each `Result:` line: `pass`, `fail`, or `not built`. The author watches and does not help. If a check fails, the author fixes it afterwards and writes down what changed.

The results live in that comment block. The tester types them in while sitting with the author, or sends them to the author to paste in. The block is already in the starter, so there is nothing to copy.

Requirements:

- The block is filled in completely: every `Result:` line has an answer, and the author, app and tester names are real. The tester is a member of your group and is not you.
- Your name appears as the tester in at least one teammate's Snack. Testing a teammate's app is part of completing the lab.
- For check 7, the tester tries something the other checks do not cover and writes one or two sentences about what happened. Pressing the same tab twice, opening the same item twice in a row, and saving every item are all fair game.
- An honest `not built` is fine. It tells the author exactly what still has to be done before 1:20.
- If any check failed, the fix is in the submitted code and the last line says what changed. A fail that was fixed counts as finished. A fail that was left alone does not.
