---
title: "Built my blog with Next.js App Router and Payload CMS"
description: "I would like to share how I build my blog using Next.js App Router and Payload CMS."
tags: ["Next.js", "Payload CMS", "Software Development"]
publishedAt: "2024-01-25T15:00:00.000Z"
updatedAt: "2024-01-25T15:00:00.000Z"
---

Once you have some understanding of web and front-end development techniques, you may want to create your own website. That is what I did. I worked as an application developer and ran my own website.

Over the New Year holidays, I spent my free time renewing the blog site's tech stack, so I would like to share how I chose the tech stack and built the blog.

## Tech stack of the former blog

Before we get to the main topic, let me introduce the tech stack of the blog I used previously.

- [Next.js Page Router](https://nextjs.org/docs/pages/building-your-application/routing)
- CSS-in-JS (Emotion)
- Manage content with Markdown files
- Static Site Generation (SSG)

The tech stack I built three years ago was good enough for a one-person operation with no hassle, since the blog can be operated by simply updating Markdown files.

While I was running the blog with the above tech stack and had no performance issues with the site or display speed. However, I did notice the following issues

- No content management system (CMS), so managing content becomes more complicated as the amount of content increases.
- As the amount of content increases, the time required to generate a static site each time the content is updated increases.

## Tech stack of the blog I built this time

Now, let me introduce the tech stack of the blog.

- [Next.js App Router](https://nextjs.org/docs/app/building-your-application/routing)
- CSS Modules
- Content Management with Payload CMS
- Use GraphQL to display content dynamically

First, since there was no content management system (CMS), I introduced a CMS called "Payload" this time. This made it possible to manage content through the UI on the web.

Also, I updated the tech stack to be in line with the latest trends. For example, I switched to the Next.js App Router released in 2023. The technologies I introduced are described in more detail below.

## Front-end

### Next.js App Router

This time I built the frontend with [Next.js App Router](https://nextjs.org/docs/app/building-your-application/routing), which was released in 2023.

The need to completely switch from [Page Router](https://nextjs.org/docs/pages/building-your-application/routing) is not high yet, but since App Router will be used more and more in the future, this was also a learning experience for me.

The App Router has many practical updates, such as a [Layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts) component that makes it easier to optimize rendering of the same layout. On the other hand, there are many aspects where the know-how is not yet fully realized, such as the newly introduced "Server Components", and stable and fast development has not yet reached the stage.

As for how I handle the separation of "Server Components" and "Client Components", I basically treat the `app` folder as "Server Components" and place "Client Components" in the `component` folder based on the atomic design method.

The App Router itself has not been released for a long time, so there are many aspects I am still exploring, but I am sure it will be fine-tuned the more I use it.

### Styling

Since I wanted to do the detailed styling of the UI design myself, Tailwind CSS, which provides the framework, was not my first choice.

I was torn between CSS Modules and CSS-in-JS, but finally chose CSS Modules.

I have been using CSS-in-JS itself for several years and have some experience with it, but I could not shake off my concerns about the incompatibility of CSS-in-JS with Server Components, which the React development team has been focusing on recently, so I decided to switch to CSS Modules at this time.

### GraphQL Client

A GraphQL Client was required to retrieve content data in GraphQL. I chose URQL for this project.

- Apollo Client:
- - A well-documented and have a large number of users.
  - I've used it before.
- Relay:
- - Suitable for use with client-side React.
  - Few examples for server-side rendering and Node.js.
- URQL
- - Lightest bundle size
  - Guaranteed to work on server-side rendering and Node.js

This time, I needed to retrieve data during server-side rendering, so I chose URQL, which is the lightest.

### Internationalization and localization (i18n)

In Next.js Pages Router, [internationalization and localization (i18n) routing](https://nextjs.org/docs/pages/building-your-application/routing/internationalization) was supported by default, but in App Router, the [guide](https://nextjs.org/docs/app/building-your-application/routing/internationalization) is provided, but the support itself needs to be done by the developer.

In this project, I chose the library called [`next-international`](https://next-international.vercel.app/) to handle this issue.

The reasons for choosing this library were that it was compatible with App Router and that it is lightweight and type-safe.

### Performance

Performance has improved slightly due to the layout components of the Next.js App Router and the reduction in styling code due to the migration to CSS Modules.

In fact, performance has improved to the point where a 250 ms speedup can be measured, but honestly, the difference is not noticeable to the eye.

## CMS/GraphQL Server

### Frameworks

Initially, I considered using a headless CMS SaaS offered by Contentful and others.

However, considering long-term use, I decided to use an open source headless CMS that I could deploy and manage as a hosted server.

However, there were not many open source headless CMSs available, and I had to choose between the following two.

- [Payload](https://payloadcms.com/)
- [Strapi](https://strapi.io/)

Both are open source CMS frameworks written in JavaScript/TypeScript, and both have similar features such as support for GraphQL and REST APIs.

I had used Strapi before, but the development experience was not so good due to the need to write more complex code and the lack of type safety compared to Payload, and there seemed to be no other choice but to use Payload.

Payload itself is still under development as it has many beta features, but it is not a bad development experience as it allows to build a CMS with less code than Strapi and is type-safe for TypeScript.

### Text editors

Payload offers a choice between Slate.js and Lexical as text editors.

Lexical is an editor framework developed by Facebook (Meta) that allows users to write content using the same notation as Markdown and to extend the editor's functionality.

I chose Lexical for this project because of its future extensibility.

## Summary

Overall, I am satisfied with the project because it has achieved functionality that could not be achieved with the previous tech stack.

I think there are some improvements that will become apparent as operations continue, so I would like to continue updating the system.
