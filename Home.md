Welcome to the jenkins-job-dsl wiki!

Highly recommended starting point is the tutorial, [[Tutorial: Using the Jenkins Job DSL]] 

Once you know how to create a "seed" job from the tutorial, start looking at the [[Real World Examples]] for examples to steal from.  For formal documentation, the [[Job DSL Commands]] page has what is available directly in the DSL at this time.

If you want to get fancy you'll want to read up on the _configure_ block which gives you direct access to the config.xml, read [[configure block]].

And finally, if you want to get more involved, [[here's how... | Contributing to the job dsl plugin Project]]

## Release Notes

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

## Project Developer Docs
_Not required for users of the DSL. And like any developer docs, they're probably out of date the moment they're written._

Architecture: [[Jenkins Job DSL Architecture]]

How to build: [[Building the Jenkins Job DSL]]
