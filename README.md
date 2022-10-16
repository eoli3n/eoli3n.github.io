# Use jekyll on github with custom plugins
https://surdu.me/2020/02/04/jekyll-git-hook.html  
https://talk.jekyllrb.com/t/error-no-implicit-conversion-of-hash-into-integer/5890/11  

### Install
```bash
# Install ruby 2.7
$ rbenv install 2.7.5
# Run this in the project dir
$ rbenv local 2.7.5
$ ruby -v
ruby 2.7.5p203 (2021-11-24 revision f69aeb8314) [x86_64-linux]

# Add webrick
$ bundle add webrick

# Install gems
$ bundle install

# Clean gh-pages branch
$ git checkout gh-pages
$ rm -rf *
$ git add -A
$ git commit -m "Initialized gh-pages branch"
$ git push

# Disable jekyll build
$ touch .nojekyll
$ git add .nojekyll

# Create git pre-push hook to automate publication
$ cat < EOF > .git/hooks/pre-push
#!/bin/bash

# If any command fails in the bellow script, exit with error
set -e

# Set the name of the folder that will be created in the parent
# folder of your repo folder, and which will temporarily
# hold the generated content.
temp_folder="/tmp/_gh-pages-temp"

# Make sure our main code runs only if we push the main branch
if [ "$(git rev-parse --symbolic-full-name --abbrev-ref HEAD)" == "main" ]
then
    # Store the last commit message from main branch
    last_message=$(git show -s --format=%s main)

    # Each time gh-pages are pushed, as vendor directory is ignored and contains gems, it needs to be retriggered
    bundle install

    # Build our Jekyll site
    bundle exec jekyll build

    # Move the generated site in our temp folder
    rsync -avp _site/ ${temp_folder}

    # Checkout the gh-pages branch and clean it's contents
    git checkout gh-pages

    # Sync the site content from the temp folder and remove the temp folder
    rsync -avp --exclude '.*' ${temp_folder}/* .

    # trigger
    #rm -rf ${temp_folder}

    # Commit and push our generated site to GitHub
    git add -A
    git commit -m "Built '$last_message'"
    git push

    # Go back to the main branch
    git checkout main
else
    echo "Not main branch. Skipping build"
fi
EOF
```
