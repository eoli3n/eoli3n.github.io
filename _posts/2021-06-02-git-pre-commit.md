---
title: Git pre commit hooks
layout: post
icon: fa-code-branch
---

[Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) are shell scripts integrated to your git directory. They are triggered before: ``pre-`` or after: ``post-`` some actions, ``commit``, ``push``, ``checkout`` and run locally. The most useful function is ``pre-commit`` which can check your code syntax, lint to improve the code quality, or trigger any local test.

Hooks are mostly always the same, users share them on github and [pre-commit](https://pre-commit.com/hooks.html) tool automates hooks deployment.

```bash
$ pip install pre-commit
```

### Deploy a hook

Create a test directory and git it.
```bash
$ mkdir git-commit-test && cd git-commit-test

$ git init
Initialized empty Git repository in /home/user/dev/git-commit-test/.git/
```

Choose hooks projects you want to use in [pre-commit hooks list](https://pre-commit.com/hooks.html).  
For this exemple, we will use yaml and shell checkers.
- https://github.com/pre-commit/pre-commit-hooks : check-yaml and trailing-whitespace
- https://github.com/shellcheck-py/shellcheck-py : shellcheck

Pre-commit configuration file is ``.pre-commit-config.yaml``, create it at the project root.
```yaml
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.0.1
  hooks:
    - id: check-yaml
    - id: trailing-whitespace
- repo: https://github.com/shellcheck-py/shellcheck-py
  rev: v0.7.2.1
  hooks:
    - id: shellcheck
```

To init hook deployment, use ``install`` subcommand.

```bash
$ pre-commit install
pre-commit installed at .git/hooks/pre-commit
```

### Test it

Let's add a shell script to the project and a yaml file with some errors.

```bash
$ cat << EOF > test.sh
#!/bin/bash
var=1
echo $var
EOF

$ cat << EOF > test.yml
- key1
  key2:
EOF
```

Add those files to the index, ``pre-commit`` defaulty ignore files outside the index.

```bash
$ git add .

$ git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   .pre-commit-config.yaml
	new file:   test.sh
	new file:   test.yml
```

And just commit to fire your hooks !

```bash
$ git commit -m "added a great shell script and his yaml file"
Check Yaml...............................................................Failed
- hook id: check-yaml
- exit code: 1

mapping values are not allowed in this context
  in "test.yml", line 2, column 6

Trim Trailing Whitespace.................................................Failed
- hook id: trailing-whitespace
- exit code: 1
- files were modified by this hook

Fixing test.sh

shellcheck...............................................................Failed
- hook id: shellcheck
- exit code: 1

In test.sh line 2:
var=1
^-^ SC2034: var appears unused. Verify use (or export if used externally).

For more information:
  https://www.shellcheck.net/wiki/SC2034 -- var appears unused. Verify use (o...
```

As pre-commit hooks failed, the commit was not created.
```bash
$ git log
fatal: your current branch 'main' does not have any commits yet
```

Trailing whitespace hooks automatically fixed his errors.
If you fix your shell script and yaml file, your commit will be ok.


```bash
$ sed -i 's/$var/\"$var\"/' test.sh

$ sed -i 's/key1/key1:/' test.yml

$ git add test.*

$ git commit -m "first sane commit"
Check Yaml...............................................................Passed
Trim Trailing Whitespace.................................................Passed
shellcheck...............................................................Passed
[main (root-commit) c689536] first sane commit
 3 files changed, 15 insertions(+)
 create mode 100644 .pre-commit-config.yaml
 create mode 100644 test.sh
 create mode 100644 test.yml
```

### Custom Ansible Hook

I wanted to be able to check ansible syntax before.

```yaml
- repo: local
  hooks:
    - id: ansible-syntax-check
      name: Ansible syntax check
      entry: ansible-playbook --syntax-check
      files: playbook.yml
      types: [file]
      language: system
```
Create the playbook.
```bash
cat << EOF > playbook.yml
- hosts: test
  tasks:
    - name: copy test.sh
      copyy:
        src: ./test.sh
        dest: dir
EOF
```
Then try to commit it.
```bash
$ git add .
$ git commit -m "added an insane playbook"
[user@osz git-commit-test]$ git commit -m "added an insane playbook"
Check Yaml...............................................................Passed
Trim Trailing Whitespace.................................................Passed
shellcheck...........................................(no files to check)Skipped
Ansible syntax check.....................................................Failed
- hook id: ansible-syntax-check
- exit code: 4

[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that
the implicit localhost does not match 'all'
ERROR! couldn't resolve module/action 'copyy'. This often indicates a misspelling, missing collection, or incorrect module path.

The error appears to be in '/home/user/dev/git-commit-test/playbook.yml': line 3, column 7, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

  tasks:
    - name: copy test.sh
      ^ here
```

As no shell script was in the index, ``shellcheck`` was skipped, but pre-commit detected our mispelled ``copy`` ansible module.
