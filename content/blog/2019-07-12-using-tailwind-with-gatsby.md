---
title: "Using Tailwind with Gatsby"
description:
  I continue the migration of my website from Hugo to Gatsby. In this blog post, I look at removing the styling that is included in the blog starter kit and add the Tailwind CSS library to my website.
tags:
- blogging
- gatsby
- tailwindcss
---

The Gatsby blog starter kit I have used to kickstart my migration from Hugo to Gatsby makes use of the [Typography plugin](https://www.gatsbyjs.org/packages/gatsby-plugin-typography/) for some basic styling, along with a couple of custom typefaces. For my website, I want to make use of the [Tailwind CSS library](https://tailwindcss.com/). In this blog post, I demonstrate the steps I used to remove the Typography styling and add Tailwind to my website.

## Removing the starter kit styling

Removing the existing styling from the starter kit was as simple as removing a few NPM packages, and cleaning up the code to remove all usage of these packages. In particular, I removed the following NPM packages:

* `gatsby-plugin-typography`
* `react-typography`
* `typeface-merriweather`
* `typeface-montserrat`
* `typography`
* `typography-theme-wordpress-2016`

I also had to remove the usages in the source code. For all the changes I made, refer to [this commit](https://github.com/jerriep/website/commit/569b0374da85733ea5bbec8990279a6efc5194b2).

## Installing Tailwind

Installing Tailwind was pretty straightforward and I followed the [Gatsby documentation](https://www.gatsbyjs.org/docs/tailwind-css/) for the most part. The only missing part was how to create a new CSS file which uses Tailwind.

First, install the `tailwindcss` and `gatsby-plugin-postcss` packages:

```bash
npm install tailwindcss gatsby-plugin-postcss --save-dev
```

Next, you can create a [Tailwind Configuration file](https://tailwindcss.com/docs/configuration). It is not needed, but I did this as I know I want to play around with this later to try and [reduce the size of the generated CSS](https://tailwindcss.com/docs/controlling-file-size).

```bash
./node_modules/.bin/tailwind init
```

This will create a `tailwind.config.js` file in the root of the project which you can use to [customise Tailwind](https://tailwindcss.com/docs/configuration).

You will also need to create a `postcss.config.js` file in the root directory of your website:

```js
module.exports = () => ({
  plugins: [require("tailwindcss")],
})
```

And include the PostCSS plugin for Gatsby in your `gatsby-config.js`:

```js
plugins: [`gatsby-plugin-postcss`],
```

The final bit is to create a global CSS file and include it in Gatsby. I created a file called `/src/styles/site.css` with the following content:

```css
@tailwind base;

@tailwind components;

@tailwind utilities;
```

You can also refer to the [Tailwind CSS Installation docs](https://tailwindcss.com/docs/installation/) for more information on this.

Once the file has been created, you need to update your `gatsby-browser.js` to import this CSS file:

```
import "./src/styles/site.css"
```

For more information, refer to the Gatsby tutorial on [Styling in Gatsby](https://www.gatsbyjs.org/tutorial/part-two/). You can also view [this commit](https://github.com/jerriep/website/commit/a35398e82d7f819f0b8d4d1ed696bc12f0f5f35b) to see all the changes I made to include Tailwind.

## Using Tailwind

You can now start using the Tailwind utility classes in your website. For example, to use the [Tailwind container](https://tailwindcss.com/docs/container), you can specify it in the `className` prop as follows:

```jsx
import React from "react"
import { Link } from "gatsby"

class Layout extends React.Component {
  render() {
    // some code omitted

    return (
      <div className="container mx-auto">
        <header>{header}</header>
        <main>{children}</main>
        <footer>
          © {new Date().getFullYear()}, Built with
          {` `}
          <a href="https://www.gatsbyjs.org">Gatsby</a>
        </footer>
      </div>
    )
  }
}

export default Layout
```

## Conclusion

In this blog post, I demonstrated how you could remove the styling that is included in the Gatsby blog starter kit and replace it with Tailwind. I also showed how you could start using the Tailwind utility classes in your components.