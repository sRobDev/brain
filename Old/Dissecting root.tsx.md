# Grokking Remix's root.tsx

## links function

The first stop on our road to understanding the foundation of Remix is `app/root.tsx` and the simple, happy `links` function. This is a function that any Remix dev will get intimately familiar with throughout their application. What it does is pretty straight forward - it returns an array of objects that will be transformed into HTML `<link>` elements. These elements are scoped to the component you've included them in - which means that the ones included here in root are scoped _globally_. This isn't a concept specific to `root.tsx` - other style files you import into components will be scoped to the children of those components as well (as you'd normally expect with CSS).

 Let's look at what it does in practice. We'll be using the exact `links` function that is present immediately after you create a fresh Remix project, in `root.tsx`. 

```javascript
export let links: LinksFunction = () => {
  return [
    { rel: "stylesheet", href: globalStylesUrl },
    {
      rel: "stylesheet",
      href: darkStylesUrl,
      media: "(prefers-color-scheme: dark)"
    },
    { rel: "stylesheet", href: deleteMeRemixStyles }
  ];
};
```

For our mental model, it's important to note that each of these objects has attributes that pertain to a traditional `link` element. As web devs, we're used to `rel`, `href`, and so on. There's even a `media` attribute included!

Now, do `npm run dev`, open your browser to `localhost:3000`, then pop open the dev tools and dive in to the elements. If you expand the `<head>`, you'll see something along these lines:

```html
<link rel="stylesheet" href="/build/_assets/global-AKFP5T7A.css">
<link rel="stylesheet" href="/build/_assets/dark-APYDFYJA.css" media="(prefers-color-scheme: dark)">
<link rel="stylesheet" href="/build/_assets/remix-5PPS2YMF.css">
```

Looks pretty familiar (keep in mind that the random letters will not be the same for you as they were for me - those are for caching purposes and will change when those files are changed).

To finally drive this `links` function home, let's create, import, and inject our own stylesheet.

`touch app/styles/myStylesheet.css`

You don't really need to put anything in there, but for the sake of not using an empty stylesheet, you can drop this in:

```css
.hello {
  color: red;
}
```

Now head back over to `root.tsx` and add your CSS import under the other style imports near the top of the file:

```javascript
import deleteMeRemixStyles from "~/styles/demos/remix.css";
import globalStylesUrl from "~/styles/global.css";
import darkStylesUrl from "~/styles/dark.css";
import myStyles from "~/styles/myStylesheet.css";
```

After that, we can add our stylesheet to the `links` function so that it can be injected into the page's HTML:

```javascript
export let links: LinksFunction = () => {
  return [
    ...
    { rel: "stylesheet", href: myStyles }
  ];
};
```

The page will now reload in your browser and you should notice a new element: 
`<link rel="stylesheet" href="/build/_assets/myStylesheet-XMIPET4B.css">`

Done! We've learned not only exactly what the `links` function is doing, but also how to create and inject our own stylesheet into the global styles. We can use this same pattern in any component that we create in Remix - just keep in mind that only the ones declared here in `root.tsx` will be considered global.

# App, Document, Layout (and Outlet!)

If you're anything like me, you tend to get a little glossy-eyed when scrolling through some generated files and seeing big chunks of JSX that you yourself didn't create. I did that several times with `root.tsx` before resolving to actually look at what they're doing - so now come along with me while we do just that!

```javascript
export default function App() {
  return (
    <Document>
      <Layout>
        <Outlet />
      </Layout>
    </Document>
  );
}
```

This is - simply - just a React component called `App` that uses several other components to eventually wrap an `Outlet`. That Outlet is where the app we currently see in our browser actually lives - it's where the content for the `/` route is rendered. That's simple enough, but what's `Document` and `Layout`?

## The \<Document\> Component

A lot of this component should look familiar once you clear up your glossy-eyes. It is really just an `html` element with a `head`, `body`, and some other necessary stuff within. 

First up is `<head>` - the only Remix-specific elements here are the `<Meta />` and `<Links />` elements. `<Meta />` is the element where the meta tags from your individual components get injected. Throughout your Remix journey, you'll use functions that look like this (taken directly from the docs):

```javaascript
import type { MetaFunction } from "remix";

export let meta: MetaFunction = () => {
  return {
    title: "Something cool",
    description:
      "This becomes the nice preview on search results."
  };
};
```

This could be in any component - it's just a way for that component to set the title and description of the current route. That `meta` function is really connecting back up to this `<Meta />` element and injecting those attributes. You can prove this in a few very simple steps:

`touch app/routes/meta.tsx`

```javascript
import type { MetaFunction } from 'remix';

export let meta: MetaFunction = () => {
  return {
    title: 'META TITLE',
    description: 'My Meta Description for /meta route'
  }
}

export default function Meta() {
  return null;
}
```

Now save and navigate to `localhost:3000/meta` - in the tab at the top of your browser you should see `META TITLE`, and if you pop open your dev tools and check out the elements, you should also see this:

`<meta name="description" content="My Meta Description for /meta route">`

And that's all the `<Meta />` element does, fundamentally!

Now, how about `<Links />`? This one isn't much more complicated than `<Meta />`. All it is is the place where the stylesheets for the current route are rendered. Comment it out, and you'll notice that your app suddenly has no CSS (make sure to uncomment this before moving on, unlike me when I was writing this and then getting confused).

Let's verify this in the same way we did with the Meta element. Inside that same `meta.tsx` component, let's import a new stylesheet and use a `links` function to inject it -

`touch app/styles/myStylesheet.css`

```
.hello {
  color: red;
}
```

```javascript
import type { MetaFunction, LinksFunction } from 'remix';
import myStyles from 'myStylesheet.css';

...

export let links: LinksFunction = () => {
  return [
    { rel: "stylesheet", href: myStyles }
  ]
};

export default function Meta() {
  return null;
}
```

Now save, refresh your browser again, and check out the `<head>` element - you should see a script tag similar to this: 

`<link rel="stylesheet" href="/build/_assets/myStylesheet-XMIPET4B.css">`

For the fun part, navigate back to your root route (`/`) and check the `<head>` again. You'll notice that the import for `myStylesheet.css` is gone - since the route doesn't need it and doesn't import it, it's totally removed from the page!

Alright, now let's get to the slightly scarier `<body>` element. This has a few Remix-specific components in it, but there's actually not much we need to worry about here. Our Remix apps as we know them are rendered within the `{children}` prop here (which is really the `<Layout>` and `<Outlet>` elements from above... there's a few levels to this, obviously). If you're familiar with React and `children` props in general, then this shouldn't be too new to you. If you aren't, I highly recommend reading up on [composition in React](https://reactjs.org/docs/composition-vs-inheritance.html).

### <RouteChangeAnnouncement /> 
This is a convenience and accessibility component provided by Remix to give screen readers a prompt when the route changes. If you want to test it out, enable the screen reader of your choice and navigate to a couple routes of your choosing (and creation)! As with our following component, this is a good one to just trust and leave. Your users will thank you.

### <ScrollRestoration />
This is a Remix-specific component (but would be useful everywhere, really) that mimic's the browser's scroll restoration when the location (route) changes. You can check out the docs for this [here](https://remix.run/docs/en/v1/api/remix#scrollrestoration), and I also highly recommend checking out the [PR for it](https://github.com/remix-run/remix/pull/399) to look over the code (which lives in `packages/remix-react/scroll-restoration.tsx`, if you have the repo forked). 

Unless you are really, _really_ sure of what you're doing, you should ignore this component and simply let it happily live its life where it currently is - and especially trust that it's doing its job. 

### <Scripts />
We're nearly done with `<Document>`! `<Scripts />` is a pretty straight forward (at least for us as users) component, much like every other Remix-specific component we've dealt with so far. What it does is simply injects the necessary `<script>` elements into the page - so all your wonderful JavaScript! As you navigate around your app, you can see these script tags change as some bundles become necessary (like for the route you're currently on) and some become unnecessary and therefore get unloaded (like when you leave that route). There's a lot going on under the hood here which we'll dissect in future articles. For now, all you need to understand is that this component handles all your JavaScript file/bundle injection for you - no manual intervention needed.

And with that, we're done with `<Document>`! Hopefully that wasn't too bad and you learned some things (just as I did) along the way. Remix has a lot of stuff going on under the hood as you can see in even just these examples, but it's doing these things in such a way that they're approachable and make sense once you break them down a little bit and see them in action - like with the `Meta` and `Links` components. 

