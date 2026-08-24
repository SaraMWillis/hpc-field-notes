# Zensical

Checking for links borked by messed up anchors. I.e. `relative/link#anchor` instead of `relative/link/#anchor`.

```
rg -P '<a\s+href=[^>]*#|\([^)]*#[^)]*\)' docs/ -g '!docs/running_jobs/visualization/' 
```

HTML for fancy button

```<html><center><a href="https://workadventure.tra220030.projects.jetstream-cloud.org/_/global/devinbayly.github.io/office-hours/office.tmj" title="Click here to join our office hours workspace" target="_blank" class="md-button md-button--primary">Join Us in Office Hours</a></center></html>
```