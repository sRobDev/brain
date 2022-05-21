TailwindCSS is an incredibly popular CSS framework, so it's natural that a lot of devs looking into Remix will want to use Tailwind. This guide will get you up and running with Tailwind inside Remix in just a few minutes. If you're new to Tailwind or aren't comfortable with it, check out our [Beginner Tailwind course](https://www.better.dev/courses/beginner-tailwind)! 

# Step 1: Create a Remix Project
For the sake of completeness, we're going from Remix initialization through Tailwind integration. Pop open your terminal and run the following command:

`npx create-remix@latest`

What this does is creates a new Remix project using the latest version of Remix. You will then go through a few prompts in the terminal - so name your project whatever you want, pick a deployment target (for this guide, I selected `Remix App Server`, but it won't really make a difference), and then select whether to use TypeScript or Javascript (I selected TypeScript, but again, this won't make a difference for Tailwind). 

You should now be able to run `cd <your-remix-project>`, where you'll see the code for the example application.

# Step 2: Install Tailwind 
We're already almost there - in your project folder, run `npm install --save-dev concurrently tailwindcss`

> Note: [concurrently](https://www.npmjs.com/package/concurrently) is an npm package for running multiple commands in parallel - this way you can have Tailwind running and watching for changes in the same terminal window that you have Remix running

Then, run `npx tailwindcss init` to generate your `tailwind.config.css` file.

# Step 3: Configuration
Open up your generated `tailwind.config.css` - this is the file that controls all your global configuration for Tailwind in your project. It will initially look something like this:

```javascript
module.exports = {
  purge: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

We need to add `mode` and update the `purge` properties, like the following:

```javascript
module.exports = {
  mode: "jit"
  purge: ["./app/**/*.{ts,tsx}"],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}

```

> Note: If you selected JavaScript when initializing your Remix project, you should set the purge line to be `["./app/**/*.{js,jsx}"]`

Now, we also need to update our `package.json` scripts to be the following:

```
"scripts": {
	"build": "npm run build:css && remix build",
	"build:css": "tailwindcss -o ./app/tailwind.css",
	"dev": "concurrently \"npm run dev:css\" \"remix dev\"",
	"dev:css": "tailwindcss -o ./app/tailwind.css --watch",
	"postinstall": "remix setup node",
	"start": "remix-serve build"
},

```

This is where the other package we installed - `concurrently` - comes into play to make our lives easier. When we run `npm run dev` now, concurrently will fire both our `remix dev` script as well as our `dev:css` script, so that we don't have to have multiple terminal windows open for Tailwind, Remix, and so on. Once you run `npm run dev`, you should see output in your terminal similar to the following:

```
➜  tailwind-test npm run dev

> remix-app-template@ dev /Users/srob/Documents/code/tailwind-test
> concurrently "npm run dev:css" "remix dev"

[0]
[0] > remix-app-template@ dev:css /Users/srob/Documents/code/tailwind-test
[0] > tailwindcss -o ./app/tailwind.css --watch
[0]
[1] Watching Remix app in development mode...
[1] 💿 Built in 515ms
[1] Remix App Server started at http://localhost:3000
[0] Done in 5088ms.
[1] 💿 File created: app/tailwind.css
[1] 💿 Rebuilding...
[1] 💿 Rebuilt in 106ms
```

Great! One more step and we're good to go.

# Step 4: Importing/Linking
In order for our app to actually be able to use Tailwind, we have to import the generated stylesheet (which is `./app/tailwind.css` from the terminal output above). 

Open `app/root.tsx`, and find the `links` function and CSS imports (which should be somewhere near the top of the file) which should look something like this initially: 

```javascript
import globalStylesUrl from "~/styles/global.css";
import darkStylesUrl from "~/styles/dark.css";

export let links: LinksFunction = () => {
  return [
    { rel: "stylesheet", href: globalStylesUrl },
    {
      rel: "stylesheet",
      href: darkStylesUrl,
      media: "(prefers-color-scheme: dark)"
    }
  ];
};
```

Import your `tailwind.css` file and add it to the `links` function, like so:

```javascript
...
import tailwindStyles from "./tailwind.css"

export let links: LinksFunction = () => {
  return [
    { rel: "stylesheet", href: globalStylesUrl },
    {
      rel: "stylesheet",
      href: darkStylesUrl,
      media: "(prefers-color-scheme: dark)"
    },
	{ rel: "stylesheet", href: tailwindStyles}
  ];
};
```

> Note: You'll likely want to remove the other linked stylesheets in your `links` function, as well as their respective imports. I kept them in these examples simply for consistency. 

After you save, you're done! You can now use all that Tailwind has to offer in any component you write in Remix - there's no need to import it in any other components since all CSS imports in `root.tsx` are considered global.

Thanks for reading, and enjoy using Remix with Tailwind 🎉