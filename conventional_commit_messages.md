# Semantic Commit Messages
See how a minor change to your commit message style can make you a better programmer.

## Format

**<[type](#types)>(<[scope](#scopes)>): <[subject](#subject)>**<br>
<sub>`empty separator line`</sub><br>
**<[body](#body)>**<br>
<sub>`empty separator line`</sub><br>
**<[footer](#footer)>**

**[Commit Message Examples](#examples)**

### Types
* `feat` A code change that adds a new feature
* `fix` A code change that adds a bug fix
* `refactor` A code change that neither adds a new feature nor fixes a bug
* `style` A code change that do not affect the meaning (white-space, formatting, missing semi-colons, etc)
* `test` Changes that add missing tests or correcting existing tests
* `docs` Changes that affect documentation only
* `build` Changes that affect the build system or external dependencies
* `revert` A revert commit
  * In this case subject should be the header of the reverted commit. The Body should say: This reverts commit <commitHash>.

### Scopes
The `scope` provides additional contextual information.
* Is an **optional** part of the format
* Allowed Scopes depends on the specific project

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
  
  
## Git Prereceive Hook Script
```bash
#!/bin/bash

set -eo pipefail

rev_old="$1"
rev_new="$2"
ref_name="$3" 

commit_message_type_regex='(feat|fix|refactor|style|test|docs|build)'
commit_message_scope_regex='.{1,20}'
commit_message_subject_regex='.{1,100}'
commit_message_header_regex="^(revert: )?${commit_message_type_regex}(\(${commit_message_scope_regex}\))?: ${commit_message_subject_regex}$"

reject='false'

# Iterate over all the commits
for commit in $(git rev-list $rev_old..$rev_new); do
  commit_message_header=$(git show -s --format=%s $commit)
  if ! [[ "$commit_message_header" =~ ${commit_message_header_regex} ]]; then
    echo "$commit"
    echo ">> Invalid commit message header"
    echo "$commit_message_header"
    echo ""
    reject='true'
  fi
done

if [ "$reject" = 'true' ]; then
  exit 1
fi
```

-----
*References*
* https://www.conventionalcommits.org/
* https://github.com/angular/angular/blob/master/CONTRIBUTING.md
* http://karma-runner.github.io/1.0/dev/git-commit-msg.html

