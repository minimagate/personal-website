+++
title = "Blog Post Part of Example Project"
description = "This blog post is assigned to the Example Project, so it will only appear on the project page, not in the homepage blog section."
date = 2026-08-01
authors = ["Michelangelo Gubinelli"]

[extra]
project = "example-project"
+++

This blog post is assigned to the "example-project" project. Notice that it won't appear in the homepage blog section, but it will appear on the project's dedicated page.

To assign a blog post to a project, add the `[extra]` section with `project = "slug-of-project"` to the frontmatter, where the slug must match the project's slug.
