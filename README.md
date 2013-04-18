git-gohelper
============

git-gohelper is a global git alias that adds a formatting/compilation pre-commit hook and a local git-gofmt alias.

## Install / Upgrade

Run this:

```bash
git config --global alias.gohelper '![ -d ".git" ] && echo '"'"'#!/bin/bash

errors=$((go build -o _tmpbuild) 2>&1)

if [[ $? -ne 0 ]]; then
        echo >&2 "Go files have syntax errors:"
        echo >&2 "$errors"
        exit 1
fi

rm _tmpbuild

gofiles=$(git diff --cached --name-only --diff-filter=ACM | grep ".go$")
[ -z "$gofiles" ] && exit 0

unformatted=$((gofmt -l $gofiles) 2> /dev/null)

if [ -n "$unformatted" ]
then
        echo >&2 "Go files must be formatted with gofmt:"
        echo -n "  "
        for fn in $unformatted; do
                echo -n >&2 "$fn "
        done
        echo
        echo "Run \`git gofmt\`"
        exit 1
fi'"'"' > .git/hooks/pre-commit ; chmod ugo+x .git/hooks/pre-commit ; git config alias.gofmt '"'"'!echo $(git diff --cached --name-only --diff-filter=ACM | grep ".go$") | xargs gofmt -w -l | xargs git add'"'"
```

## How To

### Setup Repo
```bash
git init # If you haven't already
git gohelper # Once per repo
```

### After runing `git gohelper`

```bash
git add .
# pre-commit hook will block commit if:
#   (1) Code won't compile - You have to fix code
#   (2) Code wasn't gofmt'd
git commit -m "Some message"
# If code compiles run `git gofmt` to satisfy (2)
# This will `gofmt` and `git add` only files being committed
git gofmt
# When (1) and (2) are satisfied commit will go through
git commit -m "Some message"
```
