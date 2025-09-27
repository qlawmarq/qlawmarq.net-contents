---
title: "What is The best way to start building React apps in 2023"
description: "My opinion on what is the best way to build a React app in 2023, in comparison with Next.js, Remix, CRA, Vite, etc."
tags: ["React", "Software Development"]
publishedAt: "2023-12-10T15:00:00.000Z"
updatedAt: "2023-12-10T15:00:00.000Z"
---

By 2023, there are multiple libraries and frameworks for front-end web development, including React, Vue, Angular, and Svelte. Furthermore, even with just React as a library, there are libraries derived from React such as Next.js, Remix, CRA, and Vite. (In this article, for convenience, I will refer to all of these simply as “libraries.”)

Because web front-end technology tends to evolve quickly, content that was written only a few years ago can easily become outdated. I’ve been involved in web front-end, full-stack web, and mobile app development for about five years, and even within that period, I’ve seen a lot of changes in the libraries I use. In this article, I’ll share my opinions on which React-based library is the best choice for starting React development at the end of 2023.

## Create React App (CRA)

As its name suggests, Create React App (CRA) is a build tool for creating apps with React. It is an official tool maintained by the creators of React. It used to be widely used, and I’ve worked on a few web front-end projects based on CRA. However, as of 2023, the React development team no longer recommends using it.

In line with this, the documentation for starting React development has removed references to setting up a project with CRA. Currently, the React development team recommends using libraries like Next.js or Remix for creating React-based apps.

[https://react.dev/learn/start-a-new-react-project](https://react.dev/learn/start-a-new-react-project)

Moreover, the last release of CRA was in 2022, and it hasn’t been actively maintained since. It is fair to say that CRA has served its purpose by 2023. For more details on why CRA has reached the end of its life, see:

[https://github.com/reactjs/react.dev/pull/5487#issuecomment-1409720741](https://github.com/reactjs/react.dev/pull/5487#issuecomment-1409720741)

## Next.js

Next.js is a React-based library developed by Vercel and designed to build full-stack web applications. To put it simply, I believe that Next.js is the best choice as of 2023. Here are the reasons:

- It is regularly maintained and updated.
- It is optimized for performance and easy to work with.
- It supports various patterns, such as static site generation and custom servers.
- It has a strong community and a large user base.

Before Next.js, I remember having to set up routing, server-side rendering, image sizing, caching, build configurations, and other optimizations on my own. Next.js helps simplify these tasks considerably. Ever since its initial release in 2016, Next.js has been continually updated, bringing annual improvements in optimization and performance. For instance, static site generation was introduced in version 9, internationalized routing (i18n) in version 10, and in 2023, the App Router and Server Components were introduced to further optimize rendering performance.

In addition, many third-party tools and libraries are developed specifically for Next.js by the community and companies. For example, the monitoring platform Sentry has an SDK designed just for Next.js. Because it has many users and adopters—including large companies—it’s easier to troubleshoot issues, making it a good choice for beginners as well.

I have been using Next.js since 2019 and have rarely experienced framework-related issues; it has been stable in my experience. In 2023, a major update was the introduction of the App Router, but Pages Router remains supported. Personally, I use the App Router in new projects or when performance optimization is required, while I still opt for the Pages Router for existing projects or when migrating from another React-based app to Next.js.

## Remix

Remix is a React-based library for building full-stack web applications, maintained by the Shopify team. It is relatively new and bears similarities to Next.js in that it enables full-stack development. Some key differences from Next.js include:

- A heavy focus on server-side rendering.
- Greater emphasis on performance optimization.
- A design philosophy that adheres closely to web standards.

In recent times, the React development team has been working on further server-side optimizations, such as React Server Components, to improve performance. Remix was an early adopter of server-side features and has been dedicating itself to performance improvements from the start. Additionally, Remix actively leverages web-standard APIs and mechanisms, aiming to keep the library’s proprietary concepts to a minimum, which generally lowers the learning curve.

In 2023, I experimented with building a few websites in Remix to get a feel for it. The approach to routing and how components are split within routes is straightforward, and in some areas, I think it even surpasses Next.js. However, my conclusion is that it may still be premature to use Remix in production in 2023.

There are several reasons, but in short, the development team, community, and third-party support are still weaker compared to Next.js, raising concerns about inadequate long-term library support. For example, I needed to implement i18n (internationalization) in a Remix-based application. As of 2023, there was no official documentation for i18n, and only a single library existed for internationalization in Remix. For larger services or long-term operations, community and library support seems too sparse, so it’s best to be cautious about using Remix in production.

In contrast, Next.js officially documents i18n support, and there are multiple libraries to choose from, making it feel more reliable for long-term use. Also, since the mid-2023 release of the App Router in Next.js, performance optimizations via React Server Components are possible, so I think trying that first might be a better approach.

## Vite

Vite is a build tool that supports multiple front-end libraries such as React, Vue, and Svelte. With CRA now deprecated, Vite is one way to start a React project without depending on a specific framework. However, my conclusion is that I do not recommend it. The following link explains it in more detail, but essentially, you would have to handle more complex optimizations yourself:

[https://nextjs.org/docs/app/building-your-application/upgrading/from-vite](https://nextjs.org/docs/app/building-your-application/upgrading/from-vite)

In 2023, the React development team has been moving toward server-side optimizations like React Server Components. By contrast, Vite at this stage is still primarily centered on client-side React. If you want to use server-side rendering (SSR) with Vite, you need additional configurations. For now, I think it makes sense to simply take advantage of the optimizations provided by Next.js or Remix.

## Expo

Expo is an SDK based on React Native, mainly used for developing iOS and Android apps. It also supports building websites, enabling development for web/iOS/Android platforms all within Expo. However, keep in mind that styling and other constraints can become an issue for web compatibility.

I’ve built several iOS/Android apps using Expo. While it works, making the behavior and design consistent across different platforms can be quite challenging. Basically, if you need to develop apps for both iOS and Android, and you already have experience with React, then I can recommend Expo.

## Gatsby

Gatsby is a React-based library that, in my experience, is often used in scenarios similar to WordPress and other CMS-based sites. It originally focused on static site generation (SSG), although it appears to now also support server-side rendering (SSR).

Several years ago, I used Gatsby for blogs and landing pages with its static site generation feature, but once Next.js started supporting static site generation, I found fewer reasons to use Gatsby. Additionally, static site generation can become slower to build as the amount of content grows, which is a disadvantage. At this point, I don’t recommend Gatsby much anymore.

Compared to Next.js, Gatsby’s features appear lacking, and the Next.js community and user base seem much larger. Therefore, unfortunately, I can’t think of a strong reason to choose Gatsby today.

## Conclusion

- If you need stability or long-term support: **Next.js**
- For large-scale projects: **Next.js**
- For small-scale projects: **Next.js**
- For iOS/Android apps: **Expo**
- If you’re not sure what you need: **Next.js**

For reference:

[https://npmtrends.com/create-react-app-vs-expo-vs-gatsby-vs-next-vs-remix](https://npmtrends.com/create-react-app-vs-expo-vs-gatsby-vs-next-vs-remix)
