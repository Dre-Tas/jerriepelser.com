---
title: "Migrating my website from Hugo to Gatsby"
description:
  I have decided to migrate my personal website from Hugo to Gatsby and write a series of blog posts on the process. In this first blog post, I'll discuss the reasons for the change as well as my decision to go with Gatsby.
tags:
- blogging
- hugo
- gatsby
---

## Introduction

Since starting with this website in 2013, I have used various blogging platforms. I initially started with WordPress, then [moved to Statamic](/blog/switching-to-statamic/), and then later to [Hugo](https://gohugo.io/). There is no real rational reason for these changes, other than the fact that I have always considered my personal website to be a bit of a playground to experiment with things.

I have decided to make a change again and switch my website from Hugo to [Gatsby](https://www.gatsbyjs.org/). As of the writing of this particular blog post on 1 July 2019, my website is still running on Hugo, so consider this the start of the journey. I'll be blogging about my experience as I go along.

## The decision to go with Gatsby

I want to quickly discuss my reasons for going with Gatsby over any of the other potential options out there.

### 1. It is a chance to learn React

I have never made use of React, but I have been interested in learning it for a long time. Gatsby is based on React, so, even though I guess I am strictly not learning React itself, I will be learning about many of the concepts which make up a React application. These are skills that I can use later, should the opportunity arrive to use React in a proper project.

### 2. I will have more control

With Hugo, I would run into issues every so often in that I cannot do things that fall outside of the structure of the underlying framework. One such example was when I was trying to create the sidebar for the [Airport Explorer book](https://www.jerriepelser.com/books/airport-explorer). I had to deal with Hugo's proprietary syntax and ran into several limitations, and even though I managed to produce something acceptable in the end, it was not a pleasant experience.

With Gatsby, on the other hand, I have access to the full power of the JavaScript, React and Gatsby ecosystems. I believe this will make it much easier to bend it to my will.

### 3. The Gatsby ecosystem is extensive

As mentioned in the previous point, the Gatsby ecosystem is extensive. The Gatsby website lists, at the time of writing this, [over 900 plugins](https://www.gatsbyjs.org/plugins/) that are available for you to use. Add to that the fact that you also have the React and JavaScript ecosystems at your disposal, and I think the possibilities become endless.

## The plan

So, the plan of action is first to create a website that has parity with the current one. I have also been itching to use [Tailwind](https://tailwindcss.com/) for a while, so I'll be using that for the styling and theming of the website.

Once that is in place, I plan to add some more features to the website, such as blog series, more books, integration with Algolia etc.

I hope you'll learn with me along the way. Stay tuned!