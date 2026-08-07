# My Hugo Blog

This is a Hugo blog using the Technical Ledger design. It has a light default mode and a dark mode toggle. You do **not** need to know Go to use it. Hugo is written in Go, but you work mainly with:

- Markdown files in `content/` for writing posts
- `hugo.toml` for site settings
- HTML templates in `layouts/`
- CSS in `assets/css/`

The original imported article remains at the project root as `blogpost.md`. The Hugo-ready copy is `content/posts/jeanclawd-building-a-local-ai-assistant.md`.

## Install Hugo

On macOS with Homebrew:

```sh
brew install hugo
```

Check it:

```sh
hugo version
```

## Run the site locally

From this folder:

```sh
hugo server -D
```

Open <http://localhost:1313/>. The `-D` option also shows draft posts.

## Write a post

Create a new post:

```sh
hugo new posts/my-new-post.md
```

Open the new file, change `draft: true` to `draft: false` when it is ready, and write in Markdown. Hugo automatically reloads the browser while the server is running.

## Build the site

```sh
hugo
```

The finished static files will be placed in `public/`. They can be hosted on GitHub Pages, Netlify, Cloudflare Pages, or any ordinary web server.
