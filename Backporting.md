## Introduction

This page lays out one possible workflow for backporting changes from spring-framework master onto a maintenance branch.  At the time of this writing, the task at hand is to backport several changes from the current Spring 3.1 codeline (master branch) onto a new Spring 3.0.6 branch.

This should be a trivial task thanks to the way Git branching works and how simple it is to 'cherry pick' individual commits from one branch to another.  The primary work items are:

1. Create a 3.0.6 branch
1. Identify which commits exist on 3.1 (master) that do not exist on 3.0.6
1. Choose which of those commits should be applied to 3.0.6 and apply or 'cherry-pick' them
1. Visualize the process along the way so we're sure which commits exist where
1. Test the contents of the 3.0.6 branch
1. Publish the 3.0.6 release

This document will cover all but the final testing and publication steps, as this is documented elsewhere.

*Please note this document assumes at least a basic familiarity with git concepts such as branches, commits, sha1 checksums.  Consider walking through the short and helpful [[http://gitref.org]] first if necessary.  More advanced operations such as cherry picking will be explained inline below.*

# Step one: Create 3.0.6 branch

Logically it makes sense to create the 3.0.6 branch from the tag for the Spring 3.0.5 branch:

    $ git branch 3.0.6 spring-framework-3.0.5

This command says exactly what one would expect: *create a branch called '3.0.6' from the commit tagged as 'spring-framework-3.0.5'*

This leaves us with the following commit history:

      (spring-framework-3.0.5)
                |
    *-*-*-*-*-*-*-*-*-*-*-* (master)
                |         |
             (3.0.6)    (HEAD)

that is to say, the commit history remains linear, and we now simply have an additional pointer to the commit tagged as 'spring-framework-3.0.5'.

Finally, check out the new 3.0.6 branch:

    $ git co 3.0.6
    Switched to branch '3.0.6'


# Step two: Identify commits present only on master

There are a number of ways this can be done, but perhaps the nicest is with `[[git show-branch|http://www.kernel.org/pub/software/scm/git/docs/git-show-branch.html]]`:

    $ git show-branch master 3.0.6
    ! [master] Add spring-build as git submodule
     * [3.0.6] Spring 3.0.5
    --
    +  [master] Add spring-build as git submodule
    +  [master^] Add readme with instructions for building from source
    +  [master~2] Eliminate PropertySourceAggregator interface
    +  [master~3] Expose Environment ConfigurationService
    +  [master~4] Eliminate reserved 'default' profile (SPR-7778)
    +  [master~5] Polish JavaDoc
    +  [master~6] Add Hamcrest 1.1 as test-time dependency for .context
    +  [master~7] Use dot notation rather than camel case for profile props (SPR-7508)
    +  [master~8] Support default profile (SPR-7508, SPR-7778)
    +  [master~9] Rename EnvironmentBeansTests* -> ProfileXmlBeanDefinitionTests*
    +  [master~10] Remove obsolete ConfigurationClassPostProcessor.getOrder()
    +  [master~11] SPR-7705: re-order rules and befores
    +  [master~12] Add hamcrest to beans pom in the right place to make tests compile
    +  [master~13] Fix .integration-tests build path errors
    +  [master~14] Add missing JPA dependency
    +  [master~15] Add missing Hamcrest dependency
    +  [master~16] Re-order deps to allow Hamcrest to come before JUnit
    +  [master~17] Add missing ROME dep
    +  [master~18] Using random port for HTTP integration tests
    +  [master~19] SPR-7707 - Unexpected behavior with class-level @RequestMappings
    +  [master~20] SPR-7703 - minor performance improvements to servlet and portlet handlers
    +  [master~21] SPR-7308 + add updated IDE classpath + add updated OSGi manifest
    +  [master~22] SPR-7308 + add updated IDE classpath + add updated OSGi manifest
    +  [master~23] SPR-7308 + initial commit of caching abstraction + main API + Spring AOP and AspectJ support + annotation driven, declarative support + initial namespace draft
    +  [master~24] SPR-6614 - Add human-readable descriptions for statuc codes in HttpStatus
    +  [master~25] SPR-7695 - Add ETag version of WebRequest.checkNotModified()
    +  [master~26] SPR-7470 + add missing test class
    +  [master~27] SPR-7470 + add test for XML config with errors
    +  [master~28] SPR-7470 + add c: namespace
    +  [master~29] Minor post-merge cleanup
    +  [master~30] Merge 3.1.0 development branch into trunk
    +  [master~31] polishing
     * [3.0.6] Spring 3.0.5
    +* [master~32] SPR-7667

This output tells us the following:
* `master` and `3.0.6` have only one commit in common: `[master~32] SPR-7667`
* `3.0.6` contains one commit not present on master: `Spring 3.0.5`
* `master` contains roughly 30 commits not present on `3.0.6`


## Step three: Apply selected commits to the backport branch

Our job is now to determine which of these commits should be applied to 3.0.6.  For example, it's likely that `[master~25] SPR-7695` is a fix that should be backported.

Applying this change to `3.0.6` is simple, using `[[git cherry-pick|http://www.kernel.org/pub/software/scm/git/docs/git-cherry-pick.html]]`:

    $ git cherry-pick master~25
    Finished one cherry-pick.
    [3.0.6 cb0ab72] SPR-7695 - Add ETag version of WebRequest.checkNotModified()
     Author: Arjen Poutsma <apoutsma@vmware.com>
     5 files changed, 157 insertions(+), 15 deletions(-)

This command says: *select the change 25 commits earlier than HEAD on the `master` branch, and commit it to the current branch (`3.0.6`)*

Now we can take a look at `git show-branch` again, and we'll see that 3.0.6 also contains this change:

    $ git show-branch master 3.0.6
    ! [master] Add spring-build as git submodule
     * [3.0.6] SPR-7695 - Add ETag version of WebRequest.checkNotModified()
    --
     * [3.0.6] SPR-7695 - Add ETag version of WebRequest.checkNotModified()
     * [3.0.6^] Spring 3.0.5
    +  [master] Add spring-build as git submodule
    +  [master^] Add readme with instructions for building from source
    +  [master~2] Eliminate PropertySourceAggregator interface
    +  [master~3] Expose Environment ConfigurationService
    +  [master~4] Eliminate reserved 'default' profile (SPR-7778)
    +  [master~5] Polish JavaDoc
    +  [master~6] Add Hamcrest 1.1 as test-time dependency for .context
    +  [master~7] Use dot notation rather than camel case for profile props (SPR-7508)
    +  [master~8] Support default profile (SPR-7508, SPR-7778)
    +  [master~9] Rename EnvironmentBeansTests* -> ProfileXmlBeanDefinitionTests*
    +  [master~10] Remove obsolete ConfigurationClassPostProcessor.getOrder()
    +  [master~11] SPR-7705: re-order rules and befores
    +  [master~12] Add hamcrest to beans pom in the right place to make tests compile
    +  [master~13] Fix .integration-tests build path errors
    +  [master~14] Add missing JPA dependency
    +  [master~15] Add missing Hamcrest dependency
    +  [master~16] Re-order deps to allow Hamcrest to come before JUnit
    +  [master~17] Add missing ROME dep
    +  [master~18] Using random port for HTTP integration tests
    +  [master~19] SPR-7707 - Unexpected behavior with class-level @RequestMappings
    +  [master~20] SPR-7703 - minor performance improvements to servlet and portlet handlers
    +  [master~21] SPR-7308 + add updated IDE classpath + add updated OSGi manifest
    +  [master~22] SPR-7308 + add updated IDE classpath + add updated OSGi manifest
    +  [master~23] SPR-7308 + initial commit of caching abstraction + main API + Spring AOP and AspectJ support + annotation driven, declarative support + initial namespace draft
    +  [master~24] SPR-6614 - Add human-readable descriptions for statuc codes in HttpStatus
    +  [master~25] SPR-7695 - Add ETag version of WebRequest.checkNotModified()
    +  [master~26] SPR-7470 + add missing test class
    +  [master~27] SPR-7470 + add test for XML config with errors
    +  [master~28] SPR-7470 + add c: namespace
    +  [master~29] Minor post-merge cleanup
    +  [master~30] Merge 3.1.0 development branch into trunk
    +  [master~31] polishing
    +* [3.0.6~2] SPR-7667

Notice toward the top that the `SPR-7695` commit is now present on the `3.0.6` branch.  And it remains present at `master~25` as well.

At this point, you might be wondering why this does not show up on a single line with `+*`, indicating it is the same commit, and present on both branches.  The reason for this is that it is not actually the *same commit*, but is rather the *same diff* having been applied to both branches.  That is, the change made is the same, but the two commits have different SHA1 checksum values.  This is the one of the key differences between truly merging two branches together and cherry-picking individual commits from one branch to another.

So at this point, the output isn't exactly ideal for facilitating the backporting process.  It would be nice if we could see not *which commits* are applied to which branches, but *which changes* or *diffs* are applied to each branch.  And this is exactly what `[[git cherry|http://www.kernel.org/pub/software/scm/git/docs/git-cherry.html]]` was designed for:

    $ git cherry -v 3.0.6 master
    + b0ea2b13d21992002a811dc52d89b7638040fd4a polishing
    + f480333d31d8307b8c96409e7bb4f06ec0cab0ca Merge 3.1.0 development branch into trunk
    + 4214bc0735c66c669a50f6391b917c54854eea5c Minor post-merge cleanup
    + c13905ad1627f9d91e54ffd199c64380bbf18eb3 SPR-7470 + add c: namespace
    + 6ef987bcedd6c97f9a7683fbefa60b05f66cf408 SPR-7470 + add test for XML config with errors
    + 095a36e853e7c3977f064f97db7f4af641145bdc SPR-7470 + add missing test class
    - 7cc3f49910b6ed281f1aee0851e603dfa96ed48a SPR-7695 - Add ETag version of WebRequest.checkNotModified()
    + 416777022dff7a420d3d0d3d1a04c71905b67828 SPR-6614 - Add human-readable descriptions for statuc codes in HttpStatus
    + 85c02981b5f92f7dae92ffe2c9bb0657e6a7caac SPR-7308 + initial commit of caching abstraction + main API + Spring AOP and AspectJ support + annotation driven, declarative support + initial namespace draft
    + 4f8105ccaa1c0d1b1ab99fb108a60ea787a0d33a SPR-7308 + add updated IDE classpath + add updated OSGi manifest
    + caf1a0875ac51544aec99129e29e3bbfa94f9b12 SPR-7308 + add updated IDE classpath + add updated OSGi manifest
    + 01e79cfeddf788b3059f9012c0e19aca87e3902c SPR-7703 - minor performance improvements to servlet and portlet handlers
    + 8762ec956c8d8e18979770a0ddc836b971e88fc1 SPR-7707 - Unexpected behavior with class-level @RequestMappings
    + 01120eb2f0c2b0ecb59daa4d31643e5e3a3beed2 Using random port for HTTP integration tests
    + b73224427f296a1f016e0334bd189eb9e2bf3b2f Add missing ROME dep
    + a3df1c4e416c3d12072bafd0c63a8c93d85fbe55 Re-order deps to allow Hamcrest to come before JUnit
    + cfe1fdb5e596ccf977d8eaef60f2acb1fe700248 Add missing Hamcrest dependency
    + 36ec06a9171c290e168f9257726a45edb58c05e6 Add missing JPA dependency
    + 197a46d0ab5b7008fbc68eaa7468f363fca0b91b Fix .integration-tests build path errors
    + c52915bde6ac2eab7554fd665870d0c4f4592307 Add hamcrest to beans pom in the right place to make tests compile
    + b109a07fd99262204605b560bdba036c95eb093b SPR-7705: re-order rules and befores
    + c8b49158916431afabb1989a87a09465aaff1c3a Remove obsolete ConfigurationClassPostProcessor.getOrder()
    + b33da670e5304ae44fc0817cebf200a69a5f457e Rename EnvironmentBeansTests* -> ProfileXmlBeanDefinitionTests*
    + 5062dc31af63691f91a8a473803653d4d36d9a39 Support default profile (SPR-7508, SPR-7778)
    + e0c5ced6957b9fdc6667d5a41811ac019994d15e Use dot notation rather than camel case for profile props (SPR-7508)
    + b50abc67deb639c5a29c286cb19bd08780ec5f4d Add Hamcrest 1.1 as test-time dependency for .context
    + e693d9fa587db6f3392fb545cffbd3e50893c4a1 Polish JavaDoc
    + b3e36a335d98945f8bf04ebb80ebe31664283720 Eliminate reserved 'default' profile (SPR-7778)
    + 8770ea96b03106a06ee7ce401dc2fe39f419f816 Expose Environment ConfigurationService
    + f46a455c72370714a7494ff95bf9d4cd927ebac8 Eliminate PropertySourceAggregator interface
    + 2ba55ce3ae6427eff1297044ef40174cb78e24cf Add readme with instructions for building from source
    + 86385ea9f1db9da3df54b5ae8caf57b79c9ac835 Add spring-build as git submodule

This command says: *find all the changes on `master` that have not been applied to `3.0.6`*
Per the explanation on the [[git-cherry manpage|http://www.kernel.org/pub/software/scm/git/docs/git-cherry.html]], lines prefixed with a `+` indicate a change that has not been applied, while lines prefixed with `-` indicate the change exists on both branches.  That's exactly what we want to see.  We can now work our way through the rest of the commits deciding which should be applied, and cherry-picking as we go.  This time around, we can cherry pick the sha1 checksum instead of specifying a relative commit like "master~25" as was done before:

    $ git cherry-pick 416777022dff7a420d3d0d3d1a04c71905b67828
    Finished one cherry-pick.
    [3.0.6 7b1a825] SPR-6614 - Add human-readable descriptions for statuc codes in HttpStatus
     Author: Arjen Poutsma <apoutsma@vmware.com>
     1 files changed, 68 insertions(+), 58 deletions(-)

Afterward, our `git cherry` output updates to the following:

    $ git cherry -v 3.0.6 master
    + b0ea2b13d21992002a811dc52d89b7638040fd4a polishing
    + f480333d31d8307b8c96409e7bb4f06ec0cab0ca Merge 3.1.0 development branch into trunk
    + 4214bc0735c66c669a50f6391b917c54854eea5c Minor post-merge cleanup
    + c13905ad1627f9d91e54ffd199c64380bbf18eb3 SPR-7470 + add c: namespace
    + 6ef987bcedd6c97f9a7683fbefa60b05f66cf408 SPR-7470 + add test for XML config with errors
    + 095a36e853e7c3977f064f97db7f4af641145bdc SPR-7470 + add missing test class
    - 7cc3f49910b6ed281f1aee0851e603dfa96ed48a SPR-7695 - Add ETag version of WebRequest.checkNotModified()
    - 416777022dff7a420d3d0d3d1a04c71905b67828 SPR-6614 - Add human-readable descriptions for statuc codes in HttpStatus
    + 85c02981b5f92f7dae92ffe2c9bb0657e6a7caac SPR-7308 + initial commit of caching abstraction + main API + Spring AOP and AspectJ support + annotation driven, declarative support + initial namespace draft
    + 4f8105ccaa1c0d1b1ab99fb108a60ea787a0d33a SPR-7308 + add updated IDE classpath + add updated OSGi manifest
    + caf1a0875ac51544aec99129e29e3bbfa94f9b12 SPR-7308 + add updated IDE classpath + add updated OSGi manifest
    + 01e79cfeddf788b3059f9012c0e19aca87e3902c SPR-7703 - minor performance improvements to servlet and portlet handlers
    + 8762ec956c8d8e18979770a0ddc836b971e88fc1 SPR-7707 - Unexpected behavior with class-level @RequestMappings
    + 01120eb2f0c2b0ecb59daa4d31643e5e3a3beed2 Using random port for HTTP integration tests
    + b73224427f296a1f016e0334bd189eb9e2bf3b2f Add missing ROME dep
    + a3df1c4e416c3d12072bafd0c63a8c93d85fbe55 Re-order deps to allow Hamcrest to come before JUnit
    + cfe1fdb5e596ccf977d8eaef60f2acb1fe700248 Add missing Hamcrest dependency
    + 36ec06a9171c290e168f9257726a45edb58c05e6 Add missing JPA dependency
    + 197a46d0ab5b7008fbc68eaa7468f363fca0b91b Fix .integration-tests build path errors
    + c52915bde6ac2eab7554fd665870d0c4f4592307 Add hamcrest to beans pom in the right place to make tests compile
    + b109a07fd99262204605b560bdba036c95eb093b SPR-7705: re-order rules and befores
    + c8b49158916431afabb1989a87a09465aaff1c3a Remove obsolete ConfigurationClassPostProcessor.getOrder()
    + b33da670e5304ae44fc0817cebf200a69a5f457e Rename EnvironmentBeansTests* -> ProfileXmlBeanDefinitionTests*
    + 5062dc31af63691f91a8a473803653d4d36d9a39 Support default profile (SPR-7508, SPR-7778)
    + e0c5ced6957b9fdc6667d5a41811ac019994d15e Use dot notation rather than camel case for profile props (SPR-7508)
    + b50abc67deb639c5a29c286cb19bd08780ec5f4d Add Hamcrest 1.1 as test-time dependency for .context
    + e693d9fa587db6f3392fb545cffbd3e50893c4a1 Polish JavaDoc
    + b3e36a335d98945f8bf04ebb80ebe31664283720 Eliminate reserved 'default' profile (SPR-7778)
    + 8770ea96b03106a06ee7ce401dc2fe39f419f816 Expose Environment ConfigurationService
    + f46a455c72370714a7494ff95bf9d4cd927ebac8 Eliminate PropertySourceAggregator interface
    + 2ba55ce3ae6427eff1297044ef40174cb78e24cf Add readme with instructions for building from source
    + 86385ea9f1db9da3df54b5ae8caf57b79c9ac835 Add spring-build as git submodule

Notice that there are now two lines prefixed with `-`.

Continue the process above until all necessary commits have been cherry picked over to the `3.0.6` branch.


## Summary

Here we've shown how simple it is to port changes from one branch to another.  Now that all the exposition is out of the way, let's review the actual commands necessary to perform this kind of backporting task:

1. `git checkout -b 3.0.6 spring-framework-3.0.5` # *more convenient than `git branch X` followed by `git checkout X`*
1. `git cherry-pick <commit-sha>`
1. repeat for as many commits as necessary
1. visualize the state of 3.0.6 using `git show-branch`, `git cherry`, and/or a git gui
1. test the changes, publish the release.


In the process we've shown two tools for visualizing what's going on: `git show-branch` and `git cherry`. Of course you can choose to use either or both tools based on preference and style.  It's also possible to use `gitx`, `gitk`, or any of the other git guis to visualize the process.  In Git, there's usually more than one way to do it.  Both tools have been shown here to help provide context about what we're actually looking at: *commits* vs *changes* or *diffs* having been applied.

As a final note, let's inspect one of the cherry-picked changes, using `[[git log|http://www.kernel.org/pub/software/scm/git/docs/git-log.html]]`. Remember, our current working context is still `3.0.6`, so the log we're seeing is the commit log of that branch:

    $ git log -2
    commit 7b1a82538676c412d5767635f061e09e965853b0 (HEAD, 3.0.6)
    Author: Arjen Poutsma <apoutsma@vmware.com>
    Date:   Fri Oct 29 10:56:43 2010 +0000

        SPR-6614 - Add human-readable descriptions for statuc codes in HttpStatus

    commit cb0ab729d8cc8b4445a12b0702264299dd0c47ad
    Author: Arjen Poutsma <apoutsma@vmware.com>
    Date:   Fri Oct 29 10:28:47 2010 +0000

        SPR-7695 - Add ETag version of WebRequest.checkNotModified()

Notice how the 'Author' remains Arjen Poutsma, who did indeed originally author the commit, even though it was cherry-picked and commited to `3.0.6` by someone else -- Chris Beams, in this case.  Looking a little more closely with `git show`, we see the whole story:

    $ git show --pretty=full --name-only cb0ab729d8cc8b4445a12b0702264299dd0c47ad
    commit cb0ab729d8cc8b4445a12b0702264299dd0c47ad
    Author: Arjen Poutsma <apoutsma@vmware.com>
    Commit: Chris Beams <cbeams@vmware.com>

        SPR-7695 - Add ETag version of WebRequest.checkNotModified()

    org.springframework.web.portlet/src/main/java/org/springframework/web/portlet/context/PortletWebRequest.java
    org.springframework.web/src/main/java/org/springframework/web/context/request/FacesWebRequest.java
    org.springframework.web/src/main/java/org/springframework/web/context/request/ServletWebRequest.java
    org.springframework.web/src/main/java/org/springframework/web/context/request/WebRequest.java
    org.springframework.web/src/test/java/org/springframework/web/context/request/ServletWebRequestTests.java

Notice that Git makes a distinction here between the author of a change and who committed it.  In this case, it's nice because the backporting was a maintentance activity that should not obscure original authorship, while at the same time it's nice to know who committed the change to the backport branch.  This author/committer distinction is also useful when dealing with contributed patches, giving the author their due even though they did not have the rights to make the commit themselves.

