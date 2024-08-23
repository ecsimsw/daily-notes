## What I want to do.

Assume that we have 3 separated repository each names are `A`, `B`, `C`. 
And we want to make mono repo, let’s call that `root`

What I wanted to do is merge `A`, `B`, `C` to root without losing git logs while removing git configuration things (e.g, .git file). And I didn’t care about branches of each repositories. We’re planning to use only default branch.

Belows are steps how I made this.

## Steps

1. Create `root` repository

2. Clone `A`, `B`, `C` on local

3. Inside `A`, make directory as how that repository are migrated. What you want to make is root/{FOLDER_NAME}, then make {FOLDER_NAME}.

```
e.g.) 
git clone A
cd A 
mkdir A
```

4. Move all the files of `A` to folder you just made.   
`ls -a1 | grep -v ^CURRENT_DIR_NAME | xargs -I{} git mv {} FOLDER_NAME`

5. Git commit 
`git commit -m "COMMIT_MESSAGE"`

6. In root, add `A` repo folder as remote 
`git remote add A_REMOTE_NAME A_DIRECTORY_PATH`

7. Fetch it
`git fetch A_REMOTE_NAME`

8. Merge A on root
`git merge A_REMOTE_NAME/BRANCH_NAME --allow-unrelated-histories`

9. Repeat these steps for `A`, `B`, `C`

## Ref, 

https://medium.com/@filipenevola/how-to-migrate-to-mono-repository-without-losing-any-git-history-7a4d80aa7de2
