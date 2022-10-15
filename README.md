# Use jekyll on github with custom plugins
https://surdu.me/2020/02/04/jekyll-git-hook.html

### Install
```bash
$ bundler install
$ git checkout gh-pages
$ rm -rf *
$ git add -A
$ git commit -m "Initialized gh-pages branch"
$ git push
$ cat < EOF > .git/hooks/pre-push
#!/bin/bash

# If any command fails in the bellow script, exit with error
set -e

# Set the name of the folder that will be created in the parent
# folder of your repo folder, and which will temporarily
# hold the generated content.
temp_folder="_gh-pages-temp"

# Make sure our main code runs only if we push the main branch
if [ "$(git rev-parse --symbolic-full-name --abbrev-ref HEAD)" == "main" ]
then
    # Store the last commit message from main branch
    last_message=$(git show -s --format=%s main)

    # Build our Jekyll site
    bundle exec jekyll build

    # Move the generated site in our temp folder
    mv _site ../${temp_folder}

    # Checkout the gh-pages branch and clean it's contents
    git checkout gh-pages
    rm -rf *

    # Copy the site content from the temp folder and remove the temp folder
    cp -r ../${temp_folder}/* .
    rm -rf ../${temp_folder}

    # Commit and push our generated site to GitHub
    git add -A
    git commit -m "Built \`$last_message\`"
    git push

    # Go back to the main branch
    git checkout main
else
    echo "Not main branch. Skipping build"
fi
EOF
```
