---
title: Using Tailwind UI and Next.js Together
subtitle: Installation and Setup
layout: post
date: 2020-04-04T12:00:00-04:00
tags: [reactjs, nextjs, tailwindcss, tailwindui, javascript]
---

[Next.js](https://nextjs.org) is an incredible framework for building server-side React applications. [Tailwind CSS](https://tailwindcss.com/) is a utility-first CSS framework that makes designing and building websites a breeze. And [Tailwind UI](https://tailwindui.com) is a commercial set of pre-baked components made in Tailwind CSS by two incredible designers, Adam Wathan and Steve Schoger. Using these tools together, I can develop whole web applications that look beautiful, from scratch, in record time.   

To figure out the right way to set them up together, I pulled the relevant pieces of documentation from each of Next.js, Tailwind CSS and Tailwind UI and synthesized them into this guide. I reference and link to those parts of documentation where appropriate. 

1. Set up Next.js with Tailwind CSS in development
2. Add Tailwind UI
3. Optimize for production with PurgeCSS

## 1. Set Up Next.js and Tailwind CSS in Development

First, we get a basic Next.js application going and make sure it works: 
```
npx create-next-app <project-name>
cd <project-name>
npm run dev
```
Confirm it works by navigating in your browser to `localhost:3000`, where you should see a page that looks something like this:

![Next.js Default Page](/img/next-js-screenshot.png)

Now, we install Tailwind CSS using the first two steps of the [Tailwind CSS installation docs](https://tailwindcss.com/docs/installation/).

```
npm install --save tailwindcss
```

Make the file `styles/index.css` with the following content:
```
@tailwind base;

@tailwind components;

@tailwind utilities;
```

Next.js uses PostCSS, and we use that to integrate Tailwind CSS. In the top level of your project directory, create the  `postcss.config.js` file with this content: 
```
// postcss.config.js
module.exports = {
  plugins: [
    'tailwindcss',
    'autoprefixer',
  ],
}
```


The Tailwind CSS installation instructions [suggests what to put in that file](https://tailwindcss.com/docs/installation#using-tailwind-with-postcss), but we need to use strings for plugin names instead of `require()` due to a [requirement of Next.js](https://nextjs.org/docs/advanced-features/customizing-postcss-config). 


If you create a `postcss.config.js` file as we have, it [completely overwrites the defaults used by Next.js](https://nextjs.org/docs/advanced-features/customizing-postcss-config). On first glance, the defaults have a lot of stuff in it like `postcss-flexbox-fixes` and `postcss-preset-env` along with some configuration that we seem to have ignored. This is intentional. 

Since by and large you won't be using CSS directly but will be using Tailwind CSS utility classes, you don't need that stuff and can just use the minimal `postcss.config.js` config from above.

Now, actually add the stylesheet to `pages/_app.jsx`, which [should globally add](https://nextjs.org/docs/basic-features/built-in-css-support#adding-a-global-stylesheet) Tailwind CSS to all of the components you create:

```
// pages/_app.jsx
import '../styles/index.css'

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}

export default MyApp
```

Edit `pages/index.jsx` to use some Tailwind CSS classes to make sure its working:

```
// pages/index.jsx
import Head from 'next/head'

const Home = () => (
  <div>
    <Head>
      <title>Next.js TailwindCSS</title>
      <link rel="icon" href="/favicon.ico" />
    </Head>

    <div className="container mx-auto">
        <h1 className="text-lg text-center m-4">TailwindUI/Next.js</h1>
        <p className="bg-green-600">This is a test of the tailwind next integration.</p>
    </div>
  </div>
)

export default Home
```

If it doesn't look like it working, you may need to stop the Next.js dev server and restart it because it may not pick up the changes to the `postcss.config.js` file unless you do a manual restart. 

If you're still stuck, Next.js publishes an [example app with Tailwind CSS](https://github.com/zeit/next.js/tree/v9.3.3/examples/with-tailwindcss) so you can see how your matches up.

## 2. Add Tailwind UI
Install Tailwind UI: 
```
npm install --save @tailwindcss/ui
```

Create a `tailwind.config.js` file and add Tailwind UI to it along with [the suggested modifications for fonts and sidebar spacing](https://www.notion.so/Tailwind-UI-Documentation-f9083ed0e2694690ac89253e88afb2b6#920134eaa0a8434bb31fffda3c001ddf):
```
// tailwind.config.js
const defaultTheme = require('tailwindcss/defaultTheme')

module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter var', ...defaultTheme.fontFamily.sans],
      },
    },
  },
  plugins: [
    require('@tailwindcss/ui'),
  ]
}
```
Add the recommended font to the `index.jsx` file `<Head>` tag:
```
...
<Head>
  <title>Next.js TailwindCSS</title>
  <link rel="icon" href="/favicon.ico" />
  <link rel="stylesheet" href="https://rsms.me/inter/inter.css" />
</Head>
...
```

To test that Tailwind UI is working properly, in `pages/index.jsx`, copy and paste in an element that does not have any Javascript (e.g., the Marketing Blog Section component). 

You may need find-and-relpace the `class` attribute to `className` if your IDE doesn't do this for your automatically. If it doesn't look exactly like the examples on the Tailwind UI component gallery, you may need to add the `antialiased` class to the highest level `<div>` tag.

## 3. Optimize for Production with PurgeCSS
Install PurgeCSS:

```
npm install --save @fullhuman/postcss-purgecss
```

Add PurgeCSS configuration to `postcss.config.js` that will only apply in production:
```
// postcss.config.js
module.exports = {
    plugins: [
        'tailwindcss',
        process.env.NODE_ENV === 'production'
            ? [
                '@fullhuman/postcss-purgecss',
                {
                    content: [
                        './pages/**/*.{js,jsx,ts,tsx}',
                        './components/**/*.{js,jsx,ts,tsx}',
                    ],
                    defaultExtractor: content => content.match(/[\w-/.:]+(?<!:)/g) || [],
                },
            ]
            : undefined,
        'autoprefixer'
    ],
}
```

You need to tell PurgeCSS to ignore Tailwind CSS's base and components classes using comment directives in `styles/index.css` [so that important base styles aren't purged](https://tailwindcss.com/docs/controlling-file-size#setting-up-purgecss): 

```
// styles/index.css
/* purgecss start ignore */
@tailwind  base;
@tailwind  components;
/* purgecss end ignore */

@tailwind  utilities;
```

When you run `npm run dev`, you may get warnings about that a "null PostCSS plugin was provided". That warning is an artifact of  `postcss.config.js` returning `undefined` for instead of PurgeCSS when you are not in production. It can be safely ignored.


