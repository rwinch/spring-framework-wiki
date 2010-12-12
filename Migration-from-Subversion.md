## Steps followed migrating spring-framework from Subversion

*These steps loosely followed the migration process for spring-integration.  See [[INT-1348|https://jira.springsource.org/browse/INT-1348]]*

*GitHub's Subversion import feature was tried, but timed out due to the large size of the Spring source tree*

1. Create new github repository at [[https://github.com/cbeams/spring-framework]]
1. `sudo gem install svn2git --source http://gemcutter.org`
1. `svn2git --metadata --authors ~/.svn2git/[[authors|https://jira.springsource.org/secure/attachment/17435/authors]] --verbose https://src.springframework.org/svn/spring-framework`
1. `cd spring-framework`
1. `git remote add github-mirror git@github.com:cbeams/spring-framework.git`
1. `git push --all github-mirror`
1. `git push --tags github-mirror`

## Keeping the spring-framework Github mirror up to date with canonical Subversion repo

So long as https://src.springframework.org/svn/spring-framework remains the canonical repository for Spring development, this Git repository may be kept up to date with the following:

1. `cd spring-framework` (must be the original svn2git repository checked out above)
1. `git svn fetch`  # notice which branches are updated
1. `git co <branch>` # for each branch affected by the fetch
1. `git svn merge remotes/<branch>` # to actually merge the changes in from svn
1. `git push github-mirror && git push --tags github-mirror`