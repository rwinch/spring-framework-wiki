_Looking to grab snippets of Maven XML? Jump to [Resolving Spring artifacts](#resolving-spring-artifacts)_.

## Introduction

So you're ready to start compiling and running against Spring. How do you actually get Spring jars (artifacts) on your classpath, then?

### Modularity and the need for dependency management

The first thing to understand is that the Spring Framework is modular in nature, made up of about 20 different jars:

    spring-aop        spring-context-support      spring-test              spring-jdbc
    spring-asm        spring-core                 spring-tx                spring-jms
    spring-aspects    spring-expression           spring-web               spring-orm
    spring-beans      spring-instrument           spring-webmvc            spring-oxm
    spring-context    spring-instrument-tomcat    spring-webmvc-portlet    spring-struts

Since the release of Spring 3, there is no longer an "über-jar" containing all Spring classes, and this is a good thing!  For example, most folks want to take advantage of the core dependency injection container (`spring-context`) but comparatively few need Spring's Portlet MVC support (`spring-webmvc-portlet`) classes hanging around on their classpath.

Of course, many of these modules are interdependent, for example `spring-context` depends on `spring-beans` which in turn depends on `spring-core`, and so on.  And while Spring has very few required external dependencies, some do exist. For example, most modules in the framework depend on the `commons-logging` API and `spring-aop` depends on the `aopalliance` API.  This means that you'll need these JARs on your classpath at runtime.

For these reasons we strongly recommend that all users take advantage of the _transitive dependency management_ capabilities of today's various modern build systems (if you're not already doing so, of course).  This isn't just for Spring's sake, but for the sanity of your overall project. Chances are you have many more dependencies than just Spring, and chances are that those libraries themselves have many dependencies.  Transitive dependency management can have its own challenges, but on balance most folks agree that it's better than manually-managed jar hell.

Most folks today choose either [Maven](http://maven.apache.org), [Gradle](http://gradle.org), or [Ant/Ivy](http://ant.apache.org/ivy) as a build system.  The good news here is that each of these respects the Maven POM (project object model) dependency descriptor format as well as the metadata and directory structure that all "Maven-compatible" repositories must implement.

Spring Framework (and all Spring-* projects, for that matter), publish their individual module jars with Maven metadata both to both the [Maven Central](http://search.maven.org) and [SpringSource](http://repo.springsource.org/) repositories.  We'll talk more about accessing these repositories in a moment, but let's first understand Spring artifact versioning.

### Spring artifact versioning

Before we go any further, let's take a moment to understand the semantics of Spring versioning:

    3.1.0.RELEASE
    | | | | - version type
    | | | --- maintenance version
    | | ----- major version
    | ------- project generation

* *Project generations* tend to be very long-lived, usually over a period of years; changes in this number indicate very significant new features and overall themes for the project, possible backward compatibility breakages, possible pruning of obsolete functionality.
* *Major versions* should be almost entirely backward-compatible within its own generation, e.g. Spring Framework 3.2 must be backward compatible with 3.1, etc.  Major versions tend to be thematically driven, introducing a cohesive set of well-tested features. Frequency of major versions may differ widely across projects; Spring Framework tends to release a major version every 1-2 years.
* *Maintenance versions* tend to be developed in parallel with the next major version, and released with greater frequency; should contain primarily bugfixes and minor improvements if any; become progressively more stable, and thus more conservative over time. For example, Spring Framework 3.1.1 may contain many bugfixes, 3.1.2 less; 3.1.3 even fewer; if a 3.1.7 or 3.1.8 are ever reached they should contain only the most critical fixes. In the meantime, the next major version (3.2.0) should be wrapping up.
* *version type* will be one of the following: BUILD-SNAPSHOT, M(ILESTONE)#, RC#, or RELEASE, for example:
```
3.1.0.BUILD-SNAPSHOT - nightly snapshot of 3.1.0 development
3.1.0.M1             - first milestone release toward 3.1.0 GA
3.1.0.M2             - second milestone release toward 3.1.0 GA
3.1.0.RC1            - first release candidate toward 3.1.0 GA
3.1.0.RC2            - second release candidate toward 3.1.0 GA
3.1.0.RELEASE        - final GA (generally available) release of 3.1.0
3.1.1.BUILD-SNAPSHOT - nightly snapshot of the 3.1.1 maintenance release
3.1.1.RELEASE        - final GA release of 3.1.1 
```

_Snapshot_ builds are by nature, unstable and subject to dramatic change; _milestones_ usually deliver several significant features, many improvements and bug fixes, and are stable enough for wide testing by Spring users; a _release candidate_ indicates that feature development is complete, and that the release is in a bugfix-only phase; GA (_RELEASE_) versions indicate that the version is complete, stable, and ready for production use.

The good news is that _all_ Spring projects adhere to this version scheme, so once you understand it, it applies everywhere.

<a name="resolving-spring-artifacts"/>
## Resolving Spring artifacts

### Via Maven Central
As mentioned above, all GA (_RELEASE_) versions of Spring Framework artifacts are published to Maven Central. For example, [here](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-context%22) is the listing from the Maven Central search of all GA releases of the `spring-context` module.

If you are using Maven, the Central repository is always automatically searched, so you need only to add a `<dependency>` entry to your project's POM:
```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>3.1.0.RELEASE</version>
    </dependency>
```

Notice that the `groupId` value is `org.springframework`. This is true for all Spring Framework modules.  Other Spring projects such as Spring Batch may further qualify the groupId, e.g. `org.springframework.batch`.

### Via the SpringSource repository
RC, Milestone and Snapshot versions are published to the [SpringSource repository](http://repo.springsource.org). In addition to being published to Maven Central, GA releases are published to the SpringSource repository as well.

#### Snapshots
The following configuration will resolve the latest `spring-context` 3.1.0.BUILD-SNAPSHOT:
```xml
<repository>
    <id>repository.springsource.snapshot</id>
    <name>SpringSource Snapshot Repository</name>
    <url>http://repo.springsource.org/snapshot</url>
</repository>
...
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>3.1.0.BUILD-SNAPSHOT</version>
</dependency>
```

#### Milestones and RCs
The following configuration will resolve `spring-context` 3.1.0.M2:
```xml
<repository>
    <id>repository.springsource.milestone</id>
    <name>SpringSource Milestone Repository</name>
    <url>http://repo.springsource.org/milestone</url>
</repository>
...
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>3.1.0.M2</version>
</dependency>
```

And the following will resolve `spring-context` 3.1.0.RC1:
```xml
<repository>
    <id>repository.springsource.milestone</id>
    <name>SpringSource Milestone Repository</name>
    <url>http://repo.springsource.org/milestone</url>
</repository>
...
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>3.1.0.RC1</version>
</dependency>
```
#### GA releases
The following configuration will resolve `spring-context` 3.1.0.RELEASE:
```xml
<repository>
    <id>repository.springsource.release</id>
    <name>SpringSource GA Repository</name>
    <url>http://repo.springsource.org/release</url>
</repository>
...
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
```

## Manually downloading Spring distributions
If for whatever reason you are not using a build system with dependency management capabilities, you can download Spring Framework _distribution zips_ from the SpringSource [community download page](http://www.springsource.com/download/community?project=Spring%20Framework). These distributions contain all source and binary jar files, as well as Javadoc and reference documentation, but _do not_ contain external dependencies!  However, if you build from source, you can create the `-with-dependencies.zip` locally. See the "To build a distribution" section of [Building from source] for details.
