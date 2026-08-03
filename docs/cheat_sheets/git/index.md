# Git


### Whoops. I forgot a gitignore. 

My bad.

```bash title="Example of removing site directory from commit. I didn't just literally do this..."
echo "site/" >> .gitignore
git rm -r --cached site/

## The `--cached` flag removes the files from Git's index without deleting
## them from your working directory. Since `site/` is now in `.gitignore`,
## Git won't pick them up again.

git add .gitignore
git diff --cached --stat
git commit --amend --no-edit
```