# Conventinal Commit Messages 
See how a minor change to your commit message style can make a difference. [Examples](#examples)

<img src="https://img.icons8.com/dusk/1600/commit-git.png" width="200" height="200" />

## Commit Formats

### Default
<pre>
<b><a href="#types">&lt;type&gt;</a></b></font>(<b><a href="#scopes">&lt;optional scope&gt;</a></b>): <b><a href="#subject">&lt;subject&gt;</a></b>
<sub>empty separator line</sub>
<b><a href="#body">&lt;optional body&gt;</a></b>
<sub>empty separator line</sub>
<b><a href="#footer">&lt;optional footer&gt;</a></b>
</pre>

### Merge
<pre>
Merge branch '<b>&lt;branch name&gt;</b>'
</pre>
<sup>Follows default git merge message</sup>

### Revert
<pre>
Revert "<b>&lt;commit headline&gt;</b>"
<sub>empty separator line</sub>
This reverts commit <b>&lt;commit hash&gt;</b>.
<b>&lt;optinal reason&gt;</b>
</pre>
<sup>Follows default git revert message</sup>

### Types
* `feat` A code change that adds a new feature
* `fix` A code change that adds a bug fix
* `refactor` A code change that neither adds a new feature nor fixes a bug
* `style` A code change that do not affect the meaning (white-space, formatting, missing semi-colons, etc)
* `test` Changes that add missing tests or correcting existing tests
* `docs` Changes that affect documentation only
* `build` Changes that affect the build system or external dependencies
* `ops` Changes that affect operational components like infrastructure, backup or recovery


### Scopes
The `scope` provides additional contextual information.
* Is an **optional** part of the format
* Allowed Scopes depends on the specific project
* Don't use issue identifiers as scopes

### Subject
The `subject` contains a succinct description of the change.
* Is a **mandatory** part of the format
* Use the imperative, present tense: "change" not "changed" nor "changes"
* Don't capitalize the first letter
* No dot (.) at the end

### Body
The `body` should include the motivation for the change and contrast this with previous behavior.
* Is an **optional** part of the format
* Use the imperative, present tense: "change" not "changed" nor "changes"
* This is the place to mention issue identifiers and their relations

### Footer
The `footer` should contain any information about **Breaking Changes** and is also the place to **reference Issues** that this commit refers to.
* Is an **optional** part of the format
* **optionally** reference an issue by its id.
* **Breaking Changes** should start with the word `BREAKING CHANGE:` folowed by space or two newlines. The rest of the commit message is then used for this.


### Examples
* ```
  feat(shopping cart): add the amazing button
  ```
* ```
  feat: remove ticket list endpoint
  
  refers to JIRA-1337
  BREAKING CHANGE: ticket enpoints no longer supports list all entites.
  ```
* ```
  fix: add missing parameter to service call
  
  The error occurred because of <reasons>.
  ```
* ```
  build: release version 1.0.0
  ```
* ```
  build: update dependencies
  ```
* ```
  refactor: implement calculation method as recursion
  ```
* ```
  style: remove empty line
  ```
* ```
  revert: refactor: implement calculation method as recursion
  
  This reverts commit 221d3ec6ffeead67cee8c730c4a15cf8dc84897a.
  ```
  
  
## Git Hook Scripts to ensure commit message header format

### commit-msg Hook (local)
* create following file `.git-hooks/commit-msg` and make it executable
```shell
#!/usr/bin/env sh

# commit-msg hook that will ensure commit messge format

commit_msg_type_regex='feat|fix|refactor|style|test|docs|build'
commit_msg_scope_regex='.{1,20}'
commit_msg_subject_regex='.{1,100}'
commit_msg_regex="^(${commit_msg_type_regex})(\(${commit_msg_scope_regex}\))?: (${commit_msg_subject_regex})\$"
merge_msg_regex="^Merge branch '.+'\$"
revert_msg_regex="^Revert \".+\"\$"

commit_msg_header=$(head -1 $1)
if ! [[ "$commit_msg_header" =~ (${commit_msg_regex})|(${merge_msg_regex})|(${revert_msg_regex}) ]]; then
  echo "ERROR: Invalid commit message format" >&2
  echo "\n$commit_msg_header" >&2
  exit 1
fi
```
* set commit hook directory `git config core.hooksPath ".git-hooks"`

### pre-receive Hook (server side)
* create following file `.git/hooks/pre-receive`
```shell
#!/usr/bin/env sh

# Pre-receive hook that will block commits with messges that do not follow regex rule

commit_msg_type_regex='feat|fix|refactor|style|test|docs|build'
commit_msg_scope_regex='.{1,20}'
commit_msg_subject_regex='.{1,100}'
commit_msg_regex="^(${commit_msg_type_regex})(\(${commit_msg_scope_regex}\))?: (${commit_msg_subject_regex})\$"
merge_msg_regex="^Merge branch '.+'\$"
revert_msg_regex="^Revert \".+\"\$"

zero_commit="0000000000000000000000000000000000000000"

# Do not traverse over commits that are already in the repository
excludeExisting="--not --all"

error=""
while read oldrev newrev refname; do
  # branch or tag get deleted
  if [ "$newrev" = "$zero_commit" ]; then
    continue
  fi

  # Check for new branch or tag
  if [ "$oldrev" = "$zero_commit" ]; then
    rev_span=`git rev-list $newrev $excludeExisting`
  else
    rev_span=`git rev-list $oldrev..$newrev $excludeExisting`
  fi

  for commit in $rev_span; do
    commit_msg_header=$(git show -s --format=%s $commit)
    if ! [[ "$commit_msg_header" =~ (${commit_msg_regex})|(${merge_msg_regex})|(${revert_msg_regex}) ]]; then
      echo "$commit" >&2
      echo "ERROR: Invalid commit message format" >&2
      echo "$commit_msg_header" >&2
      error="true"
    fi
  done
done

if [ -n "$error" ]; then
  exit 1
fi

```

-----
## References
* https://www.conventionalcommits.org/
* https://github.com/angular/angular/blob/master/CONTRIBUTING.md
* http://karma-runner.github.io/1.0/dev/git-commit-msg.html
<br>

* https://github.com/github/platform-samples/tree/master/pre-receive-hooks  
* https://github.community/t5/GitHub-Enterprise-Best-Practices/Using-pre-receive-hooks-in-GitHub-Enterprise/ba-p/13863

