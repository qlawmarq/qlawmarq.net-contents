---
title: "My Approach to AI-Driven Development as of May 2025"
description: "Sharing practical insights on AI-driven development as of May 2025. Based on experience writing over 100,000 lines of code using tools like Cursor, Claude, and MCP, this article explains effective collaboration with AI, techniques for quality improvement, and future perspectives."
tags: ["AI", "Software Development"]
publishedAt: "2025-05-13T12:00:00.000Z"
---

## Introduction

We're already halfway through 2025, and today I'd like to share my current insights on AI-driven software development. Specifically, I'll discuss the tools I'm using and how I'm applying them.

AI has become indispensable to me. Just a few years ago, I thought Google search was the key to efficiency, but now I use AI far more frequently. In fact, this article was reviewed and translated by AI.

## Overview

I've been adopting AI-driven development methods since 2024, and as of May 2025, I've collaborated with AI to commit over 100,000 lines of code on GitHub. Along the way, I've experienced both successes and failures.

AI-driven development is still a relatively new concept with both supporters and critics.

From my experience, AI-driven development has dramatically improved my productivity. Tasks that used to take a week or several days can now be completed in half a day.

This technology is truly "destructive and fatal," and I'm convinced it will eventually minimize human involvement in many fields, though I believe it's still underestimated in public discourse where opinions remain divided.

That said, current LLM-centered AI technology isn't perfect. Subjectively, about 80% of AI-generated code can be used as-is, but the remaining 20% requires human modification or adjustment.

In essence, I don't consider the quality of current AI outputs to be consistently stable. This issue of inconsistent AI response accuracy will likely be resolved over time as AI technology advances.

With various workarounds, I've collaborated with AI to write over 100,000 lines of code. While current AI technology is certainly viable for production use, it requires some strategic implementation—that's my opinion.

## My AI-Driven Development Approach

While there are many ways to utilize AI, this article will focus on AI-driven development and sharing related know-how.

Here are the tools I'm currently using:

- [Cursor](https://www.cursor.com/)
- VSCode + [Roo Code](https://github.com/RooVetGit/Roo-Code)/[Cline](https://github.com/cline/cline)
- [Claude Desktop](https://claude.ai/)
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction)
- Various other tools (v0, [n8n](https://github.com/n8n-io/n8n), and personally developed tools)

Though this might seem like a lot, my current main tools are Cursor and Model Context Protocol (MCP).

I wanted to discuss automation using [n8n](https://github.com/n8n-io/n8n), a workflow automation tool, but I'll omit this as it's somewhat outside the context of AI-driven development.

### Cursor

For code editing, Cursor is my primary tool. Occasionally, I use VSCode with AI interaction plugins (Roo Code/Cline) as a secondary option. I used to use GitHub Copilot, but no longer do.

I've been using Cursor for about a year now, and it's become essential for me—I'm subscribed to the annual plan that costs nearly $200.

The major difference between Cursor and other editors is how easily it implements techniques like In-context Learning or Retrieval Augmented Generation (RAG), which improve AI response accuracy by incorporating project-specific information into prompts.

Normally, using RAG requires registering information as an index, but Cursor automatically registers your codebase information as an index (though manual registration is also possible).

Additionally, you can customize your project using the .cursor/rules configuration file to exclude information from indexing and define prompt methodologies.

Usage is very simple—just open the chat UI and give instructions about what you want to achieve. Using Agent Mode, it will not only implement code but also automatically run tests.

It's a formidable tool that can be used to some extent just by paying and installing, even without software development knowledge.

However, I feel it requires some technique to use effectively at present (more on this later).

### LLMs and MCP

Over 90% of the LLM models I use are Claude. Based on my experience, I believe Claude is currently the most proficient at writing code.

I occasionally use Gemini or GPT as well. Gemini is very convenient as you can use its API for free despite rate limits, so I incorporate it into tools for LLM-based code analysis. GPT excels as a partner for discussing philosophical and complex topics.

Another LLM-related tool I frequently use is MCP, which emerged in late 2024. MCP (Model Context Protocol) is a standard protocol for LLMs to integrate with external tools and services in a standardized way. This enables seamless access to various tools through LLMs.

For example, using the "markitdown" MCP server, you can convert PDF files to markdown format through LLM interaction. With the "filesystem" MCP server, you can access and edit local files through LLM interaction.

What's groundbreaking about MCP, as you may have realized, is that it enables almost any task to be performed through LLM interaction.

With GitHub's MCP server, you can review and merge code through LLMs like Claude Desktop without directly accessing GitHub.

With playwright's MCP server, you can even conduct E2E tests using web browsers through an LLM.

Indeed, whether it's information exploration via web search or YouTube information management, anything with API access can be done through an LLM.

Currently, the MCP servers I primarily use are:

- [markitdown](https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp) (converting PDF and Excel files to markdown)
- [filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) (accessing local environment files)
- [GitHub](https://github.com/github/github-mcp-server) (accessing GitHub)
- [playwright](https://github.com/microsoft/playwright-mcp) (executing E2E tests using Playwright)
- [context7](https://github.com/upstash/context7) (sharing the latest information about libraries with LLMs)
- [brave-search](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search) (web search)

By mastering LLMs and MCP servers, you can perform software development tasks like design and implementation based on information from MCP servers.

A current drawback of MCP is that initial setup requires knowledge of software development tools like Node.js, Python, and Docker.

For macOS users, you can easily set up Claude Desktop and MCP servers using [my macOS setup script](https://github.com/qlawmarq/dotfiles). If you have questions, you should be able to get help from AI.

## Practical Application

As mentioned, using these tools can make software development dozens of times more efficient.

However, using AI while ensuring quality output requires some technique, as current AI response accuracy varies and doesn't always deliver satisfactory results.

Here are some practices I focus on:

### Provide Detailed Instructions

First, ambiguous instructions increase the randomness in AI behavior, resulting in lower response accuracy.

Essentially, you want to share context and information necessary for development tasks with the AI.

For example, when implementing a function, it's good to provide detailed descriptions of the processing logic and return value information.

Also, instructions like "Don't start work with unclear points" can be included in your prompt.

With Cursor, you can define instructions in advance as prompts in .cursor/rules.

I include coding guidelines like these in my instructions:

> \- Always confirm unclear points before starting work.  
> \- Focus on future extensibility and avoid technical debt.  
> \- Adhere to best practices like DRY (Don't Repeat Yourself) principle and single responsibility principle.  
> \- Always prioritize maintainability and readability.  
> \- Don't blindly follow existing code structures; critically review the entire system.

You can find people sharing prompts on GitHub, so it might be worth checking:

- [PatrickJS/awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules)
- [kinopeee/cursorrules](https://github.com/kinopeee/cursorrules)

### Break Tasks into Smaller Pieces

Next, it's important to break tasks into smaller pieces before delegating to AI.

In my experience, complex tasks with larger workloads tend to result in less consistent quality.

This might seem to contradict the first point about detailed instructions, but context overload decreases AI response accuracy (and consumes massive tokens).

Also, with complex applications, you might get stuck in a loop where AI fails to resolve errors, but with minimal tasks, you can avoid errors and develop step by step.

For example, if you need to implement both UI and API for a TODO list, it's better to split the sessions between UI and API implementation. Breaking it down even further would be even better.

### Include Test Implementation and Execution in Tasks

Since quality isn't entirely stable, test execution is almost mandatory.

With Cursor, you can define test execution as a development rule in .cursor/rules.

Cursor also automatically detects and fixes lint and type errors, but if rules are complex and extensive, context overload can make AI behavior unstable, so moderation is key.

### Have AI Review the Output

Besides testing, having AI review code for maintainability and readability can reveal unexpected issues. Like anything, human reviews have limitations.

It's good to separate sessions and give instructions like "review with a critical perspective" to distinguish between worker and reviewer personas.

There are various methods: use MCP to access local files for review, use GitHub's MCP server to review on GitHub, or use Cursor to request reviews.

While I don't run reviews automatically with API and CI/CD because of cost, well-funded projects could integrate this into CI/CD for convenience.

### Regenerate if Quality is Questionable

As a last resort, if the output isn't what you expected, change the instructions (prompt) and generate again.

It's interesting to experiment with giving vague instructions and having AI regenerate multiple times—you'll often get completely different answers.

## Conclusion

In summary, while current AI technology still has challenges, there's no doubt that properly integrating it into the software development process dramatically improves productivity. Considering the benefits, it's worth exploring active adoption.

For those wanting to create something using AI, Cursor is a great starting point. You can even use [MCP](https://docs.cursor.com/context/model-context-protocol) through Cursor, so Cursor alone is sufficient.

For organizations with resources, considering Devin might be good, though I've only used it lightly.

Be mindful of costs—various tools can add up. I invest nearly $100 monthly in AI-related tools, so I hope AI tool subscription fees become more affordable.

There are other insights—like connecting UI generated in Figma to v0 or Cursor for easy webpage creation, how static typing languages are suitable for AI-driven development, or automating workflows using n8n and Gemini API—but I'll omit these for now.

Incidentally, this field has apparently become a hunting ground for criminals, so be cautious of tactics that try to install malicious libraries.[^1] Be especially wary of terms like "low-cost API keys."

## The Future of AI-Driven Development

Finally, I'll share my thoughts on the future of this field.

As of mid-2025, I believe we've already reached the point where downstream aspects of software development—"writing code based on given designs" or "devising designs from requirements"—can be delegated to AI.

Going forward, I expect AI to penetrate further into upstream aspects of software development.

Specifically, I think means to replace project manager and scrum master roles with AI will emerge.[^2] Actually, they already exist but haven't been widely adopted yet.

Automation is another key aspect. I believe a future where human involvement is minimized through AI intervention across various fields will be realized.

Some areas like research and pharmaceuticals are already achieving dramatic efficiency gains by having AI work while humans sleep.[^3] By integrating CI/CD with AI, a future where various development tasks are completed without human presence may not be far off.

In the military field, advanced country militaries are considering strategies to overwhelm competitors through AI-enabled decision-making at speeds impossible for humans.[^4] I believe management and decision-making aspects will also be streamlined by AI.

In any case, this technology is "destructive and fatal." While its impact isn't fully apparent yet, I believe it will have significant implications in the near future.

In the near future, I might need to consider retiring from software development myself.

---

[^1]: Ravie Lakshmanan (2025). Malicious npm Packages Infect 3,200+ Cursor Users With Backdoor, Steal Credentials. [https://thehackernews.com/2025/05/malicious-npm-packages-infect-3200.html](https://thehackernews.com/2025/05/malicious-npm-packages-infect-3200.html)

[^2]: Nieto-Rodriguez, A., & Vargas, R. V. (2023). How AI Will Transform Project Management. Harvard Business Review. [https://hbr.org/2023/02/how-ai-will-transform-project-management](https://hbr.org/2023/02/how-ai-will-transform-project-management)

[^3]: Schmidt, E. (2023). "This is how AI will transform how science gets done". MIT Technology Review. [https://www.technologyreview.com/2023/07/05/1075865/eric-schmidt-ai-will-transform-science/](https://www.technologyreview.com/2023/07/05/1075865/eric-schmidt-ai-will-transform-science/)

[^4]: Kanaka Rajan. "The Risks of Artificial Intelligence in Weapons Design." Harvard Medical School, (2024). [https://hms.harvard.edu/news/risks-artificial-intelligence-weapons-design](https://hms.harvard.edu/news/risks-artificial-intelligence-weapons-design)
