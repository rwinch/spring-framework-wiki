_This document explains the purpose, nature, and best practices for use of the SpringSource repository at  http://repo.springsource.org. For basic instructions on downloading Spring artifacts manually or using Maven and other build systems, see [[downloading Spring artifacts]]._

### Table of Contents
* [For the impatient](#wiki-impatient)
* [What is the SpringSource repository?](#wiki-what_is_repo)
* [What repositories are available?](#wiki-available_repositories)
* [Can I resolve EBR artifacts via repo.springsource.org?](#wiki-ebr)
* [Will artifacts still be published to Maven Central?](#wiki-maven_central)
* [What about existing artifacts published to S3 / maven.springsource.org?](#wiki-s3)
* [The artifact(s) I need are not available via the SpringSource repository. How do I get them added?](#wiki-add_repository)
* [What are the benefits of the SpringSource repository?](#wiki-benefits)

***
<a name="wiki-impatient"/>
# For the impatient
* End user application builds should use http://repo.springsource.org/release, /milestone and /snapshot to resolve Spring project artifacts. [details](#wiki-available_repositories)
* Spring project builds should use http://repo.springsource.org/libs-release, /libs-milestone, and /libs-snapshot to resolve Spring project artifacts and all transitive dependencies. [details](#wiki-available_repositories)
* Older-style http://maven.springsource.org URLs will continue to work, but switch to repo.springsource.org for new development. [details](#wiki-s3)
* GA Spring project artifacts will continue to be available via Maven Central. [details](#wiki-maven_central)

***
<a name="wiki-what_is_repo"/>
# What is the SpringSource repository?
The SpringSource repository is an [Artifactory](http://www.jfrog.com) instance hosted by JFrog on behalf of SpringSource at http://repo.springsource.org.  Artifactory is a sophisticated _repository manager_ and it is used to host all SpringSource open source project artifacts as well as their transitive dependencies.  It can be browsed and searched directly via the web at http://repo.springsource.org, and accessed by Maven, Gradle and Ivy as well as any other build system capable working against Maven/Ivy repository layouts.

Spring project teams use repo.springsource.org as a single source for resolving all project dependencies and storing all project artifacts.  Users of Spring projects can use repo.springsource.org to resolve Spring artifacts and their dependencies.

***
<a name="wiki-available_repositories"/>
# What repositories are available?
repo.springsource.org hosts three categories of repositories: _local_, _remote_ and _virtual_.  The complete list of repositories can be browsed [here](http://repo.springsource.org/webapp/simplebrowserroot.html); below we describe the most important among these these.

**Local repositories** typically host artifacts produced directly by Spring projects.  The three main local repositories are:

* [libs-snapshot-local](http://repo.springsource.org/webapp/browserepo.html?pathId=libs-snapshot-local%3A): BUILD-SNAPSHOT (development) versions
* [libs-milestone-local](http://repo.springsource.org/webapp/browserepo.html?pathId=libs-milestone-local%3A): M* (milestone) and RC (release candidate) versions
* [libs-release-local](http://repo.springsource.org/webapp/browserepo.html?pathId=libs-release-local%3A): RELEASE (generally available) versions

_Note: see [[spring artifact versioning|Downloading Spring artifacts#wiki-artifact_versioning]] for more information_

There also exist [plugins-snapshot-local](http://repo.springsource.org/webapp/browserepo.html?pathId=plugins-snapshot-local%3A) and [plugins-release-local](http://repo.springsource.org/webapp/browserepo.html?pathId=plugins-release-local%3A) repositories, used for hosting custom build system plugins, e.g. the [docbook-reference-plugin](https://github.com/SpringSource/gradle-plugins).

**Remote repositories** serve as caches for artifacts hosted elsewhere, for example [Maven Central](http://repo1.maven.org) or the [JBoss repository](http://repository.jboss.org/nexus). These repositories are not typically accessed directly, but rather are _aggregated_ by virtual repositories as discussed below.

Three additional important repositories are [libs-snapshot-s3-cache](http://repo.springsource.org/webapp/browserepo.html?pathId=libs-snapshot-s3-cache%3A), [libs-milestone-s3-cache](http://repo.springsource.org/webapp/browserepo.html?pathId=libs-milestone-s3-cache%3A) and [libs-release-s3-cache](http://repo.springsource.org/webapp/browserepo.html?pathId=libs-release-s3-cache%3A).  These cache Spring project artifacts previously deployed to s3://maven.springframework.org. More on these below.

**Virtual repositories** aggregate any combination of _local_ and _remote_ repositories to provide convenient and controlled access to select sets of artifacts.  For developers building applications against Spring projects, the most important virtual repositories are:

* [snapshot](http://repo.springsource.org/snapshot) (aggregates _libs-snapshot-local_ and _libs-snapshot-s3-cache_)
* [milestone](http://repo.springsource.org/milestone) (aggregates _libs-milestone-local_ and _libs-milestone-s3-cache_)
* [release](http://repo.springsource.org/release) (aggregates _libs-milestone-local_ and _libs-milestone-s3-cache_)

Therefore, adding http://repo.springsource.org/release as a repository within your build will ensure that you have seamless access to all SpringSource GA release artifacts, whether they were originally deployed against our older S3 repository or directly against repo.springsource.org.

If you use http://maven.springframework.org URLs in your builds today, you'll be interested to know that DNS for maven.springframework.org now points to repo.springsource.org.  These old repository urls (e.g. http://maven.springframework.org/snapshot) used to pull directly from S3, but they now pull from repo.springsource.org.  There's nothing you need to change to adapt to this; you're free to keep these URLs in use (we'll preserve the DNS entry), but for clarity, we recommend switching to repo.springsource.org for any new development.

Another class of virtual repository allows for resolution not only of SpringSource-produced artifacts, but also of their transitive dependencies:

* [libs-release](http://repo.springsource.org/libs-release) (aggregates _release_, and all third-party GA _remote_ repositories)
* [libs-milestone](http://repo.springsource.org/libs-milestone) (aggregates _milestone_ and _libs-release_)
* [libs-snapshot](http://repo.springsource.org/libs-snapshot) (aggregates _snapshot_, _libs-milestone_, and all third-party snapshot _remote_ repositories)

These virtual repositories are quite powerful.  Declaring http://repo.springsource.org/libs-release in your build script ensures that you can resolve *all* Spring project artifacts and downstream dependencies, but *only* GA versions of them.

By switching from _libs-release_ to _libs-milestone_, this constraint is relaxed to include not only all GA libs and dependencies, but also any marked as Milestone or RC.

And finally, by declaring _libs-snapshot_, you open it all the way up, giving your build access to any Spring artifacts and downstream dependencies, from GA versions to development snapshots.

The point here is that one _need not add additional repository entries_ in one's build script, but should rather _choose the repository with the necessary level of stability_.  The goal is to leave Spring project build scripts with one URL for all their needs.  Keep in mind that this approach helps ensure builds perform well, too -- the fewer repositories configured at the build script level, the fewer HTTP requests that need to be issued from your machine.  Let the server do that work!

Users of Spring projects (i.e. application developers) may also choose to use these repositories, but should note that for security reasons, only authenticated users (i.e. Spring project committers) may resolve and cache new dependencies against repo.springsource.org.  Read [[this FAQ entry|Gradle-build-and-release-FAQ#wiki-authentication]] for complete details, but in short, non-authenticated users will probably not be able to resolve _every_ one of their application dependencies against repo.springsource.org, but will rather need to fall back to Maven Central or another third-party repository.

_Note: naming may change in the near future for libs-release, libs-milestone and libs-snapshot in order to better express the hierarchical and stability-oriented nature of these repositories._

***
<a name="wiki-ebr"/>
# Can I resolve EBR artifacts via the repo.springsource.org?
Certain third-party libraries, such as IBM's UOW jar or the atinject TCK jar exist *only* in the SpringSource EBR, and the Spring Framework has (optional) dependencies on several of these.  For this reason, the [ebr-maven-external](http://repo.springsource.org/webapp/browserepo.html?pathId=ebr-maven-external-cache%3A) remote repository has been added to Artifactory, and if necessary it can add be added as an additional repository to your build configuration.  Otherwise, Spring users resolving dependencies as bundles suitable for use in an OSGi container should continue to use the dedicated repository URLs as described in the [EBR FAQ](http://ebr.springsource.com/repository/app/faq#wiki-q8).

***
<a name="wiki-maven_central"/>
# Will artifacts still be published to Maven Central?
Yes. As always, GA (.RELEASE) versions of Spring Framework and all the other Spring projects will be [published to Maven Central](http://search.maven.org/#wiki-search%7Cga%7C1%7Cg%3A%22org.springframework%22) as well as repo.springsource.org. Spring project leads and committers should read [[this FAQ entry|Gradle build and release FAQ#wiki-maven_central]] for details.

***
<a name="wiki-s3"/>
# What about existing artifacts published to S3 / maven.springsource.org?
maven.springframework.org DNS now points to repo.springsource.org.  /snapshot, /milestone and /release paths map seamlessly and pick up artifacts in S3 as well as in repo.springsource.org. Read the entry above on [available repositories](#wiki-available_repositories) for details.

***
<a name="wiki-add_repository"/>
# The artifact(s) I need are not available via the SpringSource repository. How do I get them added?
If you are a project lead or committer and notice that an artifact is not available via repo.springsource.org/libs-release, /libs-snapshot, etc, you may need to request that a new repository be added to the Artifactory configuration.  Do the following:

1. First, determine that the artifact is actually missing.  Go to http://repo.springsource.org and perform various searches to ensure that the artifact definitely does not exist in any repository.
2. Determine the 'canonical home' of the artifact in question. For example, `cascading:cascading-core` lives at http://clojars.org/repo.  If you cannot find the jar in any Maven repository, and must download it manually or build from source, see the note that follows these steps.
3. Send mail to buildmaster@springframework.org.  Include the canonical repository URL as discussed in (2), and the groupId/artifactId/version/classifier of the artifact you need.  Please also mention which Spring project depends on this artifact (which will be used for documentation purposes).
4. The team will add the new repository and aggregate it as appropriate under libs-release (or libs-snapshot / libs-milestone where appropriate) and will respond letting you know when it's ready.  Turnaround time here should be quick, as this is a trivial exercise to perform.

## A note on jars unavailable in any Maven repository

Occasionally a dependency will be unavailable in any Maven repository and must be downloaded manually or built from source.  In such cases, you may upload these artifacts directly into the `ext-snapshot-local` or `ext-release-local` repositories using the "Deploy" tab available from the homepage at http://repo.springsource.org.  The wizard there will walk you through POM generation, specifying GAVC coordinates, etc.  IMPORTANT: Take care to understand the license of any artifacts you upload in this fashion.  If the license does not permit distribution, you cannot use these repositories.  `ext-private-local` exists for this purpose, and makes these artifacts available to internal users only.  Direct any questions about this process to buildmaster@springframework.org. 

***
<a name="wiki-benefits"/>
# What are the benefits of the SpringSource repository?
This is not an exhaustive list of Artifactory benefits, but rather a select list of the benefits that Spring project teams and developers using Spring projects can expect to enjoy from using repo.springsource.org.

## simpler build scripts, simpler release process
As described [above](#wiki-available_repositories), **Spring project builds need only one repository URL** to resolve all dependencies.  This means simpler configuration as well as faster builds, due to fewer HTTP requests being issued on the client.

Thanks to Artifactory's deep CI integration, **build scripts do not need to manage deployment or publication of artifacts**.  As described [[in this FAQ entry|Gradle build and release FAQ#wiki-release_process]], release management can be driven almost completely from the Bamboo UI at http://build.springsource.org.  This results in a significant reduction of custom logic in individual project builds, and also **removes the need for VPN, S3 and SSH access**, because **all communication with repo.springsource.org happens over HTTP**.

## one place for all Spring artifacts, one search box to find them
The SpringSource repository provides search over *all* Spring project artifacts -- jars (including source and javadoc), distribution zips, XSD files, and all dependencies.  No need for finding an S3 browser or visiting the various Java artifact search portals.  In addition to [quick](http://wiki.jfrog.org/confluence/display/RTF/Quick+Search) search, Artifactory also provides the ability to search the repository for artifacts containing a particular [class](http://wiki.jfrog.org/confluence/display/RTF/Class+Search), searching by groupId/artifactId/version/classifier ([GAVC](http://wiki.jfrog.org/confluence/display/RTF/GAVC+Search)), and even searching based on custom [properties](http://wiki.jfrog.org/confluence/display/RTF/Property+Search).  All of this is possible not only via the web, but via the Artifactory [REST API](http://wiki.jfrog.org/confluence/display/RTF/Artifactory%27s+REST+API) as well.  See [below](#wiki-customizability) for examples where custom properties and searching via the API help satisfy complex requirements.

## tight control over dependency management
As described [above](#wiki-available_repositories), repo.springsource.org's virtual repositories allow Spring project teams and Spring users alike to restrict or allow access to snapshot, milestone, RC, and GA versions -- and all transitive dependencies -- with a single URL.  For example, during different phases of project development, one may wish to disallow use of any snapshot dependencies, or disallow use of anything other than GA versions of dependencies.  repo.springsource.org makes this simple.

Artifactory also supports dynamically [discarding unwanted &lt;repository&gt; entries](http://wiki.jfrog.org/confluence/display/RTF/Virtual+Repositories#VirtualRepositories-MakingSureArtifactoryisYourSoleArtifactsProvider) from downstream POMs.  This means that you are guaranteed all your dependencies and transitive dependencies come from a single location -- your local build client will never begin resolving from other third party repositories unless you want it to. 

## improved build traceability and reproducibility
Again, thanks to Artifactory's CI integration with Bamboo, first-class metadata about builds and their artifacts are published to Artifactory, allowing for bi-directional linking between a particular build in Bamboo and the artifacts it published into Artifactory.  Environment variables, system statistics, build tool versions, and principal information are also published as part of this build metadata, ensuring that artifacts can be reproduced exactly if necessary.

## ability to host artifacts that do not exist or do not belong in Maven Central or other repositories
For example, a custom Gradle [docbook-reference-plugin](https://github.com/SpringSource/gradle-plugins) has been developed to generate HTML and PDF reference documentation for Spring projects. This plugin is specifically tailored to meet the needs of Spring projects and is not a general-purpose utility.  Therefore, it need not live in Maven Central or any other repository.  The [plugin-snapshot-local](http://repo.springsource.org/webapp/browserepo.html?pathId=plugins-snapshot-local%3A) and [plugin-release-local](http://repo.springsource.org/webapp/browserepo.html?pathId=plugins-release-local%3A) repositories at repo.springsource.org provide just the right kind of management for these artifacts.

Another example is hosting non-jar artifacts such as distribution, docs, and schema archives.  These could technically be stored in Maven Central, but repo.springsource.org is a more natural home.  In addition, the Artifactory UI allows for browsing into these archives in-web, and even rendering HTML within them (as in the case of Javadocs or HTML reference docs).  This is simple and convenient and intuitive.

Yet another example is hosting build system configuration files, such as Maven settings.xml or Gradle init scripts.  As described in the [gradle-init-scripts project readme](https://github.com/SpringSource/gradle-init-scripts#readme), this not only allows for a centralized location for project teams to store these files, but can also take advantage of Artifactory's support for [filtered resources](http://wiki.jfrog.org/confluence/display/RTF/Filtered+Resources), e.g. injecting usernames and passwords into these documents during download, etc.

The list here can go on.  In short, any artifact that we wish to share and make available for use by our build systems can be hosted at repo.springsource.org.  This allows a degree of flexibility not otherwise available.

## rich support for all major build tools
Artifactory provides dedicated tasks for building and publishing artifacts from Maven-, Gradle-, and Ant-based builds at both the [CI](http://wiki.jfrog.org/confluence/display/RTF/Bamboo+Artifactory+Plug-in#BambooArtifactoryPlug-in-ConfiguringMaven3%2CGradleandIvyBuilders) and [build](http://wiki.jfrog.org/confluence/display/RTF/Working+with+Maven) [script](http://wiki.jfrog.org/confluence/display/RTF/Working+with+Gradle) [levels](http://wiki.jfrog.org/confluence/display/RTF/Working+with+Ivy).  Given that the various SpringSource projects use all three of these, this allows for a common approach to CI and release management while affording individual teams the freedom to choose the tools that work best for them.

## watches and notifications
Project leads and team members can place [watches](http://wiki.jfrog.org/confluence/display/RTF/Watches) on any artifact, directory, or entire repository within repo.springsource.org in order to be notified by email when artifacts are added, changed or deleted.  This can be useful for monitoring one's own project, but also for tracking published changes to projects that ones project depends on.

<a name="wiki-customizability"/>
## customizability to complex requirements
For example, and as explained in [[this FAQ entry|Gradle build and release FAQ#wiki-docs_schema_dist_publication]], publishing Spring project documentation requires VPN and SSH access and detailed understanding of directory structures at static.springframework.org and other sites.  By tagging docs and distribution zips with [custom properties](http://wiki.jfrog.org/confluence/display/RTF/Properties), we are able to search Artifactory programmatically and with precision via its [REST API](http://wiki.jfrog.org/confluence/display/RTF/Artifactory%27s+REST+API), and let an external cron script operating within the VPN handle this 'heavy lifting'.

Another example is supporting the requirements for entry into Maven Central.  As explained in [[this FAQ entry|Gradle build and release FAQ#wiki-maven_central]], a custom Artifactory plugin ensures that checksums and PGP signatures are created automatically for each artifact on upload, while another plugin facilitates automatic propagation of builds from repo.springsource.org into oss.sonatype.org staging repositories.  All of this means less repetitive and error-prone work at the individual build-script level.