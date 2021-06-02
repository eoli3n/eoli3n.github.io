### Resources

- https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll
- https://jekyllrb.com/docs/ruby-101/
- https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#publishing-sources-for-github-pages-sites
- https://pages.github.com/versions/
- https://stackoverflow.com/questions/19072070/how-to-find-where-gem-files-are-installed

### Install and create jekyll site

As **root**
```bash
$ xbps-install ruby-devel
$ gem update --system
$ gem install bundler
```

As **user**
```bash
$ mkdir project && cd project
$ echo << EOF > Gemfile
source "https://rubygems.org"

gem "jekyll"
EOF
$ bundle config set --local path 'vendor/bundle'
$ bundle install
$ bundle exec jekyll new -f .
$ echo ".bundle/" >> .gitignore
# remove gem jekyll in Gemfile and add github-pages one with the correct version
$ bundle update
# change 'baseurl: "/project"' in ``_config.yml``
$ git add .
$ git remote add origin git@github.com:eoli3n/blog-test.git
$ git push origin gh-pages
```


