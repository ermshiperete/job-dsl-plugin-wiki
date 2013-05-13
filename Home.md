Welcome to the jenkins-job-dsl wiki!

Highly recommended starting point is the tutorial, [[Tutorial: Using the Jenkins Job DSL]]

Once you know how to create a "seed" job from the tutorial, start looking at the [[Real World Examples]] for examples to steal from.  For formal documentation, the [[Job DSL Commands]] page has what is available directly in the DSL at this time, and there are also some [[User Power Moves]] you can try to make your life easier.

If you want to get fancy you'll want to read up on the _configure_ block which gives you direct access to the config.xml, read [[configure block]].

And finally, if you want to get more involved, [[here's how... | Contributing to the job dsl plugin Project]]

## Release Notes

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