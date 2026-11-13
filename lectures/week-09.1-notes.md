# Week 9.1 - Notes and Examples

## Expo Project Setup

This week we leave Snack. Expo Router builds the navigation from a folder of screen files, and Snack only runs a single `App.js`, so these examples need a real Expo project on your computer. Your React Native code does not change: the same components, hooks and styles, in files you edit in VS Code.

- Before class: install Node.js (LTS), VS Code and the Expo Go app on your phone. The [Setup Tools & Dev Environment](https://seneca-ictoer.github.io/WEB530/Z-Guides/tool-dev-setup) guide has the steps.
- In VS Code, open an empty folder, open a terminal (**Terminal**, **New Terminal**) and create the project. This takes a minute or two.

```bash
npx create-expo-app@latest shop --template default@sdk-58
cd shop
npm run reset-project
```

- `default@sdk-58` picks the default template on Expo SDK 58, the version the Expo Go app in the App Store and Google Play runs. Expo Go only opens projects made for its own SDK version, so the two have to match.
- `reset-project` clears out the template's demo screens and leaves an empty `src/app/` folder with two files. It asks whether to move the demo files to `/example`: answer **n** to delete them. We will write every screen ourselves.
- Open `app.json`, find the `"web"` section and change `"output": "server"` to `"output": "single"`. With `server`, the browser preview draws each page on your computer first, and opening an address like `/products/2` directly shows a red **Hydration failed** warning. `single` is the plain browser app and is fine for everything this week. It changes nothing on phones.
- Start the development server and leave it running.

```bash
npx expo start
```

- Press **w** to open the app in the browser. Most of this week's demos happen there, because the browser has an address bar and Expo Router gives every screen an address.
- To run it on your phone, scan the QR code in the terminal: with the Expo Go app on Android, with the Camera app on iOS. On the college Wi-Fi, stop the server (**Ctrl+C**) and start it with `npx expo start --tunnel` instead. If it asks to install `@expo/ngrok`, say yes.
- Save a file and the app reloads, like Snack. Press **r** in the terminal to reload by hand.
- Update Expo Go from the App Store or Google Play before class. SDK 58 needs iOS 16.4 or later (iPhone 8 and newer) or Android 7 or later. If your phone cannot run it, use the browser (**w**): every example this week works there.

## Why Expo Router?

- In Week 4 we built navigation in code: a `NavigationContainer`, `createNativeStackNavigator()`, a `Stack.Screen` for every screen, and `navigation.navigate("Details", { id })` to move between them. Every new screen meant editing the navigator too.
- Expo Router replaces that setup with folders. **Every file in the `app` folder is a screen, and its path is its address**: `src/app/about.tsx` is the screen `/about`. Add a file and the screen exists.
- Expo Router looks for `app` either at the top of the project or inside `src`. This template uses `src/app/`, and keeps the rest of your code (data, state, components) elsewhere in `src/`. The Docusaurus guide uses the tabs template, which has `app/` at the top. Same rules, different folder.
- It is built on React Navigation. Stacks, tabs, headers, back buttons and `options` all work the way you learned in Week 4. Only the setup changes.
- Every screen having an address means links, deep links (opening the app straight to `/products/2` from a web page or notification) and the web version come for free.
- New Expo projects use Expo Router by default, and the files are `.tsx`, so everything from Week 8 applies.

## Example: Files Become Screens

- Two screens and a stack, built one file at a time. Do this in the project you just created.

### Starter Code

- `reset-project` left these two files in `src/app/`. Run the app (press **w**) and you see the one screen.
- `src/app/index.tsx` is the first screen. `index` means "the default screen of this folder", so it is the address `/`.

```tsx
import { Text, View, StyleSheet } from "react-native";

export default function Index() {
  return (
    <View style={styles.container}>
      <Text>Edit src/app/index.tsx to edit this screen.</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
});
```

- `src/app/_layout.tsx` is not a screen. It wraps every screen in its folder, and here it puts them in a stack, so you get a header and a back arrow.

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return <Stack />;
}
```

- There is no `NavigationContainer` and no list of screens. The stack finds its screens by looking at the files next to `_layout.tsx`.

### Step 1 - Add a Screen

- In VS Code, right-click the `app` folder inside `src`, choose **New File** and name it `about.tsx`. Paste this into it.

```tsx
import { StyleSheet, Text, View } from "react-native";

export default function AboutScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>About</Text>
      <Text>This screen is the file src/app/about.tsx.</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center", gap: 16 },
  title: { fontSize: 28, fontWeight: "700" },
});
```

- In the browser, change the address to `http://localhost:8081/about`. The screen is there. You did not register it anywhere: the file is the route.
- The header says `about`, the file name. We fix that in Step 3.
- A screen file must `export default` its component. That is how Expo Router finds it.

### Step 2 - Link to It

- Replace `src/app/index.tsx` with a Home screen that links to About.

```tsx
import { StyleSheet, Text, View } from "react-native";
import { Link } from "expo-router";

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home</Text>
      <Link href="/about" style={styles.link}>
        About (Link)
      </Link>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center", gap: 16 },
  title: { fontSize: 28, fontWeight: "700" },
  link: { fontSize: 18, color: "#007AFF" },
});
```

- Click the link. The address changes to `/about` and the header gets a back arrow, because the stack now has two screens in it.
- `Link` works like an `<a href>` on a web page. `href` is the path of the screen you want.

### Step 3 - Set the Titles

- Screen options go in the layout. Replace `src/app/_layout.tsx`:

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: "Home" }} />
      <Stack.Screen name="about" options={{ title: "About Us" }} />
    </Stack>
  );
}
```

- `name` is the file name without `.tsx`. `options` are the same options you used on `Stack.Screen` in Week 4.
- You only need a `Stack.Screen` for screens whose options you want to change. A file without one is still a screen.

### Step 4 - Navigate from Code

- `Link` is for something the user taps. To navigate from inside a function, such as an `onPress`, use `router`.
- In `src/app/index.tsx`, add `Button` to the `react-native` import and `router` to the `expo-router` import, then add a button under the link.

```tsx
import { Button, StyleSheet, Text, View } from "react-native";
import { Link, router } from "expo-router";
```

```tsx
<Button title="About (router.push)" onPress={() => router.push("/about")} />
```

- In `src/app/about.tsx`, add a Go Back button the same way.

```tsx
import { Button, StyleSheet, Text, View } from "react-native";
import { router } from "expo-router";
```

```tsx
<Button title="Go Back" onPress={() => router.back()} />
```

- `router.push("/about")` is Week 4's `navigation.push("About")` with a path instead of a screen name. `router.back()` is `navigation.goBack()`.
- `router.replace("/about")` swaps the current screen instead of adding one, so Back skips it. You will see it after a login screen: once you are in, Back should not return to the login.

### What to Show

1. Click the button and the link. Both reach `/about`, and Back returns to Home.
2. Now open `http://localhost:8081/about` directly (type it in the address bar and press Enter). There is no back arrow, because the stack only has this one screen. Press **Go Back**: nothing happens, and the browser console says `The action 'GO_BACK' was not handled by any navigator`. On a phone this happens when a link from outside opens the app straight to a screen.
3. Fix it by checking first. `router.canGoBack()` is true when there is a screen to go back to.

```tsx
function goBack() {
  if (router.canGoBack()) {
    router.back();
  } else {
    router.replace("/");
  }
}
```

4. Put the function inside `AboutScreen`, above the `return`, and change the button to `onPress={goBack}`. Open `/about` directly again: Go Back now takes you Home.
5. Change the address to `/nope`. Expo Router shows **Unmatched Route** for a path with no file. Click **Sitemap** on that page: it lists every route in the app, generated from your files.

### Finished Code

- Compare with these if something does not work.
- `src/app/_layout.tsx`

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: "Home" }} />
      <Stack.Screen name="about" options={{ title: "About Us" }} />
    </Stack>
  );
}
```

- `src/app/index.tsx`

```tsx
import { Button, StyleSheet, Text, View } from "react-native";
import { Link, router } from "expo-router";

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Home</Text>
      <Link href="/about" style={styles.link}>
        About (Link)
      </Link>
      <Button title="About (router.push)" onPress={() => router.push("/about")} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center", gap: 16 },
  title: { fontSize: 28, fontWeight: "700" },
  link: { fontSize: 18, color: "#007AFF" },
});
```

- `src/app/about.tsx`

```tsx
import { Button, StyleSheet, Text, View } from "react-native";
import { router } from "expo-router";

export default function AboutScreen() {
  function goBack() {
    if (router.canGoBack()) {
      router.back();
    } else {
      router.replace("/");
    }
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>About</Text>
      <Text>This screen is the file src/app/about.tsx.</Text>
      <Button title="Go Back" onPress={goBack} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center", gap: 16 },
  title: { fontSize: 28, fontWeight: "700" },
});
```

## Example: Product Shop

- A small shop: a product list, a details screen for each product, a tab bar, and a cart. Each step adds one Expo Router feature. It is the same app as the [Introduction to Expo Router](https://seneca-ictoer.github.io/WEB530/H-Expo/expo-router) page, built from an empty project instead of the tabs template.

### Starter Code

- Keep working in the same project. Delete `src/app/about.tsx`.
- The product data is not a screen, so it goes in `src/`, **next to** `app/`, not inside it. Anything in `src/app/` becomes a route. In `src`, create a folder `data` and a file `src/data/products.ts`.

```ts
export interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
}

export const PRODUCTS: Product[] = [
  {
    id: "1",
    name: "Wireless Headphones",
    price: 299.99,
    description: "Noise cancelling, with a 30-hour battery.",
  },
  {
    id: "2",
    name: "Mechanical Keyboard",
    price: 129.99,
    description: "Tactile switches and a detachable USB-C cable.",
  },
  {
    id: "3",
    name: "Ergonomic Mouse",
    price: 79.99,
    description: "Shaped to keep your wrist straight on long days.",
  },
  {
    id: "4",
    name: "4K Monitor",
    price: 449.99,
    description: "27 inches, with a stand that tilts and swivels.",
  },
];
```

- Replace `src/app/_layout.tsx` with a plain stack again.

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return <Stack />;
}
```

- Replace `src/app/index.tsx` with the product list. The rows do nothing yet.

```tsx
import { FlatList, Pressable, StyleSheet, Text, View } from "react-native";
import { PRODUCTS } from "@/data/products";

export default function ShopScreen() {
  return (
    <FlatList
      contentContainerStyle={styles.list}
      data={PRODUCTS}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <Pressable style={styles.card}>
          <View style={styles.info}>
            <Text style={styles.name}>{item.name}</Text>
            <Text style={styles.price}>${item.price.toFixed(2)}</Text>
          </View>
          <Text style={styles.arrow}>View</Text>
        </Pressable>
      )}
    />
  );
}

const styles = StyleSheet.create({
  list: { padding: 16 },
  card: {
    flexDirection: "row",
    alignItems: "center",
    backgroundColor: "#fff",
    padding: 16,
    borderRadius: 12,
    marginBottom: 12,
  },
  info: { flex: 1 },
  name: { fontSize: 16, fontWeight: "600" },
  price: { fontSize: 14, color: "#007AFF", marginTop: 4 },
  arrow: { color: "#8e8e93", fontWeight: "600" },
});
```

- `@/data/products` means "`data/products` inside `src`". The template sets up `@/` so an import still works after you move a file into a deeper folder, which we are about to do twice.
- The list and its styles are Week 5 material. Today we only add navigation.

### Step 1 - A Dynamic Route

- One details screen has to show any product. A file name in square brackets is a placeholder: `[id].tsx` matches `/products/1`, `/products/2` and anything else in that spot.
- In `src/app`, create a folder `products`, and in it a file `[id].tsx` (with the brackets).

```tsx
import { StyleSheet, Text, View } from "react-native";
import { useLocalSearchParams } from "expo-router";
import { PRODUCTS } from "@/data/products";

export default function ProductScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const product = PRODUCTS.find((p) => p.id === id);

  if (!product) {
    return (
      <View style={styles.container}>
        <Text style={styles.error}>No product with id {id}</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.sku}>Product {product.id}</Text>
      <Text style={styles.name}>{product.name}</Text>
      <Text style={styles.price}>${product.price.toFixed(2)}</Text>
      <Text style={styles.description}>{product.description}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24 },
  sku: { fontSize: 12, color: "#8e8e93", fontWeight: "700", marginBottom: 8 },
  name: { fontSize: 24, fontWeight: "700", marginBottom: 6 },
  price: { fontSize: 20, color: "#007AFF", marginBottom: 16 },
  description: { fontSize: 16, lineHeight: 22 },
  error: { fontSize: 18, color: "#ff3b30" },
});
```

- `useLocalSearchParams()` returns the values from the address. The file is `[id].tsx`, so the value is called `id`. It is always a string, which is why the products use string ids.
- The `<{ id: string }>` is the Week 8 type annotation, telling TypeScript what is in the params.
- Try it before writing any link: change the address to `/products/2`, then `/products/99`. One file, every product, and a message for an id that does not exist.
- Now make the rows link there. In `src/app/index.tsx`, import `Link` and wrap the `Pressable` in it.

```tsx
import { Link } from "expo-router";
```

```tsx
renderItem={({ item }) => (
  <Link href={`/products/${item.id}`} asChild>
    <Pressable style={styles.card}>
      <View style={styles.info}>
        <Text style={styles.name}>{item.name}</Text>
        <Text style={styles.price}>${item.price.toFixed(2)}</Text>
      </View>
      <Text style={styles.arrow}>View</Text>
    </Pressable>
  </Link>
)}
```

- `asChild` tells `Link` not to draw its own text link, but to hand the press to its child. The whole card becomes the link and keeps its styles.
- In Week 4 the id travelled as a param: `navigation.navigate("Details", { id })`. Here it is part of the address.
- If VS Code puts a red squiggle under the `href` even though the code is right, the app is fine and the editor is out of date. Expo writes the list of routes into a type file so TypeScript can check every link, and the list falls behind when you add, move or rename route files while `npx expo start` is running. Stop the server (**Ctrl+C**) and start it again, and the squiggle goes away. Expect this any time you add, move or rename a file in `src/app/`.
- Tap a product. The details work, but the header says `products/[id]`.

### Step 2 - Titles from Data

- Give the list a title in `src/app/_layout.tsx`.

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: "Shop" }} />
    </Stack>
  );
}
```

- The details title depends on which product is showing, so the layout cannot know it. A screen can set its own options by rendering a `Stack.Screen` inside itself. In `src/app/products/[id].tsx`, import `Stack` and add the line at the top of the details view.

```tsx
import { Stack, useLocalSearchParams } from "expo-router";
```

```tsx
<View style={styles.container}>
  <Stack.Screen options={{ title: product.name }} />
  <Text style={styles.sku}>Product {product.id}</Text>
```

- `Stack.Screen` without a `name` means "this screen". It draws nothing. It only sets the options. In Week 4 an `options` function read `route.params` to build a title. Here the screen already has the product, so it sets the title itself.

### Step 3 - Tabs in a Route Group

- Add a Cart tab next to the shop. Tabs need a layout of their own, so they need a folder.
- In `src/app`, create a folder named `(tabs)`, with the parentheses. Drag `src/app/index.tsx` into it.
- Parentheses make a **route group**: the folder organizes files and gets its own layout, but its name is left out of the address. `src/app/(tabs)/index.tsx` is still `/`.
- In `src/app/(tabs)`, create `cart.tsx`.

```tsx
import { StyleSheet, Text, View } from "react-native";

export default function CartScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.empty}>Your cart is empty.</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center" },
  empty: { fontSize: 16, color: "#8e8e93" },
});
```

- In `src/app/(tabs)`, create `_layout.tsx`. This layout returns `Tabs` instead of `Stack`, and every file next to it becomes a tab.

```tsx
import { Tabs } from "expo-router";
import { SymbolView } from "expo-symbols";

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen
        name="index"
        options={{
          title: "Shop",
          tabBarIcon: ({ color }) => (
            <SymbolView
              name={{ ios: "bag.fill", android: "shopping_bag", web: "shopping_bag" }}
              tintColor={color}
              size={24}
            />
          ),
        }}
      />
      <Tabs.Screen
        name="cart"
        options={{
          title: "Cart",
          tabBarIcon: ({ color }) => (
            <SymbolView
              name={{ ios: "cart.fill", android: "shopping_cart", web: "shopping_cart" }}
              tintColor={color}
              size={24}
            />
          ),
        }}
      />
    </Tabs>
  );
}
```

- `tabBarIcon` draws each tab's icon. It receives the tab's current `color`, which is highlighted when the tab is selected.
- `SymbolView` (from `expo-symbols`, already in the project) shows each platform's own icons, so it takes one name per platform: an SF Symbol for iOS, and a Material Symbols name for Android and the web. Material names use underscores: `shopping_bag`, `shopping_cart`.
- The root layout now has one child, the `(tabs)` group. Replace `src/app/_layout.tsx`. The tabs draw their own header, so hide the stack's to avoid two.

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
    </Stack>
  );
}
```

- Switch tabs and watch the address: `/` and `/cart`. No `(tabs)` in either.
- Tap a product. The details screen still works, but it covers the tab bar: `src/app/products/[id].tsx` is outside `(tabs)`, so it opens in the root stack, on top of the tabs. Sometimes that is what you want (a checkout screen, say). For browsing, the tab bar should stay.

### Step 4 - Keep the Tab Bar with a Nested Stack

- To push a screen **inside** a tab, the tab needs its own stack, so the shop tab becomes a folder with its own `_layout.tsx`. This is the Week 4 pattern of a stack nested in a tab, written as folders.
- In `src/app/(tabs)`, create a folder `products`. Drag `src/app/(tabs)/index.tsx` and `src/app/products/[id].tsx` into it, then delete the empty `src/app/products` folder. You should now have:

```
src/app/
├── _layout.tsx
└── (tabs)/
    ├── _layout.tsx
    ├── cart.tsx
    └── products/
        ├── index.tsx
        └── [id].tsx
```

- The list is now `src/app/(tabs)/products/index.tsx`, the address `/products`. The details are still `/products/2`, so the `Link` in the list does not change.
- In `src/app/(tabs)/products`, create `_layout.tsx` with the shop tab's stack.

```tsx
import { Stack } from "expo-router";

// Opening /products/2 directly still puts the list underneath, so Back works.
export const unstable_settings = {
  anchor: "index",
};

export default function ProductsLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: "Shop" }} />
    </Stack>
  );
}
```

- `anchor` names the screen that should always sit at the bottom of this stack. Without it, opening `/products/2` directly shows the details with no list underneath and no back arrow.
- In `src/app/(tabs)/_layout.tsx`, the first tab is now the `products` folder, not the `index` file. Change its `name` and hide the tab's header, because the stack inside draws one.

```tsx
<Tabs.Screen
  name="products"
  options={{
    title: "Shop",
    headerShown: false,
    tabBarIcon: ({ color }) => (
      <SymbolView
        name={{ ios: "bag.fill", android: "shopping_bag", web: "shopping_bag" }}
        tintColor={color}
        size={24}
      />
    ),
  }}
/>
```

- Nothing lives at `/` any more, so opening the app would show Unmatched Route. Create a **new file** named `index.tsx` directly in `src/app`, next to `_layout.tsx`, that sends the user to the shop. Do not paste this into `_layout.tsx`: that file keeps the root `Stack` from Step 3.

```tsx
import { Redirect } from "expo-router";

export default function Index() {
  return <Redirect href="/products" />;
}
```

- `Redirect` replaces the current screen with another, like `router.replace`, as soon as it renders.
- Tap a product: the details slide in and the tab bar stays. Switch to Cart and back: the shop tab remembers where you were, because each tab keeps its own stack.
- If a red squiggle appears under a correct `href` after these moves, restart the server (see the tip in Step 1).

### Step 5 - A Cart Shared Across Tabs

- If there is time: put products in the cart from the details screen and show them in the Cart tab. Two screens in different tabs need the same data, which is the Week 2 job for Jotai. Atoms work in an Expo project exactly as they did in Snack.
- Stop the server, install Jotai, and start it again.

```bash
npm install jotai
npx expo start
```

- State is not a screen either, so it goes in `src/`, next to `app/`. Create a folder `store` in `src` and a file `src/store/cart.ts`.

```ts
import { atom } from "jotai";

// The ids of the products in the cart. Adding the same product twice adds it twice.
export const cartAtom = atom<string[]>([]);
```

- In `src/app/(tabs)/products/[id].tsx`, add an **Add to Cart** button. First, change the `react-native` import on the first line so it includes `Button`:

```tsx
import { Button, StyleSheet, Text, View } from "react-native";
```

- Then add these three imports under the existing ones:

```tsx
import { useSetAtom } from "jotai";
import { PRODUCTS } from "@/data/products";
import { cartAtom } from "@/store/cart";
```

- Inside `ProductScreen`, add the atom setter under the line that finds the product:

```tsx
const product = PRODUCTS.find((p) => p.id === id);
const setCart = useSetAtom(cartAtom);
```

- And add the button under the description:

```tsx
<Text style={styles.description}>{product.description}</Text>
<Button title="Add to Cart" onPress={() => setCart((prev) => [...prev, product.id])} />
```

- The new cart depends on the old one, so it uses the updater form, the same rule as `useState`.
- Replace `src/app/(tabs)/cart.tsx` with a screen that reads the atom.

```tsx
import { Button, FlatList, StyleSheet, Text, View } from "react-native";
import { useAtom } from "jotai";
import { PRODUCTS } from "@/data/products";
import { cartAtom } from "@/store/cart";

export default function CartScreen() {
  const [cart, setCart] = useAtom(cartAtom);

  const items = cart.map((id) => PRODUCTS.find((p) => p.id === id)!);
  const total = items.reduce((sum, item) => sum + item.price, 0);

  if (items.length === 0) {
    return (
      <View style={styles.container}>
        <Text style={styles.empty}>Your cart is empty.</Text>
      </View>
    );
  }

  return (
    <FlatList
      contentContainerStyle={styles.list}
      data={items}
      keyExtractor={(item, index) => `${item.id}-${index}`}
      renderItem={({ item }) => (
        <View style={styles.row}>
          <Text style={styles.name}>{item.name}</Text>
          <Text>${item.price.toFixed(2)}</Text>
        </View>
      )}
      ListFooterComponent={
        <View style={styles.footer}>
          <Text style={styles.total}>Total: ${total.toFixed(2)}</Text>
          <Button title="Empty Cart" onPress={() => setCart([])} />
        </View>
      }
    />
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center" },
  empty: { fontSize: 16, color: "#8e8e93" },
  list: { padding: 16 },
  row: {
    flexDirection: "row",
    justifyContent: "space-between",
    backgroundColor: "#fff",
    padding: 16,
    borderRadius: 12,
    marginBottom: 8,
  },
  name: { fontWeight: "600" },
  footer: { marginTop: 16, gap: 12 },
  total: { fontSize: 18, fontWeight: "700" },
});
```

- The `!` after `find(...)` tells TypeScript the product will be found. The cart only ever holds ids that came from `PRODUCTS`.
- Layouts are components too, so they can read atoms. Show the cart count on the Cart tab. In `src/app/(tabs)/_layout.tsx`:

```tsx
import { useAtomValue } from "jotai";
import { cartAtom } from "@/store/cart";
```

```tsx
export default function TabsLayout() {
  const cart = useAtomValue(cartAtom);
```

```tsx
title: "Cart",
tabBarBadge: cart.length > 0 ? cart.length : undefined,
```

- Add the same product twice and a different one once. The badge says 3, the Cart tab lists all three with the total, and **Empty Cart** clears the list and the badge.

### Finished Code

- Compare with these if something does not work.
- `src/data/products.ts`

```ts
export interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
}

export const PRODUCTS: Product[] = [
  {
    id: "1",
    name: "Wireless Headphones",
    price: 299.99,
    description: "Noise cancelling, with a 30-hour battery.",
  },
  {
    id: "2",
    name: "Mechanical Keyboard",
    price: 129.99,
    description: "Tactile switches and a detachable USB-C cable.",
  },
  {
    id: "3",
    name: "Ergonomic Mouse",
    price: 79.99,
    description: "Shaped to keep your wrist straight on long days.",
  },
  {
    id: "4",
    name: "4K Monitor",
    price: 449.99,
    description: "27 inches, with a stand that tilts and swivels.",
  },
];
```

- `src/store/cart.ts`

```ts
import { atom } from "jotai";

// The ids of the products in the cart. Adding the same product twice adds it twice.
export const cartAtom = atom<string[]>([]);
```

- `src/app/_layout.tsx`

```tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
    </Stack>
  );
}
```

- `src/app/index.tsx`

```tsx
import { Redirect } from "expo-router";

export default function Index() {
  return <Redirect href="/products" />;
}
```

- `src/app/(tabs)/_layout.tsx`

```tsx
import { Tabs } from "expo-router";
import { SymbolView } from "expo-symbols";
import { useAtomValue } from "jotai";
import { cartAtom } from "@/store/cart";

export default function TabsLayout() {
  const cart = useAtomValue(cartAtom);

  return (
    <Tabs>
      <Tabs.Screen
        name="products"
        options={{
          title: "Shop",
          headerShown: false,
          tabBarIcon: ({ color }) => (
            <SymbolView
              name={{ ios: "bag.fill", android: "shopping_bag", web: "shopping_bag" }}
              tintColor={color}
              size={24}
            />
          ),
        }}
      />
      <Tabs.Screen
        name="cart"
        options={{
          title: "Cart",
          tabBarBadge: cart.length > 0 ? cart.length : undefined,
          tabBarIcon: ({ color }) => (
            <SymbolView
              name={{ ios: "cart.fill", android: "shopping_cart", web: "shopping_cart" }}
              tintColor={color}
              size={24}
            />
          ),
        }}
      />
    </Tabs>
  );
}
```

- `src/app/(tabs)/cart.tsx`

```tsx
import { Button, FlatList, StyleSheet, Text, View } from "react-native";
import { useAtom } from "jotai";
import { PRODUCTS } from "@/data/products";
import { cartAtom } from "@/store/cart";

export default function CartScreen() {
  const [cart, setCart] = useAtom(cartAtom);

  const items = cart.map((id) => PRODUCTS.find((p) => p.id === id)!);
  const total = items.reduce((sum, item) => sum + item.price, 0);

  if (items.length === 0) {
    return (
      <View style={styles.container}>
        <Text style={styles.empty}>Your cart is empty.</Text>
      </View>
    );
  }

  return (
    <FlatList
      contentContainerStyle={styles.list}
      data={items}
      keyExtractor={(item, index) => `${item.id}-${index}`}
      renderItem={({ item }) => (
        <View style={styles.row}>
          <Text style={styles.name}>{item.name}</Text>
          <Text>${item.price.toFixed(2)}</Text>
        </View>
      )}
      ListFooterComponent={
        <View style={styles.footer}>
          <Text style={styles.total}>Total: ${total.toFixed(2)}</Text>
          <Button title="Empty Cart" onPress={() => setCart([])} />
        </View>
      }
    />
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center" },
  empty: { fontSize: 16, color: "#8e8e93" },
  list: { padding: 16 },
  row: {
    flexDirection: "row",
    justifyContent: "space-between",
    backgroundColor: "#fff",
    padding: 16,
    borderRadius: 12,
    marginBottom: 8,
  },
  name: { fontWeight: "600" },
  footer: { marginTop: 16, gap: 12 },
  total: { fontSize: 18, fontWeight: "700" },
});
```

- `src/app/(tabs)/products/_layout.tsx`

```tsx
import { Stack } from "expo-router";

// Opening /products/2 directly still puts the list underneath, so Back works.
export const unstable_settings = {
  anchor: "index",
};

export default function ProductsLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: "Shop" }} />
    </Stack>
  );
}
```

- `src/app/(tabs)/products/index.tsx`

```tsx
import { FlatList, Pressable, StyleSheet, Text, View } from "react-native";
import { Link } from "expo-router";
import { PRODUCTS } from "@/data/products";

export default function ShopScreen() {
  return (
    <FlatList
      contentContainerStyle={styles.list}
      data={PRODUCTS}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <Link href={`/products/${item.id}`} asChild>
          <Pressable style={styles.card}>
            <View style={styles.info}>
              <Text style={styles.name}>{item.name}</Text>
              <Text style={styles.price}>${item.price.toFixed(2)}</Text>
            </View>
            <Text style={styles.arrow}>View</Text>
          </Pressable>
        </Link>
      )}
    />
  );
}

const styles = StyleSheet.create({
  list: { padding: 16 },
  card: {
    flexDirection: "row",
    alignItems: "center",
    backgroundColor: "#fff",
    padding: 16,
    borderRadius: 12,
    marginBottom: 12,
  },
  info: { flex: 1 },
  name: { fontSize: 16, fontWeight: "600" },
  price: { fontSize: 14, color: "#007AFF", marginTop: 4 },
  arrow: { color: "#8e8e93", fontWeight: "600" },
});
```

- `src/app/(tabs)/products/[id].tsx`

```tsx
import { Button, StyleSheet, Text, View } from "react-native";
import { Stack, useLocalSearchParams } from "expo-router";
import { useSetAtom } from "jotai";
import { PRODUCTS } from "@/data/products";
import { cartAtom } from "@/store/cart";

export default function ProductScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const product = PRODUCTS.find((p) => p.id === id);
  const setCart = useSetAtom(cartAtom);

  if (!product) {
    return (
      <View style={styles.container}>
        <Text style={styles.error}>No product with id {id}</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Stack.Screen options={{ title: product.name }} />
      <Text style={styles.sku}>Product {product.id}</Text>
      <Text style={styles.name}>{product.name}</Text>
      <Text style={styles.price}>${product.price.toFixed(2)}</Text>
      <Text style={styles.description}>{product.description}</Text>
      <Button title="Add to Cart" onPress={() => setCart((prev) => [...prev, product.id])} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24 },
  sku: { fontSize: 12, color: "#8e8e93", fontWeight: "700", marginBottom: 8 },
  name: { fontSize: 24, fontWeight: "700", marginBottom: 6 },
  price: { fontSize: 20, color: "#007AFF", marginBottom: 16 },
  description: { fontSize: 16, lineHeight: 22, marginBottom: 24 },
  error: { fontSize: 18, color: "#ff3b30" },
});
```

## Expo Router and React Navigation

| Week 4 (React Navigation)                        | Expo Router                                          |
| :----------------------------------------------- | :--------------------------------------------------- |
| `NavigationContainer` around the app             | Not needed. `src/app/_layout.tsx` is the root            |
| `createNativeStackNavigator()` and `Stack.Navigator` | A `_layout.tsx` that returns `<Stack />`         |
| `<Stack.Screen name="Details" component={...} />` | A file: `src/app/details.tsx`                           |
| `navigation.navigate("Details", { id })`          | `` router.push(`/products/${id}`) `` or `<Link href=...>` |
| `route.params.id`                                 | `useLocalSearchParams()`                            |
| `navigation.goBack()`                             | `router.back()`                                     |
| `options={({ route }) => ({ title: ... })}`      | `<Stack.Screen options={{ title }} />` inside the screen |
| A stack nested in a tab                           | A folder with its own `_layout.tsx` inside the tabs folder |

## Expo Router Checklist

- Screens go in `src/app/`. Everything else (data, atoms, shared components) goes elsewhere in `src/`, or it becomes a route.
- Every screen file needs `export default`.
- `index.tsx` is the default screen of its folder. `_layout.tsx` wraps the screens in its folder and returns `Stack`, `Tabs` or `Slot`.
- `[id].tsx` is a dynamic route. Read the value with `useLocalSearchParams()`, and remember it is a string.
- `(name)` folders group screens under one layout without changing the address.
- Use `Link` for things the user taps (`asChild` to make a `Pressable` the link) and `router.push`, `router.replace` and `router.back` from code.
- Set static options in the layout's `Stack.Screen name="..."`, and data-driven ones with a `Stack.Screen` inside the screen.
- Check a route in the browser by typing its address. If an address shows Unmatched Route, check the file name and folder.
- Keep the Week 2 updater rule in atoms too: `setCart((prev) => [...prev, id])`.
