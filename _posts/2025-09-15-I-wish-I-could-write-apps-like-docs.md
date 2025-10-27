---
layout: post
title: "I wish I could write apps the way I write docs"
summary: "I wrote out this fantasy and got sad at the implications"
---

**UPDATE (Oct 2025)**: Since publishing this post, I came across Alan Kay and team's government-funded project ([docs](https://tinlizzie.org/VPRIPapers/tr2012001_steps.pdf)) to reinvent software development. The final project report discusses their proofs of concept for an integrated suite of applications with fewer and more expressive lines of code. They describe the suite as follows:

> It has "universal documents" that can model all document forms [...] and an independent repertoire of ways to find, fetch, send, save, broadcast, etc., the documents. This subsumes the normal scattered "productivity suite" intractable stovepipes often provided instead of a truly integrated system.

The final report from this project was 13 years ago, so I wouldn't hold my breath for a viable product today. This team hasn't released STEPS yet for the open-source development community. I can't pretend to understand the report well enough to claim that readers could build STEPS in a GitHub repo right away. Still, and not to be a Pollyanna, it's energizing to learn that some of the smartest folks on the planet considered this question and got funding for it!

## The Fantasy

Imagine that the browser on your device was an app for natively browsing, creating _and_ publishing pages of dynamic software. There'd be no need for a separate IDE or editor to author interactive programs to view them in the browser.

The inspiration for this comes from using prior art like:
- Literate programming software, like [Rmarkdown](https://rmarkdown.rstudio.com/), [Hex](https://hex.tech/) and [Marimo](https://marimo.io/)
- Google Work suite and [Apps Script](https://developers.google.com/apps-script)
- [Notion](https://www.notion.com/)
- The web
- [Acme](https://en.wikipedia.org/wiki/Acme_(text_editor))

In addition, reading about important projects in computing history fires up the imagination of what could have been:
- [Project Xanadu](https://en.wikipedia.org/wiki/Project_Xanadu)
- [Plan 9 from Bell Labs](https://en.wikipedia.org/wiki/Plan_9_from_Bell_Labs)
- [Smalltalk](https://en.wikipedia.org/wiki/Smalltalk)

This would be the ideal everything app, and the creation flow would be something like below. Just find-and-replace 'World' for a Google Doc or Notion space for ease of imagining.
- Open the app, call it Worlds
- Create a new World project from collections of Templates or a blank slate
- A new World is a page of addressable blocks that can hold data: code, images, videos, text, audio, etc. Worlds allows source code mode, exposing project files in a directory with links to content and fenced blocks for code in text files
- Write code or add content to define the behavior for each block. Blocks link to (blocks in other) Worlds, where the latter are accessible
- Persisting a World to storage creates a project directory of files that preserve state. A sandboxed VM builds a World from scratch each time the directory is opened.  All Worlds tooling, dependency managers, execution engine, etc., are all one integrated standardized stack
- To share on the Web/a network, publish the World to a server from inside Worlds. There can be appropriate access and edit controls defined for the World so not anybody with the link can simply pull it

A global registry indexes all publicly accessible Worlds for ease of semantic search. As more new users joined Worlds, they could easily move from browsers/consumers of content to authors. A user could click on a World they like to inspect the layout, code and the output of specific blocks. One easy-to-learn universal programming language, Hydrogen, is used to author World projects and extend functionality in Worlds the app. They could fork that World, make changes and then upload a new copy. When working with other users, pushing the World to a server makes it easy for other users with permissions to collaboratively edit and version. Since Worlds are blocks, experienced users could write reusable modular Components that extend functionalities. Components can be remotely referenced via URLs and `import`s in new blocks. These can also become parts of the native Hydrogen runtime via popularity in a secure universal registry. As the language gracefully ages, many frameworks and superset languages arise.

To make Worlds adaptable across all devices, all hardware speak a universal protocol to provide native and secure access to Worlds so users can easily leverage them in their applications. Worlds that need online connectivity for functionality (for remote storage or processing power) can adaptively use local resources. All the boring standardization stuff is maintained by an open and non-profit consortium.

All of the tooling is free and open source.

## Implications

Now it's time to think realistically. Assuming this somehow took off, either in a world in which Worlds was the web or effectively replaced it after personal computers got powerful:

- Google. AdSense. YouTube. Facebook. TikTok. More people will always consume content than create it.
- You can develop powerful apps on and for any device! People who want to create things for themselves can write group chat apps, servers to stream their content to multiple speakers and screens for themselves and their friends, mini games rife with inside jokes. This forces companies to seriously level up their software to make the case for paying for licenses and subscriptions
- Purpose-built dialects of Hydrogen take over domains in Worlds. From Hydrogen comes Helium and countless others. Corporate power influences the employability and popularity of dialects, though even companies are ultimately beholden to the bureaucratic Hydrogen standard
- Wikipedia fucking rules with interactive software.
- Server hosting becomes an even bigger business and even more susceptible to geopolitics
- Assuming hardware vendors play ball, hardware ages with increasing longevity and that makes it harder to enable new features for all. The most popular denominator hardware spec dictates the features 99% of Worlds projects actually have
- Components get out of hand and there are Components to get the right Components and their versions to work
- My mother and millions of other people watch AI baby videos on the Facebook World platform