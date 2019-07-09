---
title: "Migrating blog posts from Hugo to Gatsby"
description:
  I continue the migration of my website from Hugo to Gatsby. In this blog post, I look at moving the markdown files with the blog posts as well as the images into the Gatsby website.
tags:
- blogging
- hugo
- gatsby
---

## Copying the files across 

In yesterday's blog post, I [configured the Gatsby folder structure for my blog posts](/blog/sorting-out-gatsby-folder-structure). Now that the configuration is done, I can move the content across. Since there is some manipulation of the files and folders required, I cannot just copy the files from the Hugo website as-is. Instead, I decided to write a C# application to take care of this part for me.

This C# application will do the following:

* Scan the Hugo blog post folder for all blog post markdown files
* For each blog post:
    * Create a new folder in the Gatsby website with the name of the file
    * Copy the markdown file to the folder as `index.md`
    * Parse the markdown contents of the blog post to get all images
    * Copy all of the images across to the folder in the Gatsby website

Here is the code for the application. I used the [Markdig package](https://www.nuget.org/packages/Markdig/) for the parsing of the markdown file to extract the images.

```cs
using System;
using System.IO;
using System.Linq;
using Markdig;
using Markdig.Syntax;
using Markdig.Syntax.Inlines;

namespace BlogMigrator
{
    internal class Program
    {
        private const string HugoStaticPath = @"C:\development\jerriep\jerriepelser.com\static";
        private const string HugoBlogPostsPath = @"C:\development\jerriep\jerriepelser.com\content\blog";
        private const string GatsbyBlogPostsPath = @"C:\development\jerriep\website\content\blog";

        private static void Main(string[] args)
        {
            ProcessBlogPosts();
        }

        private static void ProcessBlogPosts()
        {
            var sourceFiles = Directory.GetFiles(HugoBlogPostsPath, "*.md");
            foreach (var sourceFile in sourceFiles) ProcessSourceBlogPost(sourceFile);
        }

        private static void ProcessSourceBlogPost(string sourceFile)
        {
            var fileInfo = new FileInfo(sourceFile);
            var filename = fileInfo.Name;

            if (filename != "_index.md")
            {
                string blogPostFolderName = Path.GetFileNameWithoutExtension(sourceFile);
                string gatsbyBlogPostFolderName = Path.Combine(GatsbyBlogPostsPath, blogPostFolderName);
                Directory.CreateDirectory(gatsbyBlogPostFolderName);
                File.Copy(sourceFile, Path.Combine(gatsbyBlogPostFolderName, "index.md"), true);

                var document = Markdown.Parse(File.ReadAllText(sourceFile));
                var links = document.Descendants<ParagraphBlock>().SelectMany(x => x.Inline.Descendants<LinkInline>())
                    .Where(l => l.IsImage);
                foreach (var link in links)
                {
                    string imagePath = HugoStaticPath + link.Url.Replace('/', '\\');
                    var imageFileInfo = new FileInfo(imagePath);

                    File.Copy(imagePath, Path.Combine(gatsbyBlogPostFolderName, imageFileInfo.Name), true);
                }
            }
        }
    }
}
```

When I run the application, it copies all the blog posts across and places the blog post content and its related images all in the same folder:

![The blog post folders each with their content and images](/images/blog/2019-07-09-migrating-blog-posts-from-hugo-to-gatsby/blog-post-folders-with-content-and-images.png)

## Fixing the image URLs

At this point, all the files are moved across to the correct place, but the images in the blog post markdown files still point to the wrong place. For example, from the blog post from yesterday, you can see that my images are located in a `/images/blog/*` folder structure.

```md
The blog starter kit also has the `gatsby-transformer-remark` plugin configure which will convert any markdown file (`.md` or `.markdown`) into HTML content and expose those blog posts in the `allMarkdownRemark` field, as you can see in the simple GraphQL query below which lists the blog posts with their title and slug.

![Running a GraphQL query to list the blog posts](/images/blog/2019-07-04-getting-started-gatsby-blog-starter-kit/markdown-remark-query.png)

The blog starter kit has a `BlogPostTemplate` defined in `blog-post.js` which will render all of the blog posts.
```

I should change that to point to the image in the same folder, for example:

```md
The blog starter kit also has the `gatsby-transformer-remark` plugin configure which will convert any markdown file (`.md` or `.markdown`) into HTML content and expose those blog posts in the `allMarkdownRemark` field, as you can see in the simple GraphQL query below which lists the blog posts with their title and slug.

![Running a GraphQL query to list the blog posts](markdown-remark-query.png)

The blog starter kit has a `BlogPostTemplate` defined in `blog-post.js` which will render all of the blog posts.
```

Doing this manually would be a time-consuming task, so I used Visual Studio Code's regular expression-based search and replace. For the search expression, I used `\!\[(.*?)\]\(\/(.+\/)*(.+\.\w+)(.*?)\)`. This regular expression defines four capture groups, which you can see in action here in the [Regex 101 regular expression tester](https://regex101.com/)

![The regular expression in action](/images/blog/2019-07-09-migrating-blog-posts-from-hugo-to-gatsby/regex-visualized.png)

Capture group 1 is the alt text, capture group 2 the path, capture group 3 the file name, and capture group 4 the title text.

For the replace expression, I used `![$1]($3$4)`. The expression builds up a new markdown image tag using the alt text, file name and title text, as you can see in action here in the search and replace screen in Visual Studio Code:

![Search and replace images](/images/blog/2019-07-09-migrating-blog-posts-from-hugo-to-gatsby/vs-code-search-replace-images.png)

## Verifying all images are correct

With the image paths fixed up, I wanted to confirm that everything was ok. Once again, going through each blog post manually would be too time-consuming, so I downloaded [Screaming Frog SEO Spider](https://www.screamingfrog.co.uk/seo-spider/) to crawl the website and find broken images.

On the first crawl, SEO spider returns a total of one page (the home page) along with some JavaScript files referenced by the page.

![First SEO Spider crawl returning only home page](/images/blog/2019-07-09-migrating-blog-posts-from-hugo-to-gatsby/seo-spider-crawl-1.png)

It appears that SEO Spider wants a sitemap file in order to understand the structure of the website. So, I installed and configured the [Gatsby sitemap plugin](https://www.gatsbyjs.org/packages/gatsby-plugin-sitemap/). This plugin only generates a sitemap in production mode, so instead of serving the website with `gatsby develop`, you'll need to use `gatsby build && gatsby serve`.

This time around SEO Spider picks up all of my website's content, and after the crawl, I head over to the **Response Codes** tab and look for HTTP 4xx response codes. Most of those are broken external links, but there appear to be three image URLs that are broken, so I can go and fix those.

![Client errors in SEO Spider](/images/blog/2019-07-09-migrating-blog-posts-from-hugo-to-gatsby/seo-spider-crawl-2.png)

## Conclusion

In this blog post, I showed how I migrated the blog post content and images for my Hugo website over to Gatsby. I also showed how I fixed the image paths in the markdown, as well as used SEO Spider to look for any broken images.