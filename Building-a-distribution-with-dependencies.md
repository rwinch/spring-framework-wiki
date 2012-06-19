Although it is [strongly recommended](Downloading Spring artifacts) that Spring Framework users take advantage the _transitive dependency management_ features of build systems like Gradle, Maven and Ivy, some teams cannot use these tools.  This is usually due to corporate restrictions or working with legacy builds.  If you're in this boat, you may want to build a Spring Framework distribution zip that contains all optional and required runtime dependencies.

This kind of "with dependencies" zip is not published to [the SpringSource repository](SpringSource repository FAQ), but it is straightforward to create one manually by following the steps below. _(Note that these steps apply to Spring Framework versions 3.2 M2 and greater)_.

1. Clone the Spring Framework source tree as described in the [building from source](https://github.com/SpringSource/spring-framework/#building-from-source) section of the README.
2. Check out the tag of the version you wish to create a distribution for.  For example, if you want to create a "with dependencies" distribution of Spring Framework 3.2 GA, you would check out the `v3.2.0.RELEASE` tag:
```
$ git checkout v3.2.0.RELEASE
```
3. Run the `depsZip` gradle task
```
$ ./gradlew depsZip
```
4. Inspect and use the `-dist-with-deps` zip file<br>
When the process above is complete, you'll find the zip file at `build/distributions/spring-framework-${VERSION}-dist-with-deps.zip`. Within it, you'll find all optional and required runtime dependencies in the `deps` subfolder.

Keep in mind that _most of Spring Framework's dependencies are optional!_  While it will technically work to "dump" all of the dependencies into your WEB-INF/lib, this is certainly not recommended.  Add only the dependencies your application actually needs.  The only universally required dependency the Spring Framework has is commons-logging.  Everything else is based on which Spring module jars you're using, and often what functionality you're using within them.