---
title: "Getting started with the Gatsby blog starter kit"
description:
  One of the first steps in moving my personal website over to Gatsby is to bring across the blog content. In this blog post I expore Gatsby's blog starter kit and look at my options for bringing my blog content across.
tags:
- blogging
- hugo
- gatsby
---

## Creating a new website with the starter kit

As I mentioned in the [previous blog post](/blog/migrating-blog-hugo-to-gatsby), I am migrating my website over to Gatsby. Initially, I considered starting from a blank slate and build everything up from scratch, but I dropped that idea and decided to speed things up, by using the Gatsby [blog starter kit](https://www.gatsbyjs.org/starters/gatsbyjs/gatsby-starter-blog/).

Once I have [installed the Gatsby CLI](https://www.gatsbyjs.org/docs/quick-start#install-the-gatsby-cli), I created a new website with the blog starter.

```bash
gatsby new website https://github.com/gatsbyjs/gatsby-starter-blog
```

After creating the new Gatsby website, the first thing I did was to run it in development mode to see what it looks like. Unfortunately, I was off to a bad start:

![Error when running Gatsby the first time](/images/blog/2019-07-04-getting-started-gatsby-blog-starter-kit/error-when-running-gatsby.png)

Googling the error "_Encountered duplicate defintitions for one or more documents: each document must have a unique name_", I came across [this issue](https://github.com/gatsbyjs/gatsby/issues/11688) on GitHub. It appears as though there is some discrepancy in the way directory names are handled, and in some parts, it is handled as-is, and in other parts, it is converted to lower case. This, in turn, results in some case-sensitivity issues which causes the error. You can notice in the screenshot above the discrepancies in the directory names.

My first reaction was to update all the NPM packages to the latest version, but that did not resolve the problem. So, I took the easy way out and renamed my directories to all be lower case. That worked around the issue and I could run the starter kit.

![Running the blog starter kit](/images/blog/2019-07-04-getting-started-gatsby-blog-starter-kit/starter-kit-running.png)

## Understanding where content comes from

The next thing was to understand where those blog posts come from. The starter kit is configured in such a manner that it picks up the blog posts from the content folder. You can see that the `content` folder contains a `blog` folder, which in turns contain folders for each of the blog posts. Each blog post folder contains an `index.md` file with the markdown (and frontmatter) for the blog post, along with any images.

![The blog content folder](/images/blog/2019-07-04-getting-started-gatsby-blog-starter-kit/blog-content-folder.png)

In the `gatsby.config` folder, the `gatsby-source-filesystem` plugin is configured to pick up the content from the blog folder.

```json
plugins: [
  {
    resolve: `gatsby-source-filesystem`,
    options: {
      path: `${__dirname}/content/blog`,
      name: `blog`,
    },
  },
  ...
],
```

The blog starter kit also has the `gatsby-transformer-remark` plugin configure which will convert any markdown file (`.md` or `.markdown`) into HTML content and expose those blog posts in the `allMarkdownRemark` field, as you can see in the simple GraphQL query below which lists the blog posts with their title and slug.

![Running a GraphQL query to list the blog posts](/images/blog/2019-07-04-getting-started-gatsby-blog-starter-kit/markdown-remark-query.png)

The blog starter kit has a `BlogPostTemplate` defined in `blog-post.js` which will render all of the blog posts.

At this point, I would suggest also reading through the following official Gatsby docs:

* [Introduction](https://www.gatsbyjs.org/docs/)
* [Gatsby Project Structure](https://www.gatsbyjs.org/docs/gatsby-project-structure/)
* [Sourcing Content and Data](https://www.gatsbyjs.org/docs/content-and-data/)
* [Building with Components](https://www.gatsbyjs.org/docs/building-with-components/#page-template-components)

## Testing my existing content

I spent an hour or so working through the source code and docs trying to get a better understanding of how things fit together. So, I thought it was time to see what happens when I bring some of my existing content into the mix.

In my existing Hugo-based website, I put all the blog posts under the `/content/blog` folder - which is strangely enough the same layout the blog starter kit uses.

![Hugo website blog folder structure](/images/blog/2019-07-04-getting-started-gatsby-blog-starter-kit/hugo-blog-post-layout.png)

I decided to try it out with my three latest blog posts by copying them from the Hugo website to the Gatsby one.

![Copying the three most recent blog posts to Gatsby](/images/blog/2019-07-04-getting-started-gatsby-blog-starter-kit/copying-recent-blog-posts-to-gatsby.png)

Switching back to the browser and refreshing my Gatsby website, you can see those three blog posts listed.

![Three most recent blog posts listed](/images/blog/2019-07-04-getting-started-gatsby-blog-starter-kit/three-most-recent-blog-posts-on-gatsby.png)

## Issues with existing content

At this point, I am happy about making progress, but there are several issues.

1. The blog starter kit puts each blog post in its own folder. My existing Hugo structure has all blog posts as individual files under the same folder. I like the Gatsby layout more for reasons I'll discuss next time.
1. All blog posts are on the path `/blog-post-slug`. My existing website has all blog posts are on the path `/blog/blog-post-slug`. I want to keep the blog posts under `/blog/*` both for organizational purposes but also to ensure I don't have broken links when I migrate across.
1. My blog posts don't have dates. There is no frontmatter for the date as it is derived from the filename. You can see in the screenshots above that in Hugo all the filenames for the blog posts start with the date (in `yyyy-mm-dd` format).
1. Also, because of the above, the actual slug for the blog posts I copied from Hugo contains the date in the slug for the blog posts (e.g. `/2019-06-26-use-conveyor-access-iis-app-over-internet`). I do not want this.

In the next blog post, I will start work on sorting out these issues.

BTW, the source code for the new website is hosted at [https://github.com/jerriep/website](https://github.com/jerriep/website) if you want to have a look.