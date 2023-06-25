https://docs.gitlab.com/ee/user/project/pages/getting_started/pages_from_scratch.html
https://docs.gitlab.com/ee/user/project/pages/


- create eoli3n.gitlab.io project
- add a new push url
```
$ git remote set-url --add --push origin git@gitlab.com:eoli3n/eoli3n.gitlab.io.git
```
- create gitlab-ci.yml
- git push


# TODO
- [ ] don't push gh-pages on gitlab : edit the pre-push script
- [ ] cache vendors
- [ ] add test build
