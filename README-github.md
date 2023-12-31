# Use jekyll on github with custom plugins
https://surdu.me/2020/02/04/jekyll-git-hook.html  
https://talk.jekyllrb.com/t/error-no-implicit-conversion-of-hash-into-integer/5890/11  

### Install
```bash
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

# In github repository settings / Pages / Branch
# Set "gh-pages" branch "/ (root)"

# Create git pre-push hook to automate publication
$ cat << "EOF" > .git/hooks/pre-push
#!/bin/bash

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
    #bundle exec jekyll build
    # see https://github.com/ggreer/jekyll-gallery-generator/issues/47#issuecomment-1872988970
    RUBYOPT="-r./file-exists" bundle exec jekyll build

    # Move the generated site in our temp folder
    rsync -avp --delete _site/ ${temp_folder}

    # Checkout the gh-pages branch and clean it's contents
    git checkout gh-pages

    # Sync the site content from the temp folder and remove the temp folder
    rsync -avp --delete --exclude '/.*' ${temp_folder}/* .

    # Disable jekyll build on github actions
    touch .nojekyll

    rm -rf ${temp_folder}

    # Commit and push our generated site to GitHub
    git add -A

    # If something to commit related to the site, commit then push
    if git commit -m "Built '$last_message'"
    then
        git push
    fi

    # Go back to the main branch
    git checkout main
else
    echo "Not main branch. Skipping build"
fi
EOF
```

