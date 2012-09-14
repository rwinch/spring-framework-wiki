_This FAQ has been written with building and releasing the Spring Framework in mind, but is also general enough to serve as a guide for any Spring project that publishes to http://repo.springsource.org.  The audience is primarily geared for project leads and committers, but should be valuable for anyone building from source or wanting to understand better how Spring is published, etc.  The focus is on Gradle-based builds, but keep in mind that Maven-based Spring projects can take advantage of the same general steps, e.g. using the [Artifactory Maven 3 Task](http://wiki.jfrog.org/confluence/display/RTF/Bamboo+Artifactory+Plug-in) instead of the Artifactory Gradle Task, etc.  Note that if you're unfamiliar with the benefits of Artifactory and repo.springsource.org in general, you may want to check out the [[SpringSource repository FAQ]]._

### Table of Contents
build-related

* [How do I check out and build the Framework?](#wiki-check_out_and_build)
* [How long should a build take?](#wiki-build_duration)
* [Why are there so many Javadoc warnings?](#wiki-javadoc_warnings)
* [Why are compile-time warnings suppressed?](#wiki-compilation_warnings)
* [What are the most important tips for Gradle newbies?](#wiki-gradle_tips)
* [Why do I get 401/403 errors when downloading dependencies?](#wiki-authentication)

release-related

* [How and where are snapshots published?](#wiki-snapshot_publication)
* [How do I perform a Milestone, RC, or GA release?](#wiki-release_process)
* [What about publishing artifacts to Maven Central?](#wiki-maven_central)
* [How are docs and schemas and distribution zips published?](#wiki-docs_schema_dist_publication)

***
<a name="wiki-check_out_and_build"/>
# How do I check out and build the Framework?
See "Building from source" in the README in root of the 3.2.x (master) source tree.  This also includes information on importing projects into Eclipse/STS/IDEA.

***
<a name="wiki-build_duration"/>
# How long should a build take?
When running `./gradlew build` for the first time, it is likely that the Gradle wrapper script will need to download Gradle for you.  Gradle distributions run around 30MB, so this step will be connection-speed dependent.

You will also need to download all of Spring's compile- and test-time dependencies.  This currently runs around 120MB.  Keep in mind here that almost all of these dependencies are optional at runtime for applications that use Spring.  It's only when building from source that you actually need them all!

Once you've bootstrapped a Gradle distribution and downloaded dependencies, you won't need to do it again; they are cached in your $HOME/.gradle directory.  A complete `./gradle clean build` at this point will take between 5 and 15 minutes depending on your clock and disk speed.  A solid state drive makes a huge difference here!

As is also mentioned below in the 'tips' section, you'll want to break yourself of any habits executing the `clean` task during normal development iterations.  Gradle has excellent _incremental build_ support, meaning that once you've built Javadoc, compiled, run test, etc, Gradle won't execute these tasks again unless the inputs for those tasks (e.g. .java files) have changed.  As a general rule, just run `./gradle build` or `./gradle test` (without `clean`) to keep things snappy.

Also, consider running with the `-a` flag to avoid evaluating other subprojects you depend on. For example, if you're iterating on changes in spring-webmvc, cd into the spring-webmvc directory and run `../gradlew -a build` to tell gradle to evaluate and build *only* that subproject.

Finally, the Gradle daemon helps greatly in eliminating startup overhead.  This feature will soon be 'on by default', but in the meantime  you need to run gradle with the `--daemon` flag, or alternatively, export your `GRADLE_OPTS` environment variable to include `-Dorg.gradle.daemon=true`

***
<a name="wiki-javadoc_warnings"/>
# Why are there so many Javadoc warnings?
When running `./gradlew build` for the first time (or any time one runs `./gradlew clean build`), you will notice that most subprojects have emit many warnings during the `javadoc` task.  This is simply because, over time, many warnings have accumulated.  Committers are encouraged to fix these proactively, and pull requests are welcome from the community.  It's a general goal to get these cleaned up as quickly as possible.

***
<a name="wiki-compilation_warnings"/>
# Why are compile-time warnings suppressed?
You'll notice that `build.gradle` includes the following line:

    [compileJava, compileTestJava]*.options*.compilerArgs = ['-Xlint:none']

This tells Gradle to suppress all warnings during compilation.  The reason for this is that the framework currently has many warnings, most of which are related to generics usage -- particularly raw type warnings -- e.g. using `Class` instead of `Class<?>`.  This is an artifact switching to Java 5 in Spring 3.  As with the Javadoc warnings mentioned above, committers are encouraged to fix these warnings whenever possible.  Once the bulk of them are eliminated, we can switch to -Xlint:all.  In the meantime, it's just creates unnecessary noise in the build output.

***
<a name="wiki-gradle_tips"/>
# What are the most important tips for Gradle newbies?

## Use the Gradle daemon
As mentioned above, the daemon greatly speeds up Gradle by eliminating startup overhead.  Run with `--daemon` or set `GRADLE_OPTS=-Dorg.gradle.daemon=true`.  Very occasionally, you may need or want to run `./gradlew --stop` to kill the daemon process.

## Stop typing `clean` unless you really need it
As mentioned above, Gradle has excellent _incremental build_ capabilities, meaning that it will only compile, run tests, generate javadoc, etc. if content has actually changed.  However, telling gradle to `clean` everything eliminates this benefit.  It is a common practice -- particularly for longtime Maven users -- to type 'clean' with every build as a matter of course.  Break yourself of this habit and you'll find builds running much faster.

## Understand selective / task-specific cleaning
Following onto the item above, there are certainly cases in which you want to clean, but you may not want to clean _everything_.  Instead of typing `./gradlew clean build`, consider using selective cleaning, in which you clean only the outputs for a given task.  For example

    $ ./gradlew :spring-oxm:cleanTest build

will eliminate the only the `spring-oxm/build/classes/test` directory and then run the entire build.  This means that you still benefit from incremental build, but for sure will re-execute spring-oxm's test suite.  The rule here is "clean<Task>", where <Task> is the capitalized name of the task you want to clean.  Other examples would be: `cleanJar`, `cleanJavadoc`, `cleanDistZip`, etc.

## Understand the `-a/--no-rebuild` switch
As mentioned earlier in this document, the `-a` switch tells Gradle to ignore evaluting and building any other subprojects.  An example would be when iterating on changes to spring-webmvc.  You may be running `./gradlew test` over and over again in the spring-webmvc directory, and each time you do this, Gradle will check all the other subprojects that spring-webmvc depends on to see if they've changed.  This is generally useful functionality, but if you know better, you may want to save yourself the extra few seconds that these checks can take.  Simply run `../gradlew -a test` within the spring-webmvc directory, and you'll see the difference.

## Run only the tasks you need
Rather than running `./gradlew build` every time you want to simply compile and test, run `./gradlew test`.  If you want only to generate the jar for a particular subproject, run `../gradlew jar` in that subproject's directory.

Run `./gradlew tasks` to get an overview of available tasks and their descriptions.  Run `./gradlew tasks --all` to get an even more exhaustive list.

## Qualify tasks by :project for precision
When you run `./gradlew build` from the root directory, Gradle recurses through all projects, finding those that actually have a 'build' task, and executing it if it exists.  This kind of heuristic behavior makes for convenient usage, but you can also be more specific.  For example, from the root directory you can run `./gradlew :spring-core:test` and Gradle will run the 'test' task only within the spring-core subproject.

A particularly relevant example of this is when running the integration tests for Spring Framework.  As of the switch to Gradle, there is no longer a dedicated integration-tests subproject.  Rather, these tests now live in the root project's src/test directory.  If you run `./gradlew test` from the root directory, Gradle will run not only the tests in the root src/test directory, but also the tests for every subproject!  To narrow it down and run _only_ the root project tests, type `./gradlew :test`.  The ':' here says "the 'test' task in the root project". Contrast this again with `./gradlew :spring-core:test`.

## Skip tasks with the `-x/--exclude-task` switch
Perhaps you'd like to run a complete `clean build`, but don't want to take the time to run the test suites?  Do this with `./gradlew clean build -x test`.  -x tells Gradle to exclude the task from the build lifecycle.  Specify multiple tasks to exclude with multiple '-x' switches, e.g. `./gradlew clean build -x javadoc -x test`

## Run a single test from the command line
Perhaps you're iterating on a particular unit test in spring-context and don't want to run the thousands of tests in its test suite just to see if your latest changes work.  Run `./gradlew test -Dtest.single=MyTests`.  There's no need to qualify the packaging here.  Gradle will simply find any class named 'MyTests' and run just those tests.

## Get more information with the `--info` switch
By default, Gradle's command line output is blissfully minimalistic.  It will only tell you at a high level what it's doing right now, e.g.:

    :spring-asm:compileJava UP-TO-DATE
    :spring-asm:processResources UP-TO-DATE
    :spring-asm:classes UP-TO-DATE
    :spring-asm:repackageAsm UP-TO-DATE
    :spring-asm:jar UP-TO-DATE
    :spring-core:compileJava

If there are warnings, e.g. compiler or Javadoc warnings, those will show up, because Gradle wants you to see that there's something wrong.  But if everything is going well, Gradle just keeps quiet.  In certain cases, however, you might want to see exactly which tests are running when, or exactly what requests are being made to download dependencies, etc.  Run `./gradlew -i` (or `--info`) to get a bit more verbosity.  Run with `-d` /  `--debug` to switch on the firehose.

## Use Gradle's built-in IDE metadata generation
Prior to the switch to Gradle, we manually maintained Eclipse and IDEA project metadata.  This a hassle and error-prone to say the least.  Gradle provides robust support for generating this metadata, and for this reason we no longer check these files into source control.  You'll notice that all these files now have entries in .gitignore.  Please don't check them in!

See the `import-into-eclipse.sh` and `import-into-idea.md` files in the root of the source tree for detailed instructions.

Note that both IDEA and STS are hard at work on even more advanced Gradle tooling from the IDE side.  At the time of this writing, neither one is quite capable of meeting all the needs of a build as complex as Spring's, but they'll get there soon.  In the meantime, Gradle's built-in support works quite well.

## Understand handling of 'optional' and 'provided' dependencies.
[This commit](https://github.com/SpringSource/spring-framework/commit/b6cb514d383dcef52ba6c609a863f19e1a4c1faf) provides all the detail you'll ever want on how we manage optional and provided dependencies with regard to our generated Maven poms.

## Use the `init.gradle` init script to configure repo.springsource.org authentication
See the FAQ item on [authentication](#wiki-authentication) below.

## Use `mavenLocal()` to resolve dependencies from your Maven cache
If you need to compile against a locally built version of a dependency, you'll need to add your local $HOME/.m2 repository to the set of repositories that Gradle searches during the build.

This can be done directly in your build script by adding `mavenLocal()` as follows:

    repositories {
        mavenLocal()
        maven { url "http://repo.springsource.org/libs-release" }
    }

However, it's recommended that you do this via an init script, to avoid accidentally checking in the `mavenLocal()` entry.  If you're already using the authentication init script described above, you can append the following to it, or if not, just create a new file in `$HOME/.gradle` named `init.gradle` with the following:

    allprojects {
        repositories {
            mavenLocal()
        }
    }

## Use `gradle install` to publish artifacts to your local Maven cache
If you're building Spring artifacts from source in order to compile against them from another project (perhaps another Spring project, even, like Spring Batch or Spring Integration or the Spring Data-* family), you'll need to publish jars into your local $HOME/.m2 cache in order to pick them up from the dependent project.  Do this with `./gradlew install`.

## Consider using the 'find-gradle' convenience script
The Gradle wrapper is a big help, but it can be cumbersome to have to use relative pathing, e.g. `./gradlew`, `../gradlew` when working with the build.  Follow the instructions here to install a simple shell script that allows you to type `gradle` anywhere and finds the nearest `gradlew` or falls back to your system-wide Gradle installation (if you have one): https://github.com/cbeams/shell-scripts/blob/master/find-gradle

***
<a name="wiki-authentication"/>
# Why do I get 401/403 errors when downloading dependencies?
Our Artifactory instance at http://repo.springsource.org is configured such that requesting and caching external dependencies, e.g. junit or cglib from Maven Central) can only be performed by authenticated users, i.e. SpringSource employees.  Once such a dependency has been successfully cached in the repository, then even anonymous/unauthenticated users may download them.  This configuration helps ensure that repo.springsource.org contains only Spring project artifacts and their transitive dependencies, and that we don't act as a kind of 'open proxy'.

This means that if you are not authenticated, you will not receive 401/403 unless _you're downloading a new dependency for the first time._  A typical example might be when upgrading to a newly-released version of JUnit or Hibernate.  The solution to this problem is to provide your Artifactory authentication credentials to Gradle, such that it can successfully respond to 401 challenges when Artifactory issues them.  The best way to do this is with a Gradle _init script_, which keeps your username and password separate from your build script.

See https://github.com/SpringSource/gradle-init-scripts#readme for simple instructions on how to dowload and install an init script that does just this.  This is recommended for all committers.

***
<a name="wiki-snapshot_publication"/>
# How and where are snapshots published?
The [3.2.x CI build plan](https://build.springsource.org/browse/SPR-B32X) is configured to use the [Artifactory Bamboo plugin](http://wiki.jfrog.org/confluence/display/RTF/Bamboo+Artifactory+Plug-in). Each time commits are pushed to [SpringSource/master](https://github.com/SpringSource/spring-framework/tree/master), this build plan

1. executes `./gradlew build`, producing jars and distribution zip files
2. publishes these artifacts to http://repo.springsource.org/libs-snapshot-local ([browse in Artifactory tree view](http://repo.springsource.org/webapp/browserepo.html?pathId=libs-snapshot-local%3Aorg%2Fspringframework))

Along with individual artifacts, [build metadata](http://wiki.jfrog.org/confluence/display/RTF/Build+Integration) is also published to Artifactory, including information about the environment and principal involved in the build, as well as allowing for bi-directional links between Bamboo and Artifactory. Click on the 'Artifactory' tab for any successful CI build to navigate to the associated Artifactory build.

![Bamboo->Artifactory](http://wiki.jfrog.org/confluence/download/attachments/14811516/bamboo.build.result.14.07.11.png?version=1&modificationDate=1310671106000)

Likewise, click on "Show in CI Server" from any individual Artifactory build to navigate back to the CI build that published it.

![Artifactory->Bamboo](http://wiki.jfrog.org/confluence/download/attachments/12157118/builds.by.name.03.05.11.png?version=1&modificationDate=1304430459000)

Users wishing to consume snapshot builds may add http://repo.springsource.org/snapshot to the list of repositories in their build script. See [[Downloading Spring artifacts]] for more details.

***
<a name="wiki-release_process"/>
# How do I perform a Milestone, RC, or GA release?

The steps are simple, and almost everything is done via the Bamboo and Artifactory UIs. 

## One-time setup

Configure your CI build plan to use the [Artifactory Maven 3 or Artifactory Gradle tasks](http://wiki.jfrog.org/confluence/display/RTF/Bamboo+Artifactory+Plug-in#BambooArtifactoryPlug-in-ConfiguringMaven3%2CGradleandIvyBuilders) as appropriate.  For "Deployer Username", use "buildmaster" (password on request).

## Steps at a glance

1. Stage the release into the libs-staging-local repository
2. Verify and test the staged artifacts
3. Promote the release to libs-milestone-local (or libs-release-local as appropriate).
4. Merge release branch
5. Announce the release

## Steps in detail

###  1. Stage the release
The Artifactory Bamboo plugin mentioned above also includes sophisticated [Release Management](http://wiki.jfrog.org/confluence/display/RTF/Bamboo+Artifactory+Plugin+-+Release+Management) capabilities. This feature allows for publishing releases _directly from CI_, including creating a release branch and/or tag; incrementing the project version; and publishing to the libs-staging-local, libs-milestone-local or libs-release-local repositories as appropriate.

To access this feature, click on the "[Default Job](https://build.springsource.org/browse/SPR-B32X-JOB1)" for the Spring 3.2.x build plan, where you'll see a link to "[Artifactory Release Management](https://build.springsource.org/build/release/viewVersions.action?buildKey=SPR-B32X-JOB1)".  Fill out the form fields there and click "Build and Release to Artifactory". Typical values -- in this case for a milestone release -- look something like the following:

[[release-staging.png]]

In the example above, the `version` property key refers to the property of the same name  declared in `gradle.properties` in the root of the source tree. This value will actually be modified and updated in source control during the release process.

Using a 'release branch' is optional, but recommended.  This means that updates to the `gradle.properties` file will occur on a branch named 3.2.0.M1, helping to isolate the release from changes on the master branch, and also allowing for simplified rollback in case a last minute change needs to be made after staging the release.

'Create VCS Tag' is also checked, indicating that a git tag named 'v3.2.0.M1' should be created, pointing to the commit where the `version` property is incremented.

Notice that 'Next development version comment' is blank.  Because this is a milestone release, the 'next integration value' of the `version` property is configured to return to it's previous value of '3.2.0.BUILD-SNAPSHOT'.  The tooling is smart enough here to avoid creating an additional commit, so no comment is necessary.

Importantly, notice that we're publishing to the 'libs-staging-local' repository - this is just what it sounds like: a staging area that allows us to test out the release internally before finally promoting it to the actual 'libs-milestone-local' repository and announcing it to the world.

With these values supplied, click 'Build and Release to Artifactory'.

###  2. Verify staged artifacts
When the staging build and release process is complete, you can navigate to the associated build record in Artifactory to verify that all modules were published as expected, e.g.:

[[build-browser.png]]

Note that in the Artifactory tree view, you can easily drill into jars and zips to inspect their contents, e.g. manifest files, javadoc, reference docs, etc, e.g.:

[[tree-drill.png]]

In the example above, clicking the 'Download' button will open the API javadocs in the browser -- a nice convenience.

You may also wish to have internal team members 'smoke test' the release, e.g. change their sample projects and dependent framework builds to point to http://repo.springsource.org/libs-staging-local and compile/test/run against the staged artifacts.

### 3. Promote the release
When verification is complete, return to the build in Bamboo from which you staged the release and click 'Artifactory'.  You'll now see 'Promotion' options as follows:

[[build-promotion.png]]

The 'Target promotion repository' is set to 'libs-milestone-local'.  Click 'Update' to move all artifacts from 'libs-staging-local' to 'libs-milestone-local'.

### 4. Merge the release branch
At this point, the release is complete and successful, so the release branch should be merged back into master, e.g.

    $ cd spring-framework              # your local spring-framework working copy
    $ git checkout master
    $ git fetch --all                  # to fetch the branch created during the release
    $ git merge springsource/master    # make sure you're up to date
    $ git merge springsource/3.1.0.M1  # assuming your remote is named 'springsource'
    $ git push springsource master:master

### 5. Announce the release!
At this point, announcements may be made and users may consume the released artifacts by adding http://repo.springsource.org/libs-milestone-local to their build scripts.

***
<a name="wiki-maven_central"/>
# What about publishing artifacts to Maven Central?

GA releases of Spring Framework are published not only to http://repo.springsource.org/libs-release-local, but also to Maven Central at http://repo1.maven.org.  This allows for maximum convenience for the majority of Spring users, given that most users have Maven-based builds and Maven resolves artifacts by default from Maven Central.

The preferred way of releasing artifacts to Maven Central is via Sonatype's Nexus server at oss.sonatype.org (OSO). This is explained in detail in [Sonatype's OSS usage guide](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide).

The SpringSource Artifactory repository has been customized with a "[nexus-push](https://github.com/JFrogDev/artifactory-user-plugins/tree/master/nexus-push)" plugin that allows for automatic propagation of builds from Artifactory to the Nexus server at OSO for publication into Maven Central.

All Spring projects -- that is, all projects having groupid `org.springframework` -- can publish to OSO under the shared 'springsource' account.  This has already been set up in the nexus-push plugin, so there's no additional setup necessary at OSO, even for new projects.

The Artifactory Bamboo plugin supports use of the nexus-push plugin through it's UI.  Step 3 of the the FAQ entry above on publishing releases described the process for promoting a build out of staging.  If the build is a GA release, simply choose the 'Push to Nexus' option, and select 'libs-release-local' as the target repository:

[[push-to-nexus.png]] 

Choosing this option means that the build will first be published into a staging repository at OSO and 'closed' in Nexus terminology.  The 'closing' process will check your build artifacts to ensure they meet the [requirements for publication into Maven Central](https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide#SonatypeOSSMavenRepositoryUsageGuide-6.CentralSyncRequirement), e.g. that POMs are properly formed, that all artifacts have checksums, PGP signatures, etc.  If there are any errors in the repository closing process, they will be displayed in the Bamboo UI and the promotion process will fail.  At this point you'll need to correct the issues and walk through the release staging process described above once again.

Note that with regard to requirements for OSO onboarding, Artifactory automatically generates sha1 and md5 checksums for all artifacts, so you don't need to worry about this. Furthermore, a [custom plugin](https://github.com/JFrogDev/artifactory-user-plugins/blob/master/pgp-sign/pgpSign.groovy) has been developed for repo.springsource.org that adds PGP signatures (.asc files) on the fly during upload using the buildmaster@springframework.org PGP key.  You simply need to make sure that your build produces jars (including -sources and -javadoc jars) and well-formed poms.  Any zip files such as distribution or doc zips are excluded from the promotion process to OSO.

When the promotion process is complete, i.e. closing the staging repository at OSO succeeds, there is one additional step - you must log into http://oss.sonatype.org with the 'springsource' account and manually 'release' to Maven Central.

1. Go to http://oss.sonatype.org and click 'log in'
1. username: springsource; password: [on request](mailto:buildmaster@springframework.org)
1. Click on "Staging Repositories".
1. You will see your closed staging repository there; click to select it.
1. Click the 'Release' button. It looks something like this:

![nexus-release-button](https://docs.sonatype.org/download/attachments/6619284/staging_release.png?version=1&modificationDate=1279720544670)

When you're prompted for a description, you can leave it blank.

Pressing the release button means that your artifacts will be published into Maven Central, but beware -- there's no going back after this point!

**Synchronization to Maven Central should be complete within three hours of pressing the 'release' button**.  In practice, it is usually two hours or less, but with just the right timing (or _wrong_ timing as the case may be), you may need to wait the full three.  If your artifacts do not show up at http://search.maven.org within four hours, email buildmaster@springframework.org and ask about escalation to Sonatype.

***
<a name="wiki-docs_schema_dist_publication"/>
# How are docs and schemas and distribution zips published?
First, you'll find three important tasks in the root build.gradle script: `docsZip`, `schemaZip`, and `distZip`.  As you might guess, these create zip files containing docs and schemas in the case of _docsZip_ and _schemaZip_ respectively; _distZip_ aggregates the contents of the first two and adds in all classes jars, -sources jars and -javadoc jars.

As described in the release process documentation above, all artifacts produced by the Spring Framework build are published into Artifactory, including these zip archives.  This provides a consistent storage mechanism, but ultimately the docs and schema zips need to be published and unpacked at http://static.springframework.org/spring-framework/docs and http://www.springframework.org/schema, respectively.  For example:

* http://static.springsource.org/spring-framework/docs/3.1.0.RELEASE (changelog, api javadocs, reference docs)
* http://www.springframework.org/schema/context (xsd files for spring-context)

Performing these uploads directly from the build script is problematic.  It requires the build to use SSH libraries in order to SCP and unpack the zip files, which is already complex, but worse it requires that the operator of the build script has the correct SSH key authentication configured on the remote servers, and that they are within the VMware VPN.

To avoid this complexity, a separate process called 'autorepo' runs periodically (every 20 minutes), querying Artifactory for these for docs and schema zips.  When new ones are found, the autorepo process does the heavy SSH lifting to upload and unpack them at the sites mentioned above.  This script is currently under development, but you can see the results of the prototype effort at http://static.springsource.org/autorepo/.

In order for autorepo to work properly, these artifacts must be annotated in Artifactory with custom metadata.  This metadata is attached to the artifacts on upload by the Artifactory Bamboo plugin.  Here's the configuration in the Spring Framework 3.2.x build plan:

[[artifact-properties.png]]

And here are those properties in copy-pastable form:

```
archives *:*:*:*@zip zip.name:spring-framework, zip.displayname:Spring Framework, zip.deployed:false
archives *:*:*:docs@zip zip.type:docs
archives *:*:*:dist@zip zip.type:dist
archives *:*:*:schema@zip zip.type:schema
```

_Note: Full documentation for this feature can be found in the [Artifactory Gradle plugin documentation](http://wiki.jfrog.org/confluence/display/RTF/Gradle+Artifactory+Plugin#GradleArtifactoryPlugin-ThePropertiesClosureDSL)._

The `zip.type` property tells autorepo that the artifact is a 'docs', 'schema', or 'dist'.  The `zip.deployed` property tells autorepo whether it has already uploaded and unpacked this artifact.  When autorepo detects a new docs or schema zip (zip.deployed == false), it performs the uploading and unpacking, and then sets the `zip.deployed` property to `true`.

The dist zip, on the other hand, remains within Artifactory and the SpringSource [community download page](http://www.springsource.org/download/community) queries repo.springsource.org to provide the list of dist zip downloads for each project.  It uses the same metadata mentioned above to perform the search.

The `zip.displayname` value is used by autorepo to determine where in the community download page the artifact should show up.  So this value should match whatever name your project already has on the community download page.

Once autorepo is working, you may want to see [[autorepo version updating]]
