# klusai-pages-insights

**insights.klusai.com** — field notes from building kOS.

Jekyll on GitHub Pages, using the shared brand theme
[`klusai/klusai-pages-brand`](https://github.com/klusai/klusai-pages-brand),
the same one research.klusai.com and academy.klusai.com use. Presentation is not
maintained here: change it in the brand repo and every site picks it up.

Writing a post? Read [AGENTS.md](AGENTS.md) first. The backlog is the
**KlusAI Insights** project in Linear.

```
_posts/       posts, YYYY-MM-DD-slug.md
blog.md       the index, at /blog/ (the theme links to that exact path)
index.md      home; the theme appends the four latest posts
about.md      what this site is and what it will not print
_config.yml   theme, accent, nav
```

Local preview:

```bash
bundle install
bundle exec jekyll serve
```
