---
title: Maven Build
---

## The Basics

The full build requires the following software to be installed:

* Maven (v3.1+)
* Java (v1.7+, 1.8 recommended)
* Go (v1.6+) [if building the CLI client]
* rpm tools [if building the dist packages for those platforms]

With these in place, you should be able to build everything with a:

```bash
mvn clean install
```

Alternatively you can build most things with just Java and Maven installed using:

```bash
mvn clean install -Dno-go-client -Dno-rpm
```

Other tips:

* Add `-DskipTests` to skip tests (builds much faster, but not as safe)

* You may need more JVM memory, e.g. at the command-line (or in `.profile`):

  `export MAVEN_OPTS="-Xmx1024m -Xms512m"`

* Run `-PIntegration` to run integration tests, or `-PLive` to run live tests
  ([tests described here]({{book.path.docs}}/dev/code/tests.md))

* You may need to install `rpm` package to build RPM packages: `brew install rpm` for Mac OS, `apt-get install rpm` for Ubuntu, `yum install rpm` for Centos/RHEL.
  On Mac OS you may also need to set `%_tmppath /tmp` in `~/.rpmmacros`.

* If you're looking at the maven internals, note that many of the settings are inherited from parent projects (see for instance `brooklyn-server/parent/pom.xml`)

* For tips on building within various IDEs, look [here]({{book.path.docs}}/dev/env/ide/index.md).


## When the RAT Bites

We use RAT to ensure that all files are compliant to Apache standards.  Most of the time you shouldn't see it or need to know about it, but if it detects a violation, you'll get a message such as:

    [ERROR] Failed to execute goal org.apache.rat:apache-rat-plugin:0.10:check (default) on project brooklyn-parent: Too many files with unapproved license: 1 See RAT report in: /Users/alex/Data/cloudsoft/dev/gits/brooklyn/target/rat.txt -> [Help 1]

If there's a problem, see the file `rat.txt` in the `target` directory of the failed project.  (Maven will show you this link in its output.)

Often the problem is one of the following:

* You've added a file which requires the license header but doesn't have it

  **Resolution:**  Simply copy the header from another file

* You've got some temporary files which RAT things should have headers

  **Resolution:**  Move the files away, add headers, or turn off RAT (see below)

* The project structure has changed and you have stale files (e.g. in a `target` directory)

  **Resolution:**  Remove the stale files, e.g. with `git clean -df` (and if needed a `find . -name target -prune -exec rm -rf {} \;` to delete folders named `target`)

To disable RAT checking on a build, set `rat.ignoreErrors`, e.g. `mvn -Drat.ignoreErrors=true clean install`.  (But note you will need RAT to pass in order for a PR to be accepted!)

If there is a good reason that a file, pattern, or directory should be permanently ignored, that is easy to add inside the root `pom.xml`.


## Other Handy Hints

* On some **Ubuntu** (e.g. 10.4 LTS) maven v3 is not currently available from the repositories.
  Some instructions for installing at are [at superuser.com](http://superuser.com/questions/298062/how-do-i-install-maven-3).

* The **mvnf** script 
  ([get the gist here](https://gist.github.com/2241800)) 
  simplifies building selected projects, so if you just change something in `software-webapp` 
  and then want to re-run the examples you can do:
  
  `examples/simple-web-cluster% mvnf ../../{software/webapp,usage/all}` 

## Appendix: Sample Output

A healthy build will look something like the following,
including a few warnings (which we have looked into and
understand to be benign and hard to get rid of them,
although we'd love to if anyone can help!):

```bash
% mvn clean install
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] Brooklyn REST JavaScript Web GUI
[INFO] Brooklyn Server Root
[INFO] Brooklyn Parent Project
[INFO] Brooklyn Test Support Utilities
[INFO] Brooklyn Logback Includable Configuration
[INFO] Brooklyn Common Utilities

...

[WARNING] Ignoring project type war - supportedProjectTypes = [jar]

...

[WARNING] We have a duplicate org/xmlpull/v1/XmlPullParser.class in ~/.m2/repository/xpp3/xpp3_min/1.1.4c/xpp3_min-1.1.4c.jar

...

[INFO] — maven-assembly-plugin:2.3:single (build-distribution-dir) @ brooklyn-dist —
[INFO] Reading assembly descriptor: src/main/config/build-distribution-dir.xml
{% comment %}BROOKLYN_VERSION{% endcomment %}[WARNING] Cannot include project artifact: io.brooklyn:brooklyn-dist:jar:1.0.0-SNAPSHOT; it doesn't have an associated file or directory.
[INFO] Copying files to ~/repos/apache/brooklyn/usage/dist/target/brooklyn-dist
[WARNING] Assembly file: ~/repos/apache/brooklyn/usage/dist/target/brooklyn-dist is not a regular file (it may be a directory). It cannot be attached to the project build for installation or deployment.

...

[INFO] — maven-assembly-plugin:2.3:single (build-distribution-archive) @ brooklyn-dist —
[INFO] Reading assembly descriptor: src/main/config/build-distribution-archive.xml
{% comment %}BROOKLYN_VERSION{% endcomment %}[WARNING] Cannot include project artifact: io.brooklyn:brooklyn-dist:jar:1.0.0-SNAPSHOT; it doesn't have an associated file or directory.
{% comment %}BROOKLYN_VERSION{% endcomment %}[INFO] Building tar: /Users/aled/repos/apache/brooklyn/usage/dist/target/brooklyn-1.0.0-SNAPSHOT-dist.tar.gz
{% comment %}BROOKLYN_VERSION{% endcomment %}[WARNING] Cannot include project artifact: io.brooklyn:brooklyn-dist:jar:1.0.0-SNAPSHOT; it doesn't have an associated file or directory.

...

[WARNING] Don't override file /Users/aled/repos/apache/brooklyn/usage/archetypes/quickstart/target/test-classes/projects/integration-test-1/project/brooklyn-sample/src/main/resources/sample-icon.png

...

[INFO] Reactor Summary:
[INFO] 
[INFO] Brooklyn Parent Project ........................... SUCCESS [3.072s]
[INFO] Brooklyn Utilities to Support Testing (listeners etc)  SUCCESS [3.114s]
[INFO] Brooklyn Logback Includable Configuration ......... SUCCESS [0.680s]
[INFO] Brooklyn Common Utilities ......................... SUCCESS [7.263s]
[INFO] Brooklyn Groovy Utilities ......................... SUCCESS [5.193s]
[INFO] Brooklyn API ...................................... SUCCESS [2.146s]
[INFO] Brooklyn Test Support ............................. SUCCESS [2.517s]
[INFO] CAMP Server Parent Project ........................ SUCCESS [0.075s]
[INFO] CAMP Base ......................................... SUCCESS [4.079s]
[INFO] Brooklyn REST Swagger Apidoc Utilities ............ SUCCESS [1.983s]
[INFO] Brooklyn Logback Configuration .................... SUCCESS [0.625s]
[INFO] CAMP Server ....................................... SUCCESS [5.446s]
[INFO] Brooklyn Core ..................................... SUCCESS [1:24.122s]
[INFO] Brooklyn Policies ................................. SUCCESS [44.425s]
[INFO] Brooklyn Hazelcast Storage ........................ SUCCESS [7.143s]
[INFO] Brooklyn Jclouds Location Targets ................. SUCCESS [16.488s]
[INFO] Brooklyn Secure JMXMP Agent ....................... SUCCESS [8.634s]
[INFO] Brooklyn JMX RMI Agent ............................ SUCCESS [2.315s]
[INFO] Brooklyn Software Base ............................ SUCCESS [28.538s]
[INFO] Brooklyn Network Software Entities ................ SUCCESS [3.896s]
[INFO] Brooklyn OSGi Software Entities ................... SUCCESS [4.589s]
[INFO] Brooklyn Web App Software Entities ................ SUCCESS [17.484s]
[INFO] Brooklyn Messaging Software Entities .............. SUCCESS [7.106s]
[INFO] Brooklyn Database Software Entities ............... SUCCESS [5.229s]
[INFO] Brooklyn NoSQL Data Store Software Entities ....... SUCCESS [11.901s]
[INFO] Brooklyn Monitoring Software Entities ............. SUCCESS [4.027s]
[INFO] Brooklyn CAMP REST API ............................ SUCCESS [15.285s]
[INFO] Brooklyn REST API ................................. SUCCESS [4.489s]
[INFO] Brooklyn REST Server .............................. SUCCESS [30.270s]
[INFO] Brooklyn REST Client .............................. SUCCESS [7.007s]
[INFO] Brooklyn REST JavaScript Web GUI .................. SUCCESS [24.397s]
[INFO] Brooklyn Launcher ................................. SUCCESS [15.923s]
[INFO] Brooklyn Command Line Interface ................... SUCCESS [9.279s]
[INFO] Brooklyn All Things ............................... SUCCESS [13.875s]
[INFO] Brooklyn Distribution ............................. SUCCESS [49.370s]
[INFO] Brooklyn Quick-Start Project Archetype ............ SUCCESS [12.053s]
[INFO] Brooklyn Examples Aggregator Project .............. SUCCESS [0.085s]
[INFO] Brooklyn Examples Support Aggregator Project - Webapps  SUCCESS [0.053s]
[INFO] hello-world-webapp Maven Webapp ................... SUCCESS [0.751s]
[INFO] hello-world-sql-webapp Maven Webapp ............... SUCCESS [0.623s]
[INFO] Brooklyn Simple Web Cluster Example ............... SUCCESS [5.398s]
[INFO] Brooklyn Global Web Fabric Example ................ SUCCESS [3.176s]
[INFO] Brooklyn Simple Messaging Publish-Subscribe Example  SUCCESS [3.217s]
[INFO] Brooklyn NoSQL Cluster Examples ................... SUCCESS [6.790s]
[INFO] Brooklyn QA ....................................... SUCCESS [7.117s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 8:33.983s
[INFO] Finished at: Mon Jul 21 14:56:46 BST 2014
[INFO] Final Memory: 66M/554M
[INFO] ------------------------------------------------------------------------

```
