---
title: "Designing this blog with Google Stitch and Codex"
description: "Using Google Stitch and Codex together to move from a design brief to a finished static blog."
date: 2026-08-07
tags: [google-stitch, codex, mcp, hugo, vibe-coding]
draft: false
---

I usually design my own stuff, but being the harsh critic I am it takes ages, then I get tired. I've been wanting to try [Google Stitch](https://stitch.withgoogle.com/) for a while, and I wanted to get my blog up and running so I opted for Hugo. Instead of using a default template, I provided Codex a brief for a particular look and feel, which is coined as Synthetic Biome design language.

Stitch uses Gemini Flash powering it's agent, which provides a conversational bot and a work bench with multiple pages that are connected to the history of interactions. Stitch uses your feedback to iterate on the design, typography, and even generate light / dark mode and responsive layout options. It keeps a history of all your projects should you want to revisit. To be honest, I found the process organic and intuitive.

## The brief

I gave the brief as a prompt to Codex (you can also directly do in the browser ) and connected it to Google Stitch through the Stitch MCP, The MCP gives a coding agent a way to retrieve the resulting design context rather than treating the design as a screenshot to imitate (which is also possible, but why should I).

Instead of beginning with a Hugo theme and gradually tweaking it, I started with the experience: the exact page structure, the typography, the layout, the colour system, and the small visual details that should carry across the blog.

## Negotiating the design

Codex took the brief to Stitch, brought the returned designs back into the conversation, and helped me assess what worked. We went through a few iterations—adjusting the balance between the article content and the surrounding visual system, refining the navigation, and making sure the design still suited a lightweight blog rather than an application dashboard.

The useful part was the loop. Stitch could explore the visual direction quickly, while Codex could translate feedback into concrete implementation decisions. The design was not a one-shot export; it became a back and forth between two agents.

Stitch can also export the designs as Figma, Lovable, Bolt or a .zip archive with HTML and CSS coded. I love the flexibility, and was satisfied with the quality of the output.

## Implementation

Once the direction was right, I asked Codex to implement it as is. The designed template became a Hugo project with Markdown content, reusable layouts, a small asset set, and a generated `public/` directory ready for GitHub Pages. Working with Hugo is a breeze compared to what I normally do, single page applications using Angular or React.

Codex was able to do the small changes moving forward, so I didn't have a need for Stitch all the time.

## The verdict

For me, the interesting shift is that design and implementation no longer had to be separate hand-offs. With Stitch and Codex connected through MCP, the design could be negotiated in context and then carried directly into the static site.

I also loved the granularity offered by Stitch. It exposes font selection, base colors, theme settings with primary/tertiary colours, as well as a markdown file which allows the user to fine tune CSS variables and the initial design brief provided to the agent. Changing and applying these settings would retrigger the generation, applying the new preferences.

I've experimented with things such as giving the design language as basis (e.g. Google Material), and the generated screens would follow the instruction very well.

The workflow follows the broader design-to-code pattern described in Google’s [Stitch MCP codelab](https://codelabs.developers.google.com/design-to-code-with-antigravity-stitch).
