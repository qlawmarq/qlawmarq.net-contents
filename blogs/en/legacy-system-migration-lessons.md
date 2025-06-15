---
title: "Lessons Learned from a Legacy System Migration Project"
description: "A software engineer shares real challenges and practical solutions encountered during a legacy system migration project, including TCP socket issues, endianness problems, and the limitations of AI assistance."
tags: ["Software Development"]
publishedAt: "2025-03-24T15:00:00.000Z"
---

## Introduction

Throughout my career, I've primarily worked on new IT development projects. As a result, I've had few opportunities to work with what we call "legacy systems." However, last year, I had the chance to join a project tasked with replacing a legacy system.

Before actually working with the legacy system, I admittedly had somewhat underestimated the challenge, thinking, "With AI assistance available nowadays, I should be able to adapt to older technologies without much trouble." However, once I became involved, I discovered that beyond simply adapting to outdated technologies, there were many unexpected challenges such as insufficient documentation and unclear specifications.

Since this was such a valuable experience, I'd like to share what I learned from directly working with a legacy system. In short, it was a refreshing experience, much like an archaeological excavation.

## The Legacy System I Worked With

First, the legacy system I worked with wasn't a "pure" legacy system like COBOL or mainframe applications. Nevertheless, it had been in use for over ten years and incorporated technologies that are rarely used today, making it a legacy system in the broader sense.

The project I participated in involved developing an API to serve as an interface for replacing this legacy system with a new one. Since the service had tens of thousands of users, we designed what is typically called a "gateway" system to facilitate the migration period with parallel operations.

The target system, while not cutting-edge, was a relatively modern REST API running on AWS cloud services. In contrast, the legacy system operated on on-premises servers and communicated through TCP socket connections using fixed-length data exchanges—something rarely seen nowadays.

## What Happened

Let me document what problems we faced before my memory fades.

Simply put, I naively thought that if we could somehow handle unfamiliar technologies like TCP socket communication, the project would proceed smoothly. I assumed that even if questions arose, we could investigate using response data and log outputs, and we could also rely on AI for assistance.

In reality, however, the available information was extremely scarce, and we had to gather information through tedious, manual work.

### No Specifications and No One Who Understood Them

As expected, understanding the legacy system's specifications was challenging. It was a complete black box—no one understood the specifications, and no proper documentation existed for data exchanges.

Consequently, we had to obtain the source code of systems currently communicating with this legacy system and investigate what interfaces they used for data exchange.

Having source code serve as specifications is far from ideal, but since this was a system that hadn't been maintained for over ten years, it was within our expectations.

### Unable to Establish a Connection

After completing the code needed for communication, we planned to test the connection in our validation environment, but we encountered another issue.

The system had been used for over ten years, with few opportunities to establish new connections with other systems during that period, making the connection process difficult.

We were using Cloud Interconnect to connect the on-premises server with our Google Cloud validation environment, but the existing network configuration was so complex that investigating the infrastructure and networking required significant time and effort. Ultimately, it took several months before we could establish a connection.

### Mysterious Connection Timeouts

Just when we thought communication was finally possible, we were plagued by mysterious connection timeouts. Being TCP socket communication, if something happened on the server side, the error details wouldn't be transmitted to the client side.

Although we received logs from the legacy system, they contained no useful information, and with no one understanding the specifications, the cause of the timeouts remained unknown for some time. Eventually, we had to repeatedly compare our system with existing systems that had successfully established communication—a tedious process.

While comparing data through packet capture, we first noticed differences in TCP header sizes. Surprisingly, TCP headers originating from Windows differ in size from those originating from Linux. Our newly developed system was running on Linux machine images in K8s, while the existing system operated on Windows machines. We thought this might be the cause, so we attempted communication from Windows servers and made the TCP header size adjustable, but this turned out not to be the root cause.

In conclusion, the problem was a difference in endianness/byte order. While comparing packets from the existing system with those from the new system, we noticed subtle data differences. The issue was resolved by changing from little-endian to big-endian in certain areas. Honestly, I never expected to hear terms like "endianness" or "byte order" in a modern development environment in the 2020s.

## Was AI Helpful?

This development project had no restrictions on AI usage, which allowed us to utilize AI in various ways and gain valuable insights. Of course, we also used it to address legacy system-related issues.

To summarize, AI was partially helpful in this case but didn't assist in resolving the fundamental problems.

AI was beneficial in providing knowledge about concepts like "endianness/byte order." However, for resolving the root cause of connection timeouts, despite receiving various suggestions from AI, none led to a solution.

For example, when trying to identify "endianness/byte order differences" as the cause, we collaborated with AI to compare new and existing code, but couldn't find the cause. Only through observing actual communication data behavior—tedious manual work—could we identify the root cause.

The decisive factor was the limited information (context) we could provide to AI, such as documentation, detailed specifications, and sufficient log data.

Nevertheless, AI significantly improved productivity in development-related tasks, such as generating test code. It seems AI had difficulty analyzing the legacy system side where information was scarce.

## Lessons

Ultimately, it comes down to insufficient documentation organization and error logging on the legacy system side. Through this experience, I learned firsthand how such documentation deficiencies can become major challenges for future users of the system.

Personally, it was a good opportunity to explore unfamiliar concepts like "TCP header size" and "endianness/byte order," but the resulting productivity decrease was more impactful as a practical lesson.

While many situations today rely on API specifications like OpenAPI, this experience reminded me of the importance of organizing textual documentation and implementing error logs that help identify issues—all crucial aspects of maintainability.

- Write Documentation
- - Distinguish between automatically generated parts (like API specifications through OpenAPI) and human-written parts (like design philosophy and troubleshooting guides), and develop both in a balanced manner.
- Use Standardized Protocols
- - Use widely known protocols like HTTP or GraphQL.
  - If low-level communication is necessary, consider using standardized frameworks like gRPC.
  - Avoid using non-standardized protocols like TCP socket communication with fixed-length data exchanges.
- Implement Logs for Troubleshooting
- - At minimum, log error details when errors occur.
  - Include contextual information such as request IDs in logs to make tracing related logs easier.
