_This page serves as a simple "how to" for Spring committers when manually merging pull requests from contributors._

# Placeholders

Many of the `git` commands used on this page contain placeholders as defined below.

- `<ACCOUNT>`: GitHub account for the author of the pull request
- `<BRANCH>`: branch in pull request author's account (e.g., SPR-####)
- `<PULL_REQUEST_NUMBER>`: spring-framework pull request number

### Example

To determine the values for these placeholders, let's take a look at [pull request #111](https://github.com/SpringSource/spring-framework/pull/111) as an example. From an _open_ pull request page you should be able to find an example `git` command for pulling the request into your local working directory. For example, if you click on the _info_ icon in the "This pull request can be automatically merged" bar, you should see something like: `git pull git://github.com/sbrannen/spring-framework.git SPR-9492`

From this information we determine the placeholder values to be the following.

- `<ACCOUNT>`: sbrannen
- `<BRANCH>`: SPR-9492
- `<PULL_REQUEST_NUMBER>`: 111

# Performing the Merge

## Set up remote, fetch branch, and rebase

```shell
$> git remote add <ACCOUNT> https://github.com/<ACCOUNT>/spring-framework.git
$> git fetch <ACCOUNT>
$> git checkout --track <ACCOUNT>/<BRANCH> -b <BRANCH>
$> git rebase master
```

## Modify working directory

- polish and format the contribution as necessary, in line with the [[Contributor guidelines]]
- refactor code as necessary to comply with Spring practices (i.e., aim for uniformity with existing code, naming conventions, etc.)
- update and/or add Javadoc and reference manual documentation 
- add a new entry in `src/dist/changelog.txt` if appropriate
- once changes are finalized and committed to the local branch, you typically will want to squash multiple commits into a single commit -- for example, using `git rebase --interactive --autosquash` -- however, sometimes it gives a more complete picture of the work that was performed in you leave multiple commits _as is_.
- if the _author_ is a VMware employee, ensure that the author's email (i.e., in the `Author:` attribute of the commit) points to his or her actual `@vmware.com` address -- for example, using `git commit --amend --author="Firstname Lastname <flastname@vmware.com>"`

## Merge into master and push

```shell
$> git checkout master
$> git merge --no-ff --log -m "Merge pull request #<PULL_REQUEST_NUMBER> from <ACCOUNT>/<BRANCH>" <BRANCH>
$> git push springsource master:master
```

Note that the above `git push` command assumes that you have configured a `springsource` remote similar to the following:

```shell
$> git remote show springsource
* remote springsource
  Fetch URL: git@github.com:SpringSource/spring-framework.git
  Push  URL: git@github.com:SpringSource/spring-framework.git
```

# Backporting

Once you have merged a pull request into `master`, you should evaluate whether the change is a possible candidate for backporting. If so, create a `Backport` sub-task for the JIRA issue corresponding to the pull request and schedule it for the appropriate _Maintenance_ version (e.g., "3.1 Maintenance"). This is the _'fire and forget'_ model for backporting, in which someone else comes along later and processes backports in bulk (at which point they are slated for a concrete maintenance release, e.g. 3.1.3).

See JIRA for [example backport sub-tasks](https://jira.springsource.org/secure/IssueNavigator!executeAdvanced.jspa?jqlQuery=project+%3D+SPR+and+issuetype+%3D+%27Backport%27&runQuery=true&clear=true).

# Cleaning Up

## Update remote pull request branch to rebased/modified version (optional)

_This is only possible if you have write permissions for the remote repository -- for example, if you are merging your own pull request._

```shell
$> git push --force <ACCOUNT> <BRANCH>
```

## Delete remote pull request branch (optional)

_This is only possible if you have write permissions for the remote repository -- for example, if you are merging your own pull request._

```shell
$> git push <ACCOUNT> :<BRANCH>
```

## Delete local branch (optional)

```shell
$> git branch -D <BRANCH>
```

## Delete local remote entry (optional)

```shell
$> git remote rm <ACCOUNT>
```
