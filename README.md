# Swifting.io blog content

This repository contains the content for the Swifting.io blog, published at [swiftingio.github.io](https://swiftingio.github.io).

The site is built with [Jekyll](https://jekyllrb.com/).

## Run locally

Install the Ruby dependencies:

```sh
bundle install
```

Start the local Jekyll server:

```sh
bundle exec jekyll serve
```

Open the address printed by Jekyll in your browser (normally `http://localhost:4000`).

## Add a post

Posts are stored in [`_posts/`](_posts/). Create a Markdown file named using this format:

```text
YYYY-MM-DD-title.md
```

Every post begins with YAML front matter. Use an existing author identifier from [`_authors/`](_authors/) and include a short excerpt:

```yaml
---
layout: post
author: bartek
title: Your post title
excerpt: A short summary shown in post listings.
---
```

Write the post body below the front matter in Markdown. Preview the site locally before publishing to check the content and formatting.

## Repository structure

- [`_posts/`](_posts/) — blog posts.
- [`_authors/`](_authors/) — author metadata used by posts.
- [`_config.yml`](_config.yml) — Jekyll site configuration.
- [`Gemfile`](Gemfile) — Ruby and Jekyll dependencies.
