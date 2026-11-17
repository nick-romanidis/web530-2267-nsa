# Lab 7 - Product Shop with Expo Router

## A Note on AI

AI assistants are welcome in this course, but use them to get unstuck, not to do the work. Most of this lab follows a guide, so the risk is not that AI writes your code. The risk is pasting every block without reading it and ending up with an app you cannot explain. Read each block before you paste it, and when something breaks, paste **your** file and the error and ask why, rather than asking for a fixed file. Never keep code you could not explain line by line to a classmate. Next week you change this app, and you can only change code you understand.

## Overview

You build the **Product Shop** from the [Introduction to Expo Router](https://seneca-ictoer.github.io/WEB530/H-Expo/expo-router) page in the course notes: a product list, a details screen for each product, a Settings tab, and a Jotai atom that holds the products. Plan for about 60 to 75 minutes.

- Part 1 creates the project and runs it.
- Parts 2 to 4 follow the guide, section by section.
- Part 5 makes the shop yours.

**Keep this project.** Next week's lab starts from this exact project and moves the product list into a Firebase database. Do not delete the folder, and bring it to class next week on the computer you use in class.

Last week in class you built a similar shop from an empty project, with a `src/app/` folder. This lab is a **new project from the tabs template**, with the `app/` folder at the top of the project, because that is the project next week's Firebase guide expects. Do not continue last week's `shop` project.

## Setup

- You need Node.js (LTS), VS Code, and the Expo Go app on your phone, as last week. The [Setup Tools & Dev Environment](https://seneca-ictoer.github.io/WEB530/Z-Guides/tool-dev-setup) guide has the steps.
- Update Expo Go from the App Store or Google Play before you start. The project uses Expo SDK 58, and Expo Go only opens projects made for its own SDK version. SDK 58 needs iOS 16.4 or later (iPhone 8 and newer) or Android 7 or later.
- If your phone cannot run it, use the browser preview (press **w**). Every part of this lab works there.

## Rules

- Use the file and folder names from the guide exactly: `store/productsAtom.ts`, `app/(tabs)/products/index.tsx`, `app/(tabs)/products/[id].tsx`, and so on. Next week's guide tells you which file to open by its path.
- The product list lives in **one** place: `productsAtom`. No screen keeps its own copy of the products. Next week only the atom changes where the products come from, and every screen that reads it follows.
- Styles go in each file's `StyleSheet.create`, as in the guide. No inline `style={{ ... }}` objects.

## Submission

- Submit to the `Lab 7` assessment in Blackboard:
  1. A zip of your code. In File Explorer (or Finder), open your project folder, select the `app` folder, the `store` folder and `app.json`, and compress them into one zip. Do **not** zip `node_modules`: it is hundreds of megabytes, and `npm install` rebuilds it.
  2. Three screenshots, from your phone or the browser:
     - A product's details screen, showing the header `Product Details` with a back arrow **and** the tab bar at the bottom.
     - The Settings tab from Part 5.
     - VS Code's Explorer with the `app`, `app/(tabs)`, `app/(tabs)/products` and `store` folders expanded.
- The lab is due Friday at 11:59 PM.
- Only students who participate in class and hand in a lab slip can receive marks for this lab, even though it is due after class. If you miss the class, you still need to complete the app before next week's lab, because that lab builds on it.

## Part 1 - Create and Run the Project (1 point)

Follow **Setting Up Expo Router** and **Running Your New App** in the guide.

- Create the project with the tabs template, exactly as the guide shows: `npx create-expo-app@latest MyRouterApp --template tabs@sdk-58`. You may use another name than `MyRouterApp`.
- Open `app.json`, find the `"web"` section and change `"output": "server"` to `"output": "single"`. Without this, the browser preview can show a page that ignores every click.
- Start the server and open the template's app on your phone or in the browser before you change anything. You should see the template's starter app with two tabs.

Hint: On the college Wi-Fi your phone usually needs `npx expo start --tunnel`. If it asks to install `@expo/ngrok`, say yes. If the tunnel will not start after a couple of tries, do not lose the class to it: press **w** and work in the browser.

## Part 2 - Tabs and Settings (1 point)

Follow **Configure Tabs** and **Update the Settings Screen**.

- The tabs read **Products** (cart icon) and **Settings** (gear icon).
- The Settings tab shows the title `Settings`.

Hint: Renaming `two.tsx` to `settings.tsx` breaks the second tab until `_layout.tsx` uses the new name. That is expected, and the guide fixes it in the next steps.

## Part 3 - Products from a Jotai Atom (1 point)

Follow **Install Jotai and Configure a Product List**, **Update Tab One - Make it the Product Shop** and **Create the Dynamic Product Details Screen**.

- The Products tab lists the four products from `productsAtom`, with their prices.
- Tapping a product opens its details: name, price and description.
- An id that does not exist shows `Product Not Found` instead of crashing. Test it in the browser by changing the address to `/product/99`.

Stop the server before `npm install jotai` and start it again afterwards.

At the **Checkpoint**, notice the two problems the guide describes: the header says `product/[id]`, and the tab bar disappears on the details screen. You fix both in Part 4.

## Part 4 - Keep the Tab Bar with a Nested Stack (1 point)

Follow **Resolving the Issues: Building a Tab-Nested Stack**, Steps 1 to 5.

- The details screen's header reads `Product Details`, with a back arrow that returns to the list.
- The tab bar stays visible on the details screen.
- The app opens on the product list, not on an **Unmatched Route** screen.
- There is no third tab button for the redirect file.
- Your `app/` folder matches the finished tree at the end of Step 5.

Hint: Moving files while the server is running can leave a red squiggle under a correct `href`. Stop the server (**Ctrl+C**) and start it again. If the app itself shows an old screen, press **r** to reload.

## Part 5 - Make It Yours (1 point)

- Add **two products of your own** to `productsAtom`, with ids `'5'` and `'6'` and your own name, price and description. They appear in the list and their details screens work. Next week the product list moves out of this file and into a Firebase database, so pick products you would like to see there.
- Replace the template text on the Settings screen. Under the `Settings` title, show:
  - `Built by` and your name.
  - `Products in the shop:` and the number of products, read from `productsAtom` with `useAtomValue`. With your two products added, it reads `Products in the shop: 6`. Do not type the number: it must come from the atom.
- Remove the `EditScreenInfo` line and its import from `settings.tsx`, so the screen shows only your content.
- At the top of `store/productsAtom.ts`, add a comment explaining, in your own words, why the Products list, the details screen and the Settings count all update when you add a product here, even though none of them has its own copy of the list.

Hint: Settings is in a different tab from the product list, but it reads the same atom in the same way the list does. The count needs one hook call and `.length`.
