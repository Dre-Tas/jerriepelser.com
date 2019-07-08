---
title: "Sorting out the Gatsby folder structure for my blog"
description:
  In this blog post, I continue migrating my personal website to Gatsby. Specifically, I am looking at how to sort out the folder structure for my blog posts.
tags:
- blogging
- hugo
- gatsby
---

In my [previous blog post](/blog/getting-started-gatsby-blog-starter-kit/), I started using the Gatsby blog starter kit, and at the end of that blog post, I listed several issues which I had to resolve. In this blog post, I will look at addressing these issues, namely:

1. Ensuring blogs posts are located in the path `/blog/*`
1. Moving all blog posts to their own directory.
1. Extracting the date of a blog post from the path (and removing the date from the path)

## Hosting blog posts under the /blog folder

When you use the blog starter kit, all blog posts are located under the root folder of the website.

![Incorrect path for blog posts](/images/blog/2019-07-08-sorting-out-gatsby-folder-structure/blog-path-incorrect.png)

I want all blog posts to be located under the `/blog/`, as that is where they are currently located and I do not want to do a bunch of redirects for all the blog posts to a new location. Also, I prefer them under the `/blog/` path from an organizational point of view.

By default, Gatsby will use the path of a file to determine the URL. If you look at the folder structure created by the blog starter kit, you can see that all blog posts are located under the `/content/blog` folder. So, if Gatsby is reading the files from the `/content` folder, why do my blog posts not contain `/blog/` in their path?

The reason is that the blog starter kit is actually configured to read content from the `/content/blog` folder, not the `/content` folder, as you can see in `gatsby-config.js`.

```json
module.exports = {
  siteMetadata: {
    ...
  },
  plugins: [
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        path: `${__dirname}/content/blog`,
        name: `blog`,
      },
    },
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        path: `${__dirname}/content/assets`,
        name: `assets`,
      },
    },
    ...
  ],
}
```

So, the solution is to change the configuration to read all content from the `/content` folder, which is what I want as I'll be adding other types of content later.

```json
module.exports = {
  siteMetadata: {
    ...
  },
  plugins: [
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        path: `${__dirname}/content`,
        name: `content`,
      },
    },
    {
      resolve: `gatsby-source-filesystem`,
      options: {
        path: `${__dirname}/content/assets`,
        name: `assets`,
      },
    },
    ...
  ],
}
```

With that change, all blog posts are now located under the `/blog/` path.

![Blog post are under the right path](/images/blog/2019-07-08-sorting-out-gatsby-folder-structure/blog-path-correct.png)

## Moving all blog posts to their own directory

The second part I want to do is to move all blog posts to their own directory. Last time I copied a few of my existing blog posts from Hugo to Gatsby and placed them in the blog folder. However, I want to follow the current convention of the blog starter kit which puts each blog posts in its own folder.

![Blog posts in folders vs files](/images/blog/2019-07-08-sorting-out-gatsby-folder-structure/blog-posts-folder-vs-file.png)

I like the way the starter kit is doing it as it allows me to put the markdown file and all images related to a blog post in the same directory. I name the directory the same as the existing file name, and then place the markdown file inside the directory as an `index.md` file.

![Blog posts in their own directory](/images/blog/2019-07-08-sorting-out-gatsby-folder-structure/blog-posts-in-own-folder.png)

As I mentioned earlier in this blog post, Gatsby will create a URL for each blog post based on its directory structure. The code responsible for that is located in `gatsby-node.js`. Specifically, it is the call to [createFilePath](https://www.gatsbyjs.org/packages/gatsby-source-filesystem/#createfilepath).

```js
exports.onCreateNode = ({ node, actions, getNode }) => {
  const { createNodeField } = actions

  if (node.internal.type === `MarkdownRemark`) {
    const value = createFilePath({ node, getNode })
    createNodeField({
      name: `slug`,
      node,
      value,
    })
  }
}
```

The `createFilePath` will convert use the directory structure of a file to determine the path. If the file is named `index.*`, the file name will not become part of the path. So, the file named `/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/index.md` will be converted to the path `/blog/2019-06-26-use-conveyor-access-iis-app-over-internet`.

## Extracting the date of a blog post from the path

In Hugo, the date of a blog post was determined from the filename. The first 10 characters on a blog post contained the date in the format `yyyy-mm-dd`. I want to keep this structure, but that means I need to do some special processing for blog posts to extract the date from the path. Also, you will notice that the actual date appears in the path of the blog post:

![Blog post date is in the path](/images/blog/2019-07-08-sorting-out-gatsby-folder-structure/date-in-path.png)

Once I have extracted the date, I need to remove it from the path, so `/blog/2019-06-26-use-conveyor-access-iis-app-over-internet/` should become `/blog/use-conveyor-access-iis-app-over-internet/`.

I got the answer for how to do this from the [source code of the React blog](https://github.com/reactjs/reactjs.org). They also have the date in the filename, and then use that to determine the date (and path) of each blog post.

Using the same principles as the Reacy blog, I updated my `onCreateNode` function as follows:

```js
exports.onCreateNode = ({ node, actions, getNode }) => {
  const { createNodeField } = actions

  if (node.internal.type === `MarkdownRemark`) {
    const slug = createFilePath({ node, getNode })
    const match = BLOG_POST_FILENAME_REGEX.exec(slug)
    if (match !== null) {
      const year = match[1]
      const month = match[2]
      const day = match[3]
      const filename = match[4]
      const date = new Date(year, month - 1, day)
      
      createNodeField({
        name: `slug`,
        node,
        value: `/blog/${filename}`,
      })

      createNodeField({
        name: `date`,
        node,
        value: date.toJSON(),
      })
    } else {
      createNodeField({
        name: `slug`,
        node,
        value: slug,
      })
    }
  }
}
```

The updated function will check if the slug matches the regex, in which case we know we are dealing with blog posts. The function then uses the regex match to extract the year, month and day, as well as the filename from the existing slug. Using the date parts, I reconstruct a proper date and using the filename, I construct a new slug without the date in it. I also needed to update my GraphQL queries to query the date field and update my templates to use the date field to display the date, as opposed to the date specified in frontmatter.

With that in place, I now have proper blog posts with the date defined, and the path with the date removed:

![Correct date and path for blog posts](/images/blog/2019-07-08-sorting-out-gatsby-folder-structure/correct-blog-post-date.png)

## Conclusion

In this blog post, I addressed some issues I have with the folder structure of my blog posts. With this in place, I can now write a utility application that will help me migrate all the blog posts from the Hugo blog to the Gatsby blog.