[SASS](https://sass-lang.com/) is an incredibly popular CSS preprocessor that adds quality of life improvements such as nested rules, mixins, and so on. Personally, it has always been my go-to in my projects - so learning how to add SASS to [Remix](https://remix.run/) was super important for me!

## Install SASS
For this guide, I'm assuming you already have a Remix project set up. If you don't, check out Remix's excellent [Getting Started](https://remix.run/docs/en/v1#getting-started) guide, then come back here as soon as you have it running!

In your Remix project, pop open a terminal and run this command: 

```
npm install sass
```

This will install SASS into your project and therefore allow you to use the [SASS CLI](https://sass-lang.com/documentation/cli/dart-sass), which we'll get to shortly!

## Create styles/ Folder
Because we'll be creating vanilla CSS files through SASS, we want to store our `.scss` files separately from the rest of our app. This allows us to use SASS without changing the CSS imports in our project at all (which is convenient when we've just started a project, and massively time-saving if we're converting an existing Remix project to SCSS).

Create a `styles/` folder at the _root_ level of your project (outside of `app/`). You can do this in whatever way you like, but here's a quick terminal command for it: 

```
mkdir styles/
```

Your project directory should now look like this:

![Remix Directory Structure](v1638662582/scss-remix-directory-structure.png)

## Move and Rename CSS Files
Almost there! All you have to do for this step is:
- Drag all your CSS files (that are likely living in `app/styles` and its sub-folders) into the newly-created `styles/` folder
- Rename them to be `.scss` instead of `.css`. 


## Update package.json
Remix does not include built-in support for compiling or interpreting SASS, and doesn't [seem to plan to](https://github.com/remix-run/remix/issues/707#issuecomment-980678098), so it's up to us to run the SASS compiler ourselves alongside our dev process (or in parallel, which we'll get to later!).

In `package.json`, add this line to your scripts:

```json
"scripts": {
	...
	"sass": "sass --watch --no-source-map styles:app/styles",
	...
},
```

This script may seem a little intimidating at first, but I promise it's not. Let's break it down:

- `--watch`: This is simply telling the SASS compiler to continue watching the files/folder we give it and to re-run the compiler when it notices changes (essentially, this is just hot reload for SASS)
- `--no-source-map`: This is personal preference, so feel free to leave this out - this tells the SASS compiler to _not_ generate [source maps](https://css-tricks.com/should-i-use-source-maps-in-production/) for the CSS (i.e. `global.map.css`)
- `styles:app/styles`: Here, we're telling the compiler to take SCSS files it finds in `styles/` and send them to `app/styles`

> Note: The SASS compiler retains folder structure. So if you have `.scss` files in sub-folders, i.e. `styles/header/myHeader.scss`, this will output `app/styles/header/myHeader.css`

Now pop open your terminal and run this command:

```
npm run sass
```

You'll see some output like this:

```
...
> sass --watch --no-source-map styles:app/styles

Compiled styles/dark.scss to app/styles/dark.css.
Compiled styles/demos/about.scss to app/styles/demos/about.css.
Compiled styles/global.scss to app/styles/global.css.
Sass is watching for changes. Press Ctrl-C to stop.
```

You've done it! SASS has compiled your `.scss` files into `.css` files that all the imports in Remix will work perfectly with. You can see this for yourself if you open the `app/styles/` folder. 

When you want to edit your CSS, just make sure to do it in the `.scss` files. On save, the generated CSS files will update thanks to `--watch`!

Keep in mind that you'll have to have `npm run sass` running in a separate terminal tab (or window) from your `npm run dev` command for Remix. Since they are two separate processes, they have to be running at the same time - otherwise your CSS (or app) won't update.  This setup will work perfectly fine, but there is a small change we can make to make our lives easier!

## Running SASS in Parallel with Remix
Once again, pop open your terminal and run this command:

```
npm install concurrently
```

[concurrently](https://www.npmjs.com/package/concurrently) is an NPM package that allows you to run two commands in parallel. I'm sure you see where this is going now, so let's go into our `package.json` one more time and update the `dev` script:

```json
"scripts": {
	...
    "dev": "concurrently \"npm run sass\" \"remix dev\"",
	...
},
```

Now you can simply do `npm run dev` and you'll have the SASS compiler and Remix itself running in the same terminal! This is shown in the output for this command:

```
> concurrently "npm run sass" "remix dev"

[0] 
[0] > remix-app-template@ sass /Users/srob/Documents/code/sass-remix
[0] > sass --watch --no-source-map styles:app/styles
[0] 
[1] Watching Remix app in development mode...
[0] Compiled styles/dark.scss to app/styles/dark.css.
[0] Compiled styles/demos/about.scss to app/styles/demos/about.css.
[0] Compiled styles/global.scss to app/styles/global.css.
[0] Sass is watching for changes. Press Ctrl-C to stop.
[0] 
[1] 💿 Built in 393ms
[1] Remix App Server started at http://localhost:3000
```

If you make any updates to your `.scss` files, you'll see the output here - just like if you update any of your Remix app's files. 

Now, go and enjoy your seamless developer experience with SASS in Remix!