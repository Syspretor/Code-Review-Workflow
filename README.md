# Code-Review-Workflow
Code review is necessary in industry development, this repo will introduce the code review workflow and share my experience in practice.

REFERENCE: https://github.com/pingcap/tidb/edit/master/CONTRIBUTING.md

## Workflow

### Step 1: Fork in the cloud

1. Visit https://github.com/pingcap/tidb
2. Click `Fork` button (top right) to establish a cloud-based fork.

### Step 2: Clone fork to local storage

Per Go's [workspace instructions][go-workspace], place TiDB's code on your
`GOPATH` using the following cloning procedure.

Define a local working directory:

```sh
# If your GOPATH has multiple paths, pick
# just one and use it instead of $GOPATH here.
working_dir=$GOPATH/src/github.com/pingcap
```

> If you already worked with Go development on github before, the `pingcap` directory
> will be a sibling to your existing `github.com` directory.

Set `user` to match your github profile name:

```sh
user={your github profile name}
```

Create your clone:

```sh
mkdir -p $working_dir
cd $working_dir
git clone https://github.com/$user/tidb.git
# the following is recommended
# or: git clone git@github.com:$user/tidb.git

cd $working_dir/tidb
git remote add upstream https://github.com/pingcap/tidb.git
# or: git remote add upstream git@github.com:pingcap/tidb.git

# Never push to upstream master since you do not have write access.
git remote set-url --push upstream no_push

# Confirm that your remotes make sense:
# It should look like:
# origin    git@github.com:$(user)/tidb.git (fetch)
# origin    git@github.com:$(user)/tidb.git (push)
# upstream  https://github.com/pingcap/tidb (fetch)
# upstream  no_push (push)
git remote -v
```

#### Define a pre-commit hook

Please link the TiDB pre-commit hook into your `.git` directory.

This hook checks your commits for formatting, building, doc generation, etc.

```sh
cd $working_dir/tidb/.git/hooks
ln -s ../../hooks/pre-commit .
```
Sometime, pre-commit hook can not be executable. In such case, you have to make it executable manually.

```sh
cd $working_dir/tidb/.git/hooks
chmod +x pre-commit
```

### Step 3: Branch

Get your local master up to date:

```sh
cd $working_dir/tidb
git fetch upstream
git checkout master
git rebase upstream/master
```

Branch from master:

```sh
git checkout -b myfeature
```

### Step 4: Develop

#### Edit the code

You can now edit the code on the `myfeature` branch.

#### Run stand-alone mode

If you want to reproduce and investigate an issue, you may need
to run TiDB in stand-alone mode.

```sh
# Build the binary.
make server

# Run in stand-alone mode. The data is stored in `/tmp/tidb`.
bin/tidb-server
```

Then you can connect to TiDB with mysql client.

```sh
mysql -h127.0.0.1 -P4000 -uroot test
```

If you use MySQL client 8, you may get the `ERROR 1105 (HY000): Unknown charset id 255` error. To solve it, you can add `--default-character-set utf8` in MySQL client 8's arguments.

```sh
mysql -h127.0.0.1 -P4000 -uroot test --default-character-set utf8
```

#### Run Test

```sh
# Run unit test to make sure all test passed.
make dev

# Check checklist before you move on.
make checklist
```

### Step 5: Keep your branch in sync

```sh
# While on your myfeature branch.
git fetch upstream
git rebase upstream/master
```

### Step 6: Commit

Commit your changes.

```sh
git commit
```

Likely you'll go back and edit/build/test some more than `commit --amend`
in a few cycles.

### Step 7: Push

When ready to review (or just to establish an offsite backup or your work),
push your branch to your fork on `github.com`:

```sh
git push -f origin myfeature
```

### Step 8: Create a pull request

1. Visit your fork at https://github.com/$user/tidb (replace `$user` obviously).
2. Click the `Compare & pull request` button next to your `myfeature` branch.

### Step 9: Get a code review

Once your pull request has been opened, it will be assigned to at least two
reviewers. Those reviewers will do a thorough code review, looking for
correctness, bugs, opportunities for improvement, documentation and comments,
and style.

Commit changes made in response to review comments to the same branch on your
fork.

Very small PRs are easy to review. Very large PRs are very difficult to
review.

## Code style

The coding style suggested by the Golang community is used in TiDB. See the [style doc](https://github.com/golang/go/wiki/CodeReviewComments) for details.

## Commit message style

Please follow this style to make TiDB easy to review, maintain and develop.

```
<subsystem>: <what changed>
<BLANK LINE>
<why this change was made>
<BLANK LINE>
<footer>(optional)
```

The first line is the subject and should be no longer than 70 characters, the
second line is always blank, and other lines should be wrapped at 80 characters.
This allows the message to be easier to read on GitHub as well as in various
git tools.

If the change affects more than one subsystem, you can use comma to separate them like `util/codec,util/types:`.

If the change affects many subsystems, you can use ```*``` instead, like ```*:```.

For the why part, if no specific reason for the change,
you can use one of some generic reasons like "Improve documentation.",
"Improve performance.", "Improve robustness.", "Improve test coverage."

[Os X GNU tools]: https://www.topbug.net/blog/2013/04/14/install-and-use-gnu-command-line-tools-in-mac-os-x
[go-1.8]: https://blog.golang.org/go1.8
[go-workspace]: https://golang.org/doc/code.html#Workspaces
[issue]: https://github.com/pingcap/tidb/issues
[mercurial]: http://mercurial.selenic.com/wiki/Download
