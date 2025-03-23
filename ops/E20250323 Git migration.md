## E20250323 Git migration.md

- Bitbucket에서 Github으로 마이그레이션 필요
- ./script.sh bitbucket github

```
#!/bin/bash

# 파라미터 확인
if [[ $# -ne 2 ]]; then
  echo "Usage: $0 <OLD_REMOTE> <NEW_REMOTE>"
  exit 1
fi

OLD_REMOTE="$1" 
NEW_REMOTE="$2" 

echo "Migrating all branches and tags from '$OLD_REMOTE' to '$NEW_REMOTE'..."

echo "Fetching all branches from $OLD_REMOTE..."
git fetch "$OLD_REMOTE"

branches=$(git branch -r | grep "$OLD_REMOTE" | grep -v 'HEAD' | sed "s|${OLD_REMOTE}/||")
echo "Found the following branches:"
echo "$branches"

for branch in $branches; do
    echo "Checking out branch: $branch"
    git checkout -b "$branch" "$OLD_REMOTE/$branch" 2>/dev/null || git checkout "$branch"
done

echo "Pushing all branches to $NEW_REMOTE..."
git push --all "$NEW_REMOTE"

echo "Pushing all tags to $NEW_REMOTE..."
git push --tags "$NEW_REMOTE"

echo "All branches and tags have been successfully pushed to $NEW_REMOTE!"
```
