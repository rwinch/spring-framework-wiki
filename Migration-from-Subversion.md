## Steps followed migrating spring-framework from Subversion

*These steps loosely followed the migration process for spring-integration.  See [[INT-1348|https://jira.springsource.org/browse/INT-1348]]*

*GitHub's Subversion import feature was tried, but timed out due to the large size of the Spring source tree*

1. Create new github repository at [[https://github.com/cbeams/spring-framework]]
1. `sudo gem install svn2git --source http://gemcutter.org`
1. `svn2git -A ~/.svn2git/[[authors|https://jira.springsource.org/secure/attachment/17435/authors]] --verbose https://src.springframework.org/svn/spring-framework`
1. `cd spring-framework`
1. `git remote add origin git@git.springsource.org:spring-integration/spring-integration.git`
1. `git push --all --set-upstream origin`
1. `git push --tags origin`

## Keeping the spring-framework Github repo up to date with Subversion

So long as https://src.springframework.org/svn/spring-framework remains the canonical repository for Spring development, this Git repository may be kept up to date with the following:

1. `cd spring-framework` (must be the original svn2git repository checked out above)
1. `svn2git --rebase`
1. `git push && git push --tags`