# Zensical

Checking for links borked by messed up anchors. I.e. `relative/link#anchor` instead of `relative/link/#anchor`.

```
rg -P '<a\s+href=[^>]*#|\([^)]*#[^)]*\)' docs/ -g '!docs/running_jobs/visualization/' 
```