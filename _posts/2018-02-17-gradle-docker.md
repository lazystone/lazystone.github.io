---
layout: post
title:  "Dockerizing a Spring Boot with Gradle only"
date:   2018-02-17 13:15:00 +0100
categories: programming gradle docker travis
---

> The problem with all those Docker plugins is that they assume a stable Docker API. In the past Docker has several times broken backwards compatibility of its API making all Groovy/Maven plugins useless.
>
> The most sane and future proof way to use Docker, is to just delegate your commands to the actual docker executable (at least until Docker stabilizes the API)
>
> If you use any CI system (like Travis shown on the post) it is much easier to run direct docker commands (for building and pushing images) rather than handling Docker from the build system itself (in this case Gradle)
>    
>― [kkapelon](https://www.reddit.com/r/java/comments/7xoizr/dockerizing_a_spring_boot_application_with_gradle/duat9ed/)

I personally tried to use a couple of Docker plugins initially, but found that
[one](https://github.com/Transmode/gradle-docker) is dead and [another one](https://github.com/bmuschko/gradle-docker-plugin) is too difficult
to set up.

Eventually I abandoned both of them and started to use Gradle only:

```groovy
task buildAndPushDockerImage(dependsOn: build) {
    def dockerStageDir = new File(project.buildDir, "docker")
    def repoName = 'organisationName/repositoryName'
    def tagName = buildNumber()
    def jarName = "${project.name}-${version}.jar"
    doLast {
        copy {
            from jar
            into dockerStageDir
        }

        def buildFile = new File(dockerStageDir, "Dockerfile")
        buildFile.text = """\
FROM openjdk:8-jre-slim
ADD ${jarName} ${jarName}
RUN sh -c 'touch /${jarName}'
VOLUME ["/tmp"]
ENTRYPOINT ["sh", "-c", "exec java -jar \$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom /${jarName}"]
        
EXPOSE 8080
EXPOSE 8081
"""

        exec {
            workingDir = dockerStageDir
            commandLine 'docker', 'build', '--pull', '-t', "${repoName}:${tagName}", '.'
        }
        exec {
            workingDir = dockerStageDir
            commandLine 'docker', 'push', "${repoName}:${tagName}"
        }
    }
}
```

There are a couple of important points here:
 * Use `sh -c exec` in order to avoid creation of separate java process inside docker.
 * The `-Djava.security.egd` part is explained [here](http://www.thezonemanager.com/2015/07/whats-so-special-about-devurandom.html)

The `buildNumber()` is a function which generates version number.
I just use Travis' build number as a version:

```groovy
ext.buildNumber = {
    def buildNumber = System.getenv('TRAVIS_BUILD_NUMBER')
    if (buildNumber == null || buildNumber.allWhitespace) {
        buildNumber = '1.0.0-SNAPSHOT'
    }
    return buildNumber
}
project.version = buildNumber()
```

At the end - creating docker image without any plugins is not more complicated than using any of them.

**Disclaimer** - I'm not a gradle guru, so probably it can be re-written in even more compact form.
 