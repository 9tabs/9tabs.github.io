---
layout: post
title:  "Localize Your Pip Dependencies"
date:   2018-01-21 00:11:12 -0700
categories: [snippets]
---

I recently introduced a pip package with pypi.org dependencies into my team's continuous
integration pipeline. Due to the nature of our industry, predictability, repeatability, and
traceability are high priority concerns for our customers. For this reason, it's critical that our
build and test infrastructure does not rely on any externally-hosted dependencies.

Thankfully, localizing your pip dependencies is *incredibly* easy thanks to the
[pip2pi](https://github.com/wolever/pip2pi) package. You can create a local PyPI-compatible
repository from any package (and its dependencies) in one line:

```bash
pip2tgz "/var/www/packages" mypackage
```

Users can install these packages with the following command:

```bash
pip install --index-url="file:///var/www/packages" mypackage
```

Reducing the number of potential points of failure was my intention when I stumbled on pip2pi,
but using it also made a significant difference in the initialization time for our pipelines that
depend on pip packages.

I hope this article was useful. You can follow the discussion on reddit
[here]().
