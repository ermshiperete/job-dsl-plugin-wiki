Welcome to the jenkins-job-dsl wiki!

Highly recommended starting point is the tutorial, [[Tutorial: Using the Jenkins Job DSL]]

Once you know how to create a "seed" job from the tutorial, start looking at the [[Real World Examples]] for examples to steal from.  **For formal documentation, the [[Job DSL Commands]]** page has what is available directly in the DSL at this time, and there are also some [[User Power Moves]] you can try to make your life easier.

After you get familiar with some of the commands, try them out at the [Job DSL Playground](http://job-dsl.herokuapp.com/).

If you want to get fancy you'll want to read up on the _configure_ block which gives you direct access to the config.xml, read [[configure block|The Configure Block]]. It's also possible (and easy) to [[define your own DSL commands|Extending the DSL from your Job Scripts]] using monkey patching.

There is a great load of information on [the forum](https://groups.google.com/forum/#!forum/job-dsl-plugin), but some stuff is also making its way into a [[FAQ|Frequently Asked Questions]].

And finally, if you want to get more involved, [here's how...](https://github.com/jenkinsci/job-dsl-plugin/blob/master/CONTRIBUTING.md)

## Release Notes
* 1.21 - [Build Flow](https://wiki.jenkins-ci.org/display/JENKINS/Build+Flow+Plugin) job type
* 1.21 - [Execute concurrent builds](wiki/Job-reference#wiki-execute-concurrent-builds) job option
* 1.21 - [Github Commit Notifier](wiki/Job-reference#wiki-github-commit-notifier) publisher
* 1.21 - [Build Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin)
* 1.21 - [Views](wiki/Job-DSL-Commands#wiki-view)
* 1.21 - [Publish Robot Framework reports](wiki/Job-reference#wiki-robot-framework-reports)
* 1.21 - Templates within folders can be used
* 1.21 - [Tool Environment Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Tool+Environment+Plugin)
* 1.20 - [Allow to set the Maven installation](wiki/Job-reference#maven-installation)
* 1.20 - [Maven step supports more options](wiki/Job-reference#maven-1)
* 1.20 - [Maven pre/post build steps supported](wiki/Job-reference#maven-pre-and-post-build-steps)
* 1.20 - [Conditional BuildStep Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Conditional+BuildStep+Plugin)
* 1.20 - [Added allowEmpty option to archiveArtifacts](wiki/Job-reference#archive-artifacts)
* 1.20 - [Throttle Concurrent Builds Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Throttle+Concurrent+Builds+Plugin)
* 1.20 - Some implementation classes have been moved to avoid problems with Groovy closures (**BREAKING CHANGE**, see [[Migration]])
* 1.20 - [Parameterized Trigger Plugin for build steps](https://wiki.jenkins-ci.org/display/JENKINS/Parameterized+Trigger+Plugin)
* 1.20 - [Associated Files Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Associated+Files+Plugin)
* 1.20 - [JSHint Checkstyle Plugin](https://wiki.jenkins-ci.org/display/JENKINS/JSHint+Checkstyle+Plugin)
* 1.20 - [Emma Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Emma+Plugin)
* 1.20 - [Added support for advanced Git SCM options](wiki/Job-reference#git)
* 1.20 - [Added ItemDiscover, ItemCancel, and ScmTag permissions to the Permissions enum](https://github.com/jenkinsci/job-dsl-plugin/pull/97)
* 1.19 - [Javadoc Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Javadoc+Plugin)
* 1.19 - Added support to allow seed jobs in folders. [#109](https://github.com/jenkinsci/job-dsl-plugin/pull/109)
* 1.19 - [Groovy Postbuild Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Groovy+Postbuild+Plugin)
* 1.19 - [XVNC Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Xvnc+Plugin)
* 1.19 - [Prerequisite Build Step](https://wiki.jenkins-ci.org/display/JENKINS/Prerequisite+build+step+plugin)
* 1.19 - Aggregate Downstream Test Results
* 1.19 - [Post Build Task](https://wiki.jenkins-ci.org/display/JENKINS/Post+build+task)
* 1.19 - [AnsiColor Plugin](https://wiki.jenkins-ci.org/display/JENKINS/AnsiColor+Plugin)
* 1.19 - new wrappers block, containing all build wrapper methods (**BREAKING CHANGE**, see [[Migration]])
* 1.19 - [Timestamper](https://wiki.jenkins-ci.org/display/JENKINS/Timestamper)
* 1.19 - [JENKINS-20284](https://issues.jenkins-ci.org/browse/JENKINS-20284)
* 1.19 - [Text Finder](https://wiki.jenkins-ci.org/display/JENKINS/Text-finder+Plugin)
* 1.17 - Static Analysis Plugins (PMD, Checkstyle, Findbugs, DRY, Tasks, CCM, Lint, OWASP Dependency Check, Compiler warnings, 
* 1.17 - Description Setter Plugin
* 1.17 - Keep Dependencies and Fingerprinting
* 1.17 - Mailer
* 1.17 - SSH Agent
* 1.17 - General Improvements (Filter for Run Build Parameter, support Folders)
* 1.17 - Isolated Local Maven Repositories
* 1.17 - Node Stalker Plugin
* 1.17 - JaCoCo Plugin
* 1.17 - Claim plugin
* 1.16 - Clone Workspace SCM
* 1.16 - Block on upstream/downstream jobs
* 1.16 - Job DSL Plugin (to create DSL build steps in a job)
* 1.16 - Build Timeout Plugin
* 1.16 - Cobertura Plugin
* 1.16 - Port Allocator Plugin
* 1.16 - RVM Plugin
* 1.16 - URL Trigger
* 1.16 - Better Gerrit support
* 1.16 - SBT Plugin
* 1.16 - Update blockOn call
* 1.16 - [[Help method to read files from the workspace|Job-DSL-Commands#reading-files-from-workspace]]
* 1.16 - Queue jobs after execution of the DSL
* 1.16 - MultiJob Support
* 1.15 - [[@Grab Support|Grab Support]]
* 1.15 - Build Parameters
* 1.15 - GitHub SCM Context
* 1.15 - IRC Publisher
* 1.15 - Environment Variables From Groovy Script
* 1.15 - Priority Sorter Plugin
* 1.14 - Environment Variables
* 1.14 - Groovy Build Steps
* 1.14 - System Groovy Build Steps
* 1.14 - Maven Project Support
* 1.13 - Make it possible to forget generated jobs
* 1.13 - JENKINS-16931, JENKINS-16998
* 1.12 - Copy Artifacts Plugin
* 1.12 - Batch File Build Step
* 1.12 - Violations Publisher Plugin
* 1.11 - Able to specify description
* 1.11 - Build Blocker Plugin
* 1.11 - Downstream Extended - Extended version of downstream that can also pass in complex parameters
* 1.11 - Downstream - Specify a downstream job
* 1.11 - SCP Publisher
* 1.11 - Jabber Publisher - Publish builds to Jabber
* 1.11 - Archive Junit - Archiving the results from JUnit
* 1.11 - Ant - Apache Ant Build Step
* 1.11 - logRotator - How long to keep builds
* 1.11 - publishHtml - Publish HTML Files
* 1.11 - archiveArtifacts - Archive artifacts into the build
* 1.10 - Encrypt P4 Passwords
* 1.10 - Start building onejar
* 1.9 - Fix label() to force canRoam to false
* 1.8 - Bug fixes
* 1.7 - Move to GroovyEngine, to look at Workspace for other Groovy scripts that can be used for re-usable helper classes
* 1.6 - Refactored DSL from Plugin, so that they're in different modules
* 1.5 - Fix for #39
* 1.5 - label() - Assign which labels this job can run on
* 1.5 - timeout(Integer timeoutInMinutes, boolean shouldFailBuild) - Build Timeout Plugin
* 1.5 - chucknorris() - Chuck Norris Plugin
* 1.4 - Parameters and Environment variables are binded to the script and can be used directly
* 1.4 - svn(svnUrl, closure)
* 1.4 - p4(viewspec, user, password, closure)
* 1.3 - Support for Node as argument to div(), meaning that more complex structures can be used in div statements.
* 1.3 - multiscm(closure) - Supoprt multi-scm plugin, alternative to scm tag to allow multiple SCMs
* 1.2 - permissionAll(String userName) - Add all available permissions for a user
* 1.1 - extendedEmail(recipients, subject, content, closure) - Configure email-ext plugin
* 1.1 - gerrit(closure) - Configure Gerrit Trigger plugin
* 1.0 - Initial release