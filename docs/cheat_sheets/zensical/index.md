---
tags:
    - ripgrep
    - HTML
---

# Zensical

Checking for links borked by messed up anchors using [ripgrep](https://github.com/burntsushi/ripgrep). I.e. `relative/link#anchor` instead of `relative/link/#anchor`.

```
rg -P '<a\s+href=[^>]*#|\([^)]*#[^)]*\)' docs/ -g '!docs/running_jobs/visualization/' 
```

HTML for fancy button

```<html><center><a href="link here" title="title here" target="_blank" class="md-button md-button--primary">Display String Here</a></center></html>
```