<!-- If the name of this document is changed, please update the link at http://blog.springsource.org/2009/03/03/building-spring-3 -->


_NOTE: these instructions apply to versions 3.1.x and earlier only!  As of Spring 3.2, the build system has [moved to Gradle](https://jira.springsource.org/browse/SPR-8116).  See the README in the root of the source tree for basic build instructions and also the [[Gradle build and release FAQ]]._
***


## Introduction
The Spring Framework is built using a combination of Ant and Ivy called _Spring Build_[1].

The steps to build the framework using the current build system have been documented at the SpringSource blog [here](http://blog.springsource.org/2009/03/03/building-spring-3/).  These steps are still accurate with the exception of checking out the source, as the framework has since moved from Subversion to Git/GitHub.

### Check out the sources
If you haven't already, you'll need to set up Git on your local machine[2]. See the instructions for [Windows](http://help.github.com/win-set-up-git/), [Mac](http://help.github.com/mac-set-up-git/) and [Linux](http://help.github.com/linux-set-up-git/).

### Clone the repository

    $ git clone git://github.com/SpringSource/spring-framework.git

### Or, _fork and clone_ the repository
As a variation on the above, you may wish to [create your own fork](http://help.github.com/fork-a-repo/) of the spring-framework repository.  This will give you full control, allow you to [submit pull requests](http://help.github.com/send-pull-requests/) back into the main repository and generally enjoy all the benefits that distributed version control has to offer.

Once you've created your own fork, then you can clone from it directly:

    $ git clone git@github.com:<my_username>/spring-framework.git

## Remaining steps
Once you have a local repository, you can continue with steps 2-6 as documented in the [blog post](http://blog.springsource.org/2009/03/03/building-spring-3/) mentioned above.  These steps are geared toward Eclipse users, and demonstrate how to get up and running in an IDE with Spring.  If you're more impatient, and just want the command-line instructions, here they are...

### Set up Ant
Make sure you've got a recent version of Ant (1.8.2 or better is a safe bet), and configure ANT_OPTS to ensure that the VM doesn't run out of memory during the build:

    $ ant -version
    Apache Ant(TM) version 1.8.2 compiled on October 14 2011
    
    $ export ANT_OPTS='-XX:MaxPermSize=1024m -Xmx1024m'

### To build all modules and run all tests

    $ ant -f build-spring-framework/build.xml test

The first time you run this, it'll take quite a while, as all of Spring's dependencies (even the optional ones) need to be downloaded into your local `ivy-cache` directory. After this initial downloading is complete, you can expect a complete build/test run to take anywhere between 5 and 20 minutes depending on your system.  If you don't care about running tests, and just want to build the jars, this typically takes only a couple of minutes.

### To build and install Spring artifacts into your local Maven cache
Perhaps you've made local changes to your `spring-framework` repository, and now you'd like to test them out against your application.  Assuming that your application is built with Maven or Gradle, you'll want to build the artifacts and "install" them into your local $HOME/.m2 repository.  Here's how to do it:

    $ ant -f build-spring-framework/build.xml jar install-maven-central

### To build a distribution
In rare cases, you may wish to build a distribution zip locally.  For example, if you cannot use a build system such as Maven or Gradle with dependency management capabilities, and you want to ensure that you have Spring and all necessary dependencies on your classpath.

#### Check out the version you wish to build

    $ git tag -l  # select a tag from the list, e.g. `v3.1.0.RELEASE`
    $ git co v3.1.0.RELEASE

#### Create the distribution zip
    $ ant -f build-spring-framework/build.xml package

When complete, `build-spring-framework/target/artifacts/spring-framework-3.1.0.RELEASE-dependencies.zip` will be available, containing all Spring module Jars and all external dependencies, both optional and required.

See also [[Downloading Spring artifacts]].

----

[1] With the completion of [SPR-8116](https://jira.springsource.org/browse/SPR-8116), Spring's build system will be ported to [Gradle](http://gradle.org). Put a watch on that issue, or stay tuned to [@springframework](http://twitter.com/springframework) and the SpringSource [team blog](http://blog.springsource.org) to be notified when this migration is complete.

[2] Actually, it is still possible to check out the repository using Subversion, thanks to GitHub's dedicated [support for Subversion clients](https://github.com/blog/966-improved-subversion-client-support):

    svn checkout https://github.com/SpringSource/spring-framework

(but even so, we recommend you give it a shot the Git way; you'll be glad you did)