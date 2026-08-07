---
title: "Designing this blog with Google Stitch and Codex"
description: "Using Google Stitch and Codex together to move from a design brief to a finished static blog."
date: 2026-08-07
tags: [google-stitch, codex, mcp, hugo, vibe-coding]
draft: false
---

I wanted a small personal blog: a simple static engine, a handful of pages, and a design that felt more deliberate than a default theme. The brief was intentionally short. I asked for a home page, a blog listing, an about page, and individual posts, with a visual language that would make the site feel like mine. I wanted a particular look and feel, which is coined as Synthetic Biome design language.

## Starting with a brief

I gave the brief as a prompt to Codex (you can also directly do in the browser ) and connected it to Google Stitch through the Model Context Protocol (MCP). [Stitch](https://stitch.withgoogle.com/) is designed to turn natural-language product ideas into high-fidelity interfaces, while MCP gives a coding agent a way to retrieve the resulting design context rather than treating the design as a screenshot to imitate (which is also possible, but why should I).

That changed the first step. Instead of beginning with a Hugo theme and gradually decorating it, I started with the experience: the page structure, the typography, the layout, the colour system, and the small visual details that should carry across the site.

## Negotiating the design

Codex took the brief to Stitch, brought the returned designs back into the conversation, and helped me assess what worked. We went through a few iterations—adjusting the balance between the article content and the surrounding visual system, refining the navigation, and making sure the design still suited a lightweight blog rather than an application dashboard.

The useful part was the loop. Stitch could explore the visual direction quickly, while Codex could translate feedback into concrete implementation decisions. The design was not a one-shot image export; it became a shared reference that we could question and refine.

Stitch can export the designs as Figma, Lovable, Bolt or a .zip archive with HTML and CSS coded. I love the flexibility, and was satisfied with the quality of the outcome in 

## From design to Hugo

Once the direction was right, I asked Codex to implement it as designed. The site became a Hugo project with Markdown content, reusable layouts, a small asset set, and a generated `public/` directory ready for GitHub Pages. The implementation stayed intentionally simple: static files, a few templates, responsive CSS, and no runtime application layer.

That simplicity was part of the brief. The point was not to reproduce every possible design-system abstraction. It was to carry the important decisions from Stitch into a fast, maintainable site without losing the character of the design along the way.

## The result

The final workflow felt like a conversation between three roles: I described the outcome, Stitch explored and refined the visual language, and Codex connected that language to real files and a working build. After a few iterations, the blog was implemented and ready to publish.

For me, the interesting shift is that design and implementation no longer had to be separate hand-offs. With Stitch and Codex connected through MCP, the design could be negotiated in context and then carried directly into the static site.

The workflow follows the broader design-to-code pattern described in Google’s [Stitch MCP codelab](https://codelabs.developers.google.com/design-to-code-with-antigravity-stitch), where a natural-language design is created in Stitch, design context is made available through MCP, and an agent implements and refines the result. Google also describes Stitch as a canvas for creating and iterating on high-fidelity interfaces from natural language in its [Stitch overview](https://developers.googleblog.com/en/stitch-a-new-way-to-design-uis/).
