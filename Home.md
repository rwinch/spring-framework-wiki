## Welcome!

This repository serves as a mirror of the canonical Spring Framework repository at https://src.springsource.org/svn/spring-framework.  Commits made against trunk there are pushed to master here on a roughly daily basis.

Feel free to [watch and/or fork](http://help.github.com/fork-a-repo/) this repository if you prefer to keep up with Spring development via Git instead of SVN.

[Pull requests](http://help.github.com/send-pull-requests) are welcome, particularly for quick typo fixes or other cosmetic issues. Just [fork and edit](https://github.com/blog/844-forking-with-the-edit-button)!

Also, this build is under continuous integration at https://build.springsource.org/browse/SPR-MIRROR

If you think you've found a bug with the framework and want to submit an issue, you can do that at http://jira.springsource.org.  And since you're someone git-savvy, we'd love it if you'd also submit a "reproduction project" to our [spring-framework-issues repository](https://github.com/SpringSource/spring-framework-issues#readme).  Thanks!

# Build Instructions

## Prerequisites

* Ant 1.7.1
* JDK 6
* git (obviously)
* Subversion (any recent version will do) - this is to export the spring-build submodule

## One-time setup

```bash
git clone git://github.com/cbeams/spring-framework.git
cd spring-framework
svn export https://src.springsource.org/svn/spring-build/tags/project-build-2.5.2 spring-build
```

## Build!

_Make sure you configure Ant with enough memory before running any of these commands_

```bash
export ANT_OPTS=-XX:MaxPermSize=512m -Xmx1024m # might want to put that in your .bashrc
```

### Compile and test
```
ant -f build-spring-framework/build.xml test
```

### Compile, package and install artifacts into your local Maven cache ($HOME/.m2)
```
ant -f build-spring-framework/build.xml jar install-maven-local
```

## More information

If you're looking for more information on building the framework, for example setting it up within your IDE, take a look at this [blog post](http://blog.springsource.com/2009/03/03/building-spring-3/).

----

* [[Migration from Subversion]]
* [[Backporting]]