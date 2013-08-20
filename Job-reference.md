This is the in-depth documentation of the methods available on inside the _job_ part of the DSL.

## Name
```groovy
name(String jobName)
```

The Name of the job, **required**. This could be a static name but given the power of Groovy you should get very fancy with the these.

## Display Name
```groovy
displayName(String displayName)
```

The name to display instead of the actual job name. (Available since 1.16)

## Using
```groovy
using(String templateName)
```

Refers to a template Job to be used as the basis for this job. These are loaded before any configure blocks or DSL commands.  Template Jobs are just standard Jenkins Jobs which are used for their underlying config.xml. When they are changed, the seed job will attempt to re-run, which has the side-effect of cascading changes of the template the jobs generated from it.

## Description
```groovy
description(String desc)
```

Sets description of the job. This is a not a good way of creating a dynamic description of a job.

## Label
```groovy
label(String labelStr)
```

Label which specifies which nodes this job can run on, e.g. 'X86&&Ubuntu'

## Timeout

```groovy
timeout(String type) { //type is one of: 'absolute', 'elastic', 'likelyStuck'
    limit 15       // timeout in minutes
    percentage 200 // percentage of runtime to consider a build timed out
}
```

The timeout method enables you to define a timeout for builds. It can either be absolute (build times out after a fixed number of minutes), elastic (times out if build runs x% longer than the average build duration) or likelyStuck. (Available since 1.16)

The simplest invocation looks like this:

```groovy
timeout()
```

It defines an absolute timeout with a maximum build time of 3 minutes.

Here is an absolute timeout:

```groovy
timeout('absolute') {
    limit 60              // 60 minutes before timeout
}
```

The elastic timeout accepts two parameters: a percentage for determining builds that take longer than normal an a limit that is used if there is no average successful build duration (i.e. no jobs run or all runs failed):

```groovy
timeout('elastic') {
    limit 30        // 30 minutes default timeout (no successful builds available as reference)
    percentage 300  // Build will timeout when it take 3 time longer than the reference build duration
}
```

The likelyStuck timeout times out a build when it is likely to be stuck. Does not take extra configuration parameters.

```groovy
timeout('likelyStuck')
```

The following syntax has been available before 1.16 and will be retained for compatibility reasons:

```groovy
timeout(int timeoutInMinutes, Boolean shoudFailBuild = true)
```

Using the build timeout plugin, it can fail a build after a certain amount of time.

## Port allocation
```groovy
allocatePorts 'HTTP', '8080' // allocates two ports: one randomly assigned and accessible by env var $HTTP
                             // the second is fixed and the port allocator controls concurrent usage

allocatePorts {
  port 'HTTP'                          // random port available as $HTTP
  port '8080'                          // concurrent build execution controlled to prevent resource conflicts 
  glassfish '1234', 'user', 'password' // adds a glassfish port
  tomcat '1234', 'password'            // adds a port for tomcat
}
```

The port allocation plugin enables to allocate ports for build executions to prevent conflicts between build jobs competing for a single port number (useful for any build that needs to allocate a port like Rails,Play! web containers, etc). See the [plugin documentation|https://wiki.jenkins-ci.org/display/JENKINS/Port+Allocator+Plugin] for more details. (Available since 1.16)



## Disable

```groovy
disabled(Boolean shouldDisable)
```

Provides ability to disable a job.

## Quiet period
```groovy
quietPeriod()
quietPeriod(int seconds)
```

Defines a timespan to wait for additional events (pushes, check-ins) before triggering a build. This prevents Jenkins from starting multiple jobs for check-ins/pushes that occur almost at the same time.

If the number of seconds to wait is omitted from the call the job will be configured to wait for five seconds. If you need to wait for a different amount of time just specify the number of seconds to wait. (Available since 1.16)

## Block build
```groovy
blockOn(String projectName)
blockOn(Iterable<String> projectNames)
```

Block build if certain jobs are running, supported by the <a href="https://wiki.jenkins-ci.org/display/JENKINS/Build+Blocker+Plugin">Build Blocker Plugin</a>. If more than one name is provided to projectName, it is newline separated. Per the plugin, regular expressions can be used for the projectNames, e.g. ".*-maintenance" will match all maintenance jobs.

## Block on upstream/downstream projects
```groovy
blockOnUpstreamProjects()
blockOnDownstreamProjects()
```

Blocks the build of a project when one ore more upstream (blockOnUpstreamProjects()) or a downstream projects (blockOnDownstreamProjects()) are running. (Available since 1.16)

## Build History
```groovy
logRotator(int daysToKeepInt = -1, int numToKeepInt = -1, int artifactDaysToKeepInt = -1, int artifactNumToKeepInt = -1)
```

Sets up the number of builds to keep.

## Custom workspace
```groovy
customWorkspace(String workspacePath)
```

Defines that a project should use the given directory as a workspace instead of the default workspace location. (Available since 1.16) 

## RVM
```groovy
rvm('ruby-1.9.3')
rvm('ruby-2.0@gemset')
```

Configures the job to prepare a Ruby environment controlled by RVM for the build. Requires at least the ruby version, can take also a gemset specification to prevent side effects with other builds. (Available since 1.16)

## JDK
```groovy
jdk(String jdkStr)
```

Selects the JDK to be used for this project. The jdkStr must match the name of a JDK installation defined in the Jenkins system configuration. The default JDK will be used when the jdk method is omitted.

## Security
```groovy
permission(String)
permission(String permEnumName, String user)
permission(Permissions perm, String user)
permissionAll(String user)
```

Creates permission records. The first form adds a specific permission, e.g. 'hudson.model.Item.Workspace:authenticated', as seen in the config.xml. The second form simply breaks apart the permission from the user name, to make scripting easier. The third uses a helper Enum called Permissions to hide some of the names of permissions. It is available by importing javaposse.jobdsl.dsl.helpers.Permissions. By using the enum you get some basic type checking. A flaw with this system is that Jenkins plugins can create their own permissions, and the job-dsl plugin doesn't necessarily know abou them. The last form will take everything in the Permissions enum and gives them to the user, this method also suffers from the problem that not all permissions from every plugin are included.

Permissions looks like this:

```java
Enum Permissions {
    ItemConfigure, ItemWorkspace, ItemDelete, ItemBuild, ItemRead, ItemRelease, ItemExtendedRead
    RunDelete, RunUpdate, etc.
```

# Maven

The 'rootPOM', 'goals', 'mavenOpts', 'perModuleEmail', 'archivingDisabled' and 'runHeadless' methods can only be used in jobs with type 'maven'.

## Root POM
```groovy
rootPOM(String rootPOM)
```

To use a different 'pom.xml' in some other directory than the workspace root.

## Goals
```groovy
goals(String goals) 
```

The Maven goals to execute including other command line options. 

When specified multiple times, the goals and options will be concatenated, e.g.

```groovy
goals("clean") 
goals("install") 
goals("-DskipTests") 
```

is equivalent to

```groovy
goals("clean install -DskipTests") 
```

## MAVEN_OPTS
```groovy
mavenOpts(String mavenOpts) 
```

The JVM options to be used when starting Maven. When specified multiple times, the options will be concatenated.

## Email Per Module
```groovy
perModuleEmail(boolean shouldSendEmailPerModule)
```

Enable or disable email notifications for each Maven module.

## Disable Artifact Archiving
```groovy
archivingDisabled(boolean shouldDisableArchiving)
```

Disables automatic Maven artifact archiving. Artifact archiving is enabled by default.

## Run Headless
```groovy
runHeadless(boolean shouldRunHeadless)
```

Specifiy this to run the build in headless mode if desktop access is not required. Headless mode is not enabled by default.

## Environment Variables
```groovy
environmentVariables(Map<Object,Object> vars, Closure envClosure = null)
environmentVariables(Closure envClosure) {
    env(Object key, Object value)
    envs(Map<Object, Object> map) 
    groovy(String groovyScript)
}
```

Injects environment variables into the build. They can be provided as a Map or applied as part of a context. The optional Groovy script must return a map Java object. Requires the [EnvInject plugin](https://wiki.jenkins-ci.org/display/JENKINS/EnvInject+Plugin).

## Job Priority
```groovy
priority(int value)
```

Allows jobs waiting in the build queue to be sorted by a static priority rather than the standard FIFO. The default priority is 100. A jobs with a higher priority will be executed before jobs with a lower priority. Requires the [Priority Sorter Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Priority+Sorter+Plugin).

# Source Control

## SCM retry count

```groovy
checkoutRetryCount()
checkoutRetryCount(int times)
```

Defines the number of times the build should retry to check out from the SCM if the SCM checkout fails. 

The parameterless invocation sets a default retry count of three (3) times. To specify more (or less) retry counts pass the number of times to retry the checkout. (Available since 1.16)

## Mercurial

```groovy
hg(String url, String branch = null, Closure configure = null)
```

Add Mercurial SCM source. Will not clean by default, to change this use the configure block, e.g.

```groovy
hg('http://scm') { node ->
    node / clean('true')
}
```

## Git
```groovy
git(String url, String branch = null, Closure configure = null)
```

Add Git SCM source. The Git plugin has a lot of configurable options, which are all available in the configure block,

```groovy
git('git@git') { node -> // Is hudson.plugins.git.GitSCM
    node / gitConfigName('DSL User')
    node / gitConfigEmail('me@me.com')
}
```

## GitHub
```groovy
github(String ownerAndProject, String branch = null, String protocol = 'https', String host = 'github.com', Closure configure = null)
```

Add Git SCM source and configure the source brower to point to GitHub. The Git URL is derived from the ownerAndProject, protocol and host parameters. Valid protocols are `https`, `ssh` and `git`. Accepts the same closure as the git method described above.

```groovy
github('jenkinsci/job-dsl-plugin') { node -> // Is hudson.plugins.git.GitSCM
    node / gitConfigName('DSL User')
    node / gitConfigEmail('me@me.com')
}
```

## Subversion
```groovy
svn(String svnUrl, String localDir='.', Closure configure = null)
```

Add Subversion source. 'svnUrl' is self explanatory. 'localDir' sets the <local> tag (which is set to '.' if you leave this arg out). The Configure block is handed a hudson.scm.SubversionSCM node.

You can use the Configure block to configure additional svn nodes such as a <browser> node. First a FishEyeSVN example:

```groovy
svn('http://svn.apache.org/repos/asf/xml/crimson/trunk/') { svnNode ->
    svnNode / browser(class:'hudson.scm.browsers.FishEyeSVN') {
        url 'http://mycompany.com/fisheye/repo_name'
        rootModule 'my_root_module'
    }
}
```

and now a ViewSVN example:
```groovy
svn('http://svn.apache.org/repos/asf/xml/crimson/trunk/') { svnNode ->
    svnNode / browser(class:'hudson.scm.browsers.ViewSVN') / url << 'http://mycompany.com/viewsvn/repo_name'
}
```

For more details on using the configure block syntax, see our [dedicated page](configure-block).

**BEGIN Unreleased Feature - Documentation is a work in progress** 
```groovy
svn {
    /*
     * At least one location MUST be specified.
     * Additional locations can be specified by calling location() multiple times.
     *   svnUrl   - What to checkout from SVN.
     *   localDir - Destination directory relative to workspace.
     *              If not specified, defaults to '.'.
     */
    location(String svnUrl, String localDir = '.')

    /*
     * The checkout strategy that should be used.  This is a global setting for all
     * locations.
     *   strategy - Strategy to use. Possible values:
     *                CheckoutStrategy.Update
     *                CheckoutStrategy.Checkout
     *                CheckoutStrategy.UpdateWithClean
     *                CheckoutStrategy.UpdateWithRevert
     */
    checkoutStrategy(CheckoutStrategy strategy)

    /*
     * Add an excluded region.  Each call to excludedRegion() adds to the list of
     * excluded regions.
     * If excluded regions are configured, and Jenkins is set to poll for changes,
     * Jenkins will ignore any files and/or folders that match the specified
     * patterns when determining if a build needs to be triggered.
     *   pattern - RegEx
     */
    excludedRegion(String pattern)

    /*
     * Add a list of excluded regions.  Each call to excludedRegions() adds to the
     * list of excluded regions.
     * If excluded regions are configured, and Jenkins is set to poll for changes,
     * Jenkins will ignore any files and/or folders that match the specified
     * patterns when determining if a build needs to be triggered.
     *   patterns - RegEx
     */
    excludedRegions(Iterable<String> patterns)
    includedRegion(String pattern)
    includedRegions(Iterable<String> patterns)
    excludedUser(String pattern)
    excludedUsers(Iterable<String> patterns)
    excludedCommitMsg(String pattern)
    excludedCommitMsgs(Iterable<String> patterns)
    excludedRevProp(String revisionProperty)
}
```

**END Unreleased Feature**

## Perforce

```groovy
p4(String viewspec, String user = 'rolem', String password = '', Closure configure = null)
```

Add Perforce SCM source. The user probably has to be specified. The password will be properly encrypted. Sets p4Client to builds-${JOB_NAME}. The configure block is handed a hudson.plugins.perforce.PerforceSCM. Perforce requires a few fields to be setup, so it's very likely that the configure block will be needed, especially for things like p4Port.

```groovy
p4('//depot/Tools/build') { node ->
    node / p4Port('perforce:1666')
    node / p4Tool('/usr/bin/p4')
    node / exposeP4Passwd('false')
    node / pollOnlyOnMaster('true')
}
```

## Clone Workspace

```
cloneWorkspace(String parentProject, String criteriaArg = 'Any') 
```

Support the Clone Workspace plugin, by copy the workspace of another build. This complements another job which published their workspace.

# Triggers


Triggers block contains the available triggers.

## Cron
```groovy
cron(String cronString)
```

Triggers job based on regular intervals.

## Source Control Trigger
```groovy
scm(String cronString)
```

Polls source control for changes at regular intervals.

## Github Push Notification Trigger
```groovy
githubPush()
```

Enables the job to be started whenever a change is pushed to a github repository. Requires that Jenkins has the github plugin installed and that it is registered as service hook for the repository (also works with Github Enterprise). (Since 1.16)

## Gerrit
```groovy
gerrit {
    events(Closure eventClosure) // Free form listing of event names
    project(String projectName, List<String> branches) // Can be called multiple times
    project(String projectName, String branches) // Can be called multiple times
    buildStarted(int verified, int codeReview) //Updates the gerrit report values for the build started event
     buildSuccessful(int verified, int codeReview) //Updates the gerrit report values for the build successful event
    buildFailed(int verified, int codeReview) //Updates the gerrit report values for the build failed event 
    buildUnstable(int verified, int codeReview) //Updates the gerrit report values for the build unstable event 
    buildNotBuilt(int verified, int codeReview) //Updates the gerrit report values for the build not built event 
    configure(Closure configureClosure) // Handed com.sonyericsson.hudson.plugins.gerrit.trigger.hudsontrigger.GerritTrigger
}
```

Polls gerrit for changes. This DSL method works slightly differently by exposing most of its functionality in its own block. This is accomadating how the plugin can be pointed to multiple projects and trigger on many events. The most complex part is the events block, which takes the "short name" of an event. When looking at the raw config.xml for a Job which has Gerrit trigger, you'll see multiple class names in the triggerOnEvents element. The DSL method will take the names in the events block and prepend it with "com.sonyericsson.hudson.plugins.gerrit.trigger.hudsontrigger.events.Plugin" and append "Event", meaning that shorter names like ChangeMerged and DraftPublished can be used. Straight from the unit test:

```groovy
gerrit {
    events {
        ChangeMerged
        DraftPublished
    }
    project('reg_exp:myProject', ['ant:feature-branch', 'plain:origin/refs/mybranch'])
    project('test-project', '**')
    configure { node ->
        node / gerritBuildSuccessfulVerifiedValue << '10'
    }
}
```

## URL Trigger

The URL trigger plugin checks one or more specified URLs and starts a build when a change is detected. (Since 1.16)

Currently (v1.16) on Jenkins <= 1.509.2, the alternative syntax with the cron line as a parameter rather than inside the closure is required, because job creation throws an exception if a trigger is initialized with the default `'H/5 * * * *'` schedule. These versions of Jenkins do not understand "H" in a trigger schedule.

```groovy
urlTrigger {
  cron '* 0 * 0 *'    // set cron schedule (defaults to : 'H/5 * * * *')
  restrictLabel 'foo' // restrict execution to the specified label expression

  /* Simple configuration statements */
  url('http://www.example.com/foo/') {
    proxy true           // use Jenkins Proxy for requests
    status 404           // set the expected HTTP Response status code (default: 200)
    timeout 4000         // set the request timeout in seconds (default: 300 seconds)
    check 'status'       // check the returned status code (not checked by default) 
    check 'etag'         // check ETag header (not checked by default)
    check 'lastModified' // check last modified date of resource (not checked by default)
  }

  /* Content inspection (MD5 hash) */
  url('http://www.example.com/bar/') {
    inspection 'change' //calculate MD5 sum of URL content and on hash changes
  }

  /* Content inspection for JSON or XML content with detailed checking 
     using XPath/JSONPath */
  url('http://www.example.com/baz/') {
    inspection('json'|'xml') {              // inspect XML or JSON content type
      path '//div[@class="foo"]'            // XPath for checking XML content
      path '$.store.book[0].title'          // JSONPath expression (dot syntax) 
      path "$['store']['book'][0]['title']" // JSONPath expression (bracket syntax)
    }
  }
 
  /* Content inspection for text content with detailed checking using regular expressions */
  url('http://www.example.com/fubar/') {
    inspection('text') {    // inspect content type text
      regexp '_(foo|bar).+' // regular expression for checking content changes
    }
  }
}
```

There is an alternate syntax for specifying the cronSchedule:

```groovy
urlTrigger('* 0 * 0 *'){
  //closure same as above
}
```

The trigger can check multiple URLs and virtually all options are combinable, although not all combinations may be sensible or useful (as checking one URL for both XML and JSON/text content or checking both modification date and content changes).

More on JSON path expressions: [http://goessner.net/articles/JsonPath/]

The URL trigger is particularly useful for monitoring snapshot dependencies for non-Maven/Ivy projects like SBT:
 
```groovy
urlTrigger {
  url('http://snapshots.repository.codehaus.org/org/picocontainer/picocontainer/2.11-SNAPSHOT/maven-metadata.xml' {
    check 'etag'
    check 'lastModified'
  }
}
```

The sample above monitors the metadata file of the picocontainer 2.11-SNAPSHOT that changes whenever the snapshot changes and triggers a build of the dependent project.

## Snapshot Dependencies
```groovy
snapshotDependencies(boolean checkSnapshotDependencies)
```

When enabling the snapshot dependencies trigger, Jenkins will check the snapshot dependencies from the  '\<dependency\>', '\<plugin\>' and '\<extension\>' elements used in Maven POMs and setup a job relationship to the jobs building the snapshots. This can only be used in jobs with type 'maven'.

# Build Steps

Adds step block to contain an ordered list of build steps. Cannot be used for jobs with type 'maven'.

## Shell command
```groovy
shell(String commandStr)
```

Runs a shell command.

## Batch File
```groovy
batchFile(String commandStr)
```

Supports running a Windows batch file as a build step.

## Gradle
```groovy
gradle(String tasksArg = null, String switchesArg = null, Boolean useWrapperArg = true, Closure configure = null)
```

Runs Gradle, defaulting to the Gradle Wrapper. Configure block is handed a hudson.plugins.gradle.Gradle node.


## Maven
```groovy
maven(String targetsArg = null, String pomArg = null, Closure configure = null)
```

Runs Apache Maven. Configure block is handed hudson.tasks.Maven.

## Ant
```groovy
ant(String targetsArg = null, String buildFileArg = null, String antInstallation = '(Default)', Closure antClosure = null) {
    target(targetName)
    targets(Iterable<String> targets)
    prop(Object key, Object value)
    props(Map<String, String> map)
    buildFile(String buildFile)
    javaOpt(String opt)
    javaOpts(Iterable<String> opts)
    antInstallation(String antInstallationName)
}
```

Runs Apache Ant. Ant Closure block can be used for all configuration, and it's the only way to add Java Options and System
Properties.
* Target argument - Available as an argument or closure method, each call is cumulative. The target argument can be space or newline delimited.
* Properties - Available via closure method calls. All calls are cumulative. A Map is used to back it, which will enforce unique keys for the properties.
* Ant Installation - Available as an argument or closure method. Refers to the pull down box in the UI to select which installation of Ant to use, specify the exact string seen in the UI. The last call will be the one used.
* Build File - Available as an argument or closure method. Specifies which build.xml file to use, this should be relative to the workspace.
* Java Options - Available via closure method calls. Arguments to be passed directly to the JVM.

From the unit tests, it'll use the targets "build test publish deploy":

```groovy
steps {
    ant('build') {
        target 'test'
        targets(['publish', 'deploy'])
        prop 'logging', 'info'
        props 'test.threads': 10, 'input.status':'release'
        buildFile 'dir1/build.xml'
        javaOpt '-Xmx1g'
        javaOpts(['-Dprop2=value2', '-Dprop3=value3'])
        antInstallation 'Ant 1.8'
    }
}
```

## SBT

Executes the Scala Build Tool (SBT) as a build step. (Since 1.16)

```groovy
steps {
    sbt('SBT 0.12.3',              // name of SBT installation to use (required)
        'test',                    // actions to execute (optional)
        '-Dsbt.log.noformat=true', // additional system properties (optional)
        '-Xmx2G -Xms512M',         // JVM options (optional)
        'subproject')              // subdirectory to work in (optional)
}
```

Currently all options are available via the DSL. If new plugin versions should introduce new parameters there is the possiblilty to configure them via a configure closure:

```groovy
sbt(/*standard parameters here*/) {
    newParameter 'foo'
}
```

## DSL
```groovy
dsl {
    removeAction(String removeAction)      // one of: 'DISABLE', 'IGNORE', 'DELETE'
    external (String... dslFilenames)      // one or more filenames/-paths in the workspace containing DSLs
    text (String dslSpecification)         // direct specification of DSL as String
    ignoreExisting(boolean ignoreExisting) // flag if to ignore existing jobs
}

/* equivalent calls as parameters instead of closure */
def dsl(String scriptText, String removedJobAction = null, boolean ignoreExisting = false)
def dsl(Collection<String> externalScripts, String removedJobAction = null, boolean ignoreExisting = false)
```

The DSL build step creates a new job that in turn is able to generate other jobs. Particularly useful to generate a monitoring Job for things like feature/release branches. (Available since 1.16)

Sample definition using several DSL files:
```groovy
dsl {
    removeAction 'DISABLE'
    external 'some-dsl.groovy','some-other-dsl.groovy'
    external 'still-another-dsl.groovy'
    ignoreExisting true
}

/* same definition using parameters instead of closure */
dsl(['some-dsl.groovy','some-other-dsl.groovy','still-another-dsl.groovy'], 'DISABLE', true)
```

Another sample that specifies the DSL text directly:
```groovy
dsl {
    removeAction('DELETE')
    text '''
job {
    foo()
    bar {
        baz()
    }
} 
}

/* same definition using parameters instead of closure */
dsl('''
job {
    foo()
    bar {
        baz()
    }
}
''', 'DELETE')
```

## Copy Artifacts

```groovy
copyArtifacts(String jobName, String includeGlob, String targetPath = '', boolean flattenFiles = false, boolean optionalAllowed = false, Closure copyArtifactClosure = null) {
    upstreamBuild(boolean fallback = false) // Upstream build that triggered this job
    latestSuccessful() // Latest successful build
    latestSaved() // Latest saved build (marked "keep forever")
    permalink(String linkName) // Specified by permalink: lastBuild, lastStableBuild
    buildNumber(int buildNumber) // Specific Build
    workspace() // Copy from WORKSPACE of latest completed build
    buildParameter(String parameterName) // Specified by build parameter
}
```

Supports the Copy Artifact plugin. As per the plugin, the input glob is for files in the workspace. The methods in the closure are considered the selectors, of which only one can be used.

## Groovy
```groovy
groovyCommand(String commandStr = null, String groovyInstallation = '(Default)', Closure groovyClosure = null) {   
    groovyParam(String param)
    groovyParams(Iterable<String> params)
    scriptParam(String param)
    scriptParams(Iterable<String> params)
    prop(String key, String value)
    props(Map<String, String> map)
    javaOpt(String opt)
    javaOpts(Iterable<String> opts)
    groovyInstallation(String groovyInstallationName)
    classpath(String classpathEntry)
}
groovyScriptFile(String fileName = null, String groovyInstallation = '(Default)', Closure groovyClosure = null) {   
    groovyParam(String param)
    groovyParams(Iterable<String> params)
    scriptParam(String param)
    scriptParams(Iterable<String> params)
    prop(String key, String value)
    props(Map<String, String> map)
    javaOpt(String opt)
    javaOpts(Iterable<String> opts)
    groovyInstallation(String groovyInstallationName)
    classpath(String classpathEntry)
}
```

Runs a Groovy script which can either be passed inline ('groovyCommand' method) or by specifying a script file ('groovyScriptFile' method). The closure block can be used for all configuration. All calls are cumulative except for 'groovyInstallation' where the last call will be used.
* Groovy Parameters - Specifies arguments for the Groovy executable.
* Script Parameters - Specifies arguments for the script.
* Properties - Shortcut to define '-D' parameters.
* Groovy Installation - Also available as an argument. Refers to the pull down box in the UI to select which installation of Groovy to use, specify the exact string seen in the UI.
* Java Options - Arguments to be passed directly to the JVM.
* Classpath - Defines the classpath for the script.
 
## System Groovy Scripts
```groovy
systemGroovyCommand(String commandStr, Closure systemGroovyClosure = null) {
    binding(String name, String value)
    classpath(String classpathEntry)
}
systemGroovyScriptFile(String fileName, Closure systemGroovyClosure = null) {
    binding(String name, String value)
    classpath(String classpathEntry)
}
```

Runs a system groovy script, which is executed inside the Jenkins master. Thus it will have access to all the internal objects of Jenkins and can be used to alter the state of Jenkins. The `systemGroovyCommand` method will run an inline script and the `systemGroovyScriptFile` will execute a script file from the generated job's workspace. The closure block can be used to add variable bindings and extra classpath entries for a script. The methods in the closure block can be called multiple times to add any number of bindings or classpath entries. The Groovy plugin must be installed to use these build steps.

## Grails
```groovy
grails {
    target(String targetName)              // a single target to run
    targets(Iterable<String> targets)      // multiple targets to tun
    name(String grailsName)                // the name of the Grails installation
    grailsWorkDir(String grailsWorkDir)    // grails.work.dir system property
    projectWorkDir(String projectWorkDir)  // grails.project.work.dir system property
    projectBaseDir(String projectBaseDir)  // path to the root of the Grails project
    serverPort(String serverPort)          // server.port system property
    prop(String key, String value)         // a single system property key and value
    props(Map<String, String> map)         // a map of system property key and values
    forceUpgrade(boolean forceUpgrade)     // run 'grails upgrade --non-interactive' first
    nonInteractive(boolean nonInteractive) // append --non-interactive to all build targets
    useWrapper(boolean useWrapper)         // use Grails wrapper to execute targets
}

// additional methods
grails(String targets, Closure grailsClosure)                                    // space-separated targets
grails(String targets, boolean useWrapper = false, Closure grailsClosure = null) // space-separated targets
```

Supports the Grails plugin. Only targets field is required. To pass arguments to a particular target, surround the target and its arguments with double quotes.

# Multijob Phase

```
phase(String name, String continuationConditionArg = 'SUCCESSFUL', Closure phaseClosure = null) {
    phaseName(String phaseName)
    continuationCondition(String continuationCondition)
    job(String jobName, boolean currentJobParameters = true, boolean exposedScm = true, Closure phaseJobClosure = null)  {
        currentJobParameters(boolean currentJobParameters = true) 
        exposedScm(boolean exposedScm = true)
        boolParam(String paramName, boolean defaultValue = false)
        fileParam(String propertyFile, boolean failTriggerOnMissing = false)
        sameNode(boolean nodeParam = true) 
        matrixParam(String filter)
        subversionRevision(boolean includeUpstreamParameters = false)
        gitRevision(boolean combineQueuedCommits = false) 
        prop(Object key, Object value)
        props(Map<String, String> map)
    }
}
```

Phases allow jobs to be group together to be run in parallel, they only exist in a Multijob typed job. The name and continuationConditionArg can be set directly in the phase method or in the closure. The job method is used to list each job in the phase, and hence can be called multiple times. Each call can be further configured with the parameters which will be sent to it. The parameters are show above and documented in different parts of this page. See below for an example of multiple phases strung together:

```
job(type: Multijob) {
    steps {
        phase() {
            phaseName 'Second'
            job('JobZ') {
                fileParam('my1.properties')
                fileParam('my2.properties')
            }
        }
        phase('Third') {
            job('JobA')
            job('JobB')
            job('JobC')
        }
        phase('Fourth') {
            job('JobD', false, true) {
                boolParam('cParam', true)
                fileParam('my.properties')
                sameNode()
                matrixParam('it.name=="hello"')
                subversionRevision()
                gitRevision()
                prop('prop1', 'value1')
            }
        }
   }
}
```

# Publishers

Block to contain list of publishers.

## Extended Email Plugin
```groovy
extendedEmail(String recipients = null, String subjectTemplate = null, String contentTemplate = null, Closure emailClosure = null)
```

Supports the Extended Email plugin. DSL methods works like the gerrit plugin, providing its own block to help set it up. The emailClosure is primarily used to specify the triggers, which is optional. Its definition:

```groovy
extendedEmail {
    trigger(String triggerName, String subject = null, String body = null, String recipientList = null,
            Boolean sendToDevelopers = null, Boolean sendToRequester = null, includeCulprits = null, Boolean sendToRecipientList = null)
    trigger(Map args)
    configure(Closure configureClosure) // Handed hudson.plugins.emailext.ExtendedEmailPublisher
}
```

The first trigger method allow complete control of the email going out, and maps directly to what is seen in the config.xml of a job. The triggerName needs to be one of these values: 'PreBuild', 'StillUnstable', 'Fixed', 'Success', 'StillFailing', 'Improvement', 'Failure', 'Regression', 'Aborted', 'NotBuilt', 'FirstFailure', 'Unstable'. Those names come from classes prefix with 'hudson.plugins.emailext.plugins.trigger.' and appended with Trigger. The second form of trigger, uses the names from the first, but can be called with a Map syntax, so that values can be left out more easily. To help explain it, here an example from the unite tests:

```groovy
extendedEmail('me@halfempty.org', 'Oops', 'Something broken') {
    trigger('PreBuild')
    trigger(triggerName: 'StillUnstable', subject: 'Subject', body:'Body', recipientList:'RecipientList',
            sendToDevelopers: true, sendToRequester: true, includeCulprits: true, sendToRecipientList: false)
    configure { node ->
        node / contentType << 'html'
    }
}
```

## Mailer Tasks
```groovy
mailer(String recipients, String dontNotifyEveryUnstableBuildBoolean = false, String sendToIndividualsBoolean = false)
```

This is the default mailer task. Specify the recipients, whether to flame on unstable builds, and whether to send email to individuals who broke the build. Note the double negative in the dontNotifyEveryUnstableBuild condition. If you want notification on every unstable build, keep it false. 
Simple example:

```groovy
publishers {
    mailer('me@example.com', true, true)
}
```

## Archive Artifacts
```groovy
archiveArtifacts(String glob, String excludeGlob = null, Boolean latestOnlyBoolean = false)
```

Supports archiving artifacts with each build. Simple example:

```groovy
publishers {
    archiveArtifacts 'build/test-output/**/*.html'
}
```

## Archive JUnit
```groovy
archiveJunit(String glob, boolean retainLongStdout = false, boolean allowClaimingOfFailedTests = false, boolean publishTestAttachments = false)
```

Supports archiving JUNit results for each build.

## HTML Publisher
```groovy
publishHtml {
    report(String reportDir, String reportName = null, String reportFiles = 'index.html', Boolean keepAll = false)
    report(Map args) // same names as the method above
}

```

Provides context to add html reports to be archive. The report method can be called multiple times in the closure. Simple
example with variations on how to call the report method:

```groovy
publishers {
    publishHtml {
        report('build/test-output/*', 'Test Output')
        report 'build/coverage/*', 'Coverage Report', 'coverage.html' // Without parens
        report reportName: 'Gradle Tests', reportDir: 'test/*', keepAll: true // Map synxtax
    }
}
```

## Jabber Publisher
```groovy
publishJabber(String target, String strategyName, String channelNotificationName, Closure jabberClosure) {
    strategyName 'ALL' // ALL, FAILURE_AND_FIXED, ANY_FAILURE, STATECHANGE_ONLY
    notifyOnBuildStart false
    notifySuspects false
    notifyCulprits false
    notifyFixers false
    notifyUpstreamCommitters false
    channelNotificationName 'Default' // Default, SummaryOnly, BuildParameters, PrintFailingTests
}
```

Supports <a href="https://wiki.jenkins-ci.org/display/JENKINS/Jabber+Plugin">Jabber Plugin</a>. A few arguments can be specified in the method call or in the closure.

## SCP Publisher
```groovy
publishScp(String site, Closure scpClosure) {
    entry(String source, String destination = '', boolean keepHierarchy = false)
}
```

Supports <a href="https://wiki.jenkins-ci.org/display/JENKINS/SCP+plugin">SCP Plugin</a>. First arg, site, is specified globally by the plugin. Each entry is
individually specified in the closure block, e.g. entry can be called multiple times.

## CloneWorkspace Publisher
```groovy
publishCloneWorkspace(String workspaceGlob, String workspaceExcludeGlob = '', String criteria = 'Any', String archiveMethod = 'TAR', boolean overrideDefaultExcludes = false, Closure cloneWorkspaceClosure = null) {}
```

Supports the <a href="https://wiki.jenkins-ci.org/display/JENKINS/Clone+Workspace+SCM+Plugin">Clone Workspace SCM Plugin</a>.

Due to the simplicity of this publisher, the closure support is purely provided for creating very specific configs.  Usually the non-closure variants will suffice - the simplest purely requiring the workspaceGlob alone, and the other (equivalent to pressing the "Advanced" button in the Jenkins UI) provided all settings.

## Downstream
```groovy
downstream(String projectName, String thresholdName = 'SUCCESS')
```

Specifies a downstream job. The second arg, thresholdName, can one of four values: 'SUCCESS', 'UNSTABLE', 'FAILURE', 'ALWAYS'.

## Extended Downstream
```groovy
downstreamParameterized(Closure downstreamClosure) {
     trigger(String projects, String condition = 'SUCCESS', boolean triggerWithNoParameters = false, Closure downstreamTriggerClosure = null) {
        currentBuild() // Current build parameters
        propertiesFile(String propFile) // Parameters from properties file
        gitRevision(boolean combineQueuedCommits = false) // Pass-through Git commit that was built
        predefinedProp(String key, String value) // Predefined properties
        predefinedProps(Map<String, String> predefinedPropsMap)
        predefinedProps(String predefinedProps) // Newline separated
        matrixSubset(String groovyFilter) // Restrict matrix execution to a subset
        subversionRevision() // Subversion Revision
     }
}
```

Supports <a href="https://wiki.jenkins-ci.org/display/JENKINS/Downstream-Ext+Plugin">Downstream-Ext plugin</a>. The plugin is configured by adding triggers
to other projects, multiple triggers can be specified. The projects arg is a comma separated list of downstream projects. The condition arg is one of these
possible values: SUCCESS, UNSTABLE, UNSTABLE_OR_BETTER, UNSTABLE_OR_WORSE, FAILED.  The methods inside the downstreamTriggerClosure are optional, though it
makes the most sense to call at least one.  Each one is relatively self documenting, mapping directly to what is seen in the UI. The predefinedProp and
predefinedProps methods are used to accumulate properties, meaning that they can be called multiple times to build a superset of properties.


## Violations Plugin
```groovy
violations(int perFileDisplayLimit = 100, Closure violationsClosure = null) {
    sourcePathPattern(String sourcePathPattern)
    fauxProjectPath(String fauxProjectPath)
    perFileDisplayLimit(Integer perFileDisplayLimit)
    sourceEncoding(String encoding = 'default')
    *(Integer min = 10, Integer max = 999, Integer unstable = 999, String pattern = null)
}
```
Supports <a href="https://wiki.jenkins-ci.org/display/JENKINS/Violations">Violations Plugin</a>. The violationsClosure is crucial to configure this DSL method.
For each supported type, you specify it and a few option parameters, this is represented above with the asterisk (*). If a type isn't specified, then it uses
defaults. The following example wil

```groovy
violations(50) {
   sourcePathPattern 'source pattern'
   fauxProjectPath 'faux path'
   perFileDisplayLimit 51
   checkstyle(10, 11, 10, 'test-report/*.xml')
   findbugs(12, 13, 12)
}
```

## Chuck Norris

```groovy
chucknorris()
```

Enables the Cordell Walker plugin.

## IRC

Interface for the [IRC plugin](https://wiki.jenkins-ci.org/display/JENKINS/IRC+Plugin). All the options can be set and multiple channels can be added. The channel calls support named parameters as well. The notification settings can be disabled with passing false as a parameter. A complete example:

```groovy
irc {
  channel('#channel1', 'password1', true)
  channel(name: '#channel2', password: 'password2', notificationOnly: false)
  notifyScmCommitters()
  notifyScmCulprits()
  notifyUpstreamCommitters(false)
  notifyScmFixers()
  strategy('ALL')
  notificationMessage('SummaryOnly')
}
```

## Cobertura coverage report

Supports the [Cobertura Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Cobertura+Plugin). Only the Cobertura xml reportfile pattern is required to locate all the generated reports. Advanced options are all set to defaults.

```groovy
cobertura(xmlReportFilePattern)
```

The previous example is basically equivalent to the following more verbose one, where all the default settings are visible.

```groovy
cobertura(xmlReportFilePattern) {
  onlyStable(false)    // Include only stable builds, i.e. exclude unstable and failed ones.
  failUnhealthy(false) // Unhealthy projects will be failed.
  failUnstable(false)  // Unstable projects will be failed.
  autoUpdateHealth(false)    // Auto update threshold for health on successful build.
  autoUpdateStability(false) // Auto update threshold for stability on successful build.
  zoomCoverageChart(false)   // Zoom the coverage chart and crop area below the minimum and above the maximum coverage of the past reports.
  failNoReports(true) // Fail builds if no coverage reports have been found.
  sourceEncoding('ASCII') // Character encoding of source files
  // The following targets are added by default to check the method, line and conditional level coverage:
  methodTarget(80, 0, 0) 
  lineTarget(80, 0, 0)   
  conditionalTarget(70, 0, 0)
}
```

### More about targets

Targets are used to mark the build as healthy/unhealthy/unstable based on given tresholds. Targets can be set separately for conditional, line, method, class, file and the package level.

Targets can be tuned using the following helper methods:
```groovy
conditionalTarget(healthy, unhealthy, failing)
lineTarget(healthy, unhealthy, failing)
methodTarget(healthy, unhealthy, failing)
classTarget(healthy, unhealthy, failing)
fileTarget(healthy, unhealthy, failing)
packageTarget(healthy, unhealthy, failing)
```

Each of the 3 parameters represent a percentage treshold. They have the following meaning:
* healthy: Report health as 100% when coverage is greater than {healthy}% 
* unhealthy: Report health as 0% when coverage is less than {unhealthy}%
* failing: Mark the build as unstable when coverage is less than {failing}% 

## Allow Broken Build Claiming

```groovy
allowBrokenBuildClaiming()
```

Activates broken build claiming for the [Claim plugin](https://wiki.jenkins-ci.org/display/JENKINS/Claim+plugin).

## Jacoco

This plugin allows you to capture code coverage report from the [JaCoCo Plugin](https://wiki.jenkins-ci.org/display/JENKINS/JaCoCo+Plugin). Jenkins will generate the trend report of coverage.

```
//Shown with defaults
jacocoCodeCoverage {
    execPattern '**/target/**.exec'
    classPattern '**/classes'
    sourcePattern '**/src/main/java'
    inclusionPattern '**/*.class'
    exclusionPattern '**/*Test*'
    minimumInstructionCoverage '0'
    minimumBranchCoverage '0'
    minimumComplexityCoverage '0' 
    minimumLineCoverage '0' 
    minimumMethodCoverage '0' 
    minimumClassCoverage '0' 
    maximumInstructionCoverage '0' 
    maximumBranchCoverage '0' 
    maximumComplexityCoverage '0' 
    maximumLineCoverage '0' 
    maximumMethodCoverage '0' 
    maximumClassCoverage '0'
}
```

Simplest usage will output with the defaults above
```
jacocoCodeCoverage()
```

# Parameters
**Note: In all cases apart from File Parameter the parameterName argument can't be null or empty**
_Note: The Password Parameter is not yet supported. See https://issues.jenkins-ci.org/browse/JENKINS-18141_

## Boolean Parameter
Simplest usage (taking advantage of all defaults)
```groovy
// In this use case, the value will be "true" and the description will be ''
booleanParam("myParameterName") 
```
Simple usage (omitting the description)
```groovy
// In this use case, the value will be "false" and the description will be ''
booleanParam("myParameterName", false)
```
Full usage
```groovy
// In this use case, the value will be "false" and the description will be 'the description 
// of my parameter'
booleanParam("myParameterName", false, "The description of my parameter") 
```

## ListTags Parameter
Simplest usage (taking advantage of all defaults)
```groovy
// in this case "maxTags will be set to "all", reverseByDate and reverseByName will be set to "false", 
// and description and defaultValue xml tags will not be created at all
listTagsParam("myParameterName", "http://kenai.com/svn/myProject/tags", "^mytagsfilterregex") 
```
Simple usage (omitting the description and defaultValue)
```groovy
// in this case "maxTags will be set to "all", reverseByDate and reverseByName will be set to "true", 
// and description and defaultValue xml tags will not be created at all
listTagsParam("myParameterName", "http://kenai.com/svn/myProject/tags", "^mytagsfilterregex", true, true) 
```
Full usage (omitting the description and defaultValue)
```groovy
// in this case "maxTags will be set to "all", reverseByDate and reverseByName will be set to "true", 
// and description and defaultValue xml tags will be set as shown
listTagsParam("myParameterName", "http://kenai.com/svn/myProject/tags", "^mytagsfilterregex", true, true, "defaultValue", "description") 
```
NOTE: The second-to-last parameter needs to be a String, although you are provinding in most cases a number.  This is because the default value for defaultValue is "all".

## Choice Parameter
Simplest usage (taking advantage of default description)
```groovy
// In this case the description will be set to '' and you will have a 3-option list with 
// "option 1 (default)" as the default (because it's first, not because it says so in the String 
choiceParam("myParameterName", ["option 1 (default)", "option 2", "option 3"])
```
Full usage
```groovy
// In this case the description will be set to 'my description' and you will have a 3-option list with 
// "option 1 (default)" as the default (because it's first, not because it says so in the String 
choiceParam("myParameterName", ["option 1 (default)", "option 2", "option 3"], "my description")
```

## File Parameter
Simplest usage (taking advantage of default description)
```groovy
// In this case the description will be set to ''
fileParam("test/upload.zip")
```
Full usage
```groovy
// In this case the description will be set to 'my description'
fileParam("test/upload.zip", "my description")
```

## Run Parameter
Note: The Job Name needs to be an existing Jenkins Job, though we don't check when we generate the XML.

Simplest usage (taking advantage of default description and default filter)
```groovy
// In this case the description will be set to '' and the filter won't be set
runParam("myParameterName", "myJobName")
```
Full usage
```groovy
// In this case the description will be set to 'my description' and the filter will be set to 'SUCCESSFUL'
runParam("myParameterName", "myJobName", "my description", "SUCCESSFUL")
```

## String Parameter
Simplest usage (taking advantage of default defaultValue and default description)
```groovy
// In this case the defaultValue and description will be set to ''
stringParam("myParameterName")
```
Simple usage (taking advantage of default description)
```groovy
// In this case the description will be set to ''
stringParam("myParameterName", "my default stringParam value")
```
Full usage
```groovy
// In this case the defaultValue will be set to 'my default stringParam value' and the description 
// will be set to 'my description'
stringParam("myParameterName", "my default stringParam value", "my description")
```

## Text Parameter
Simplest usage (taking advantage of default defaultValue and default description)
```groovy
// In this case the defaultValue and description will be set to ''
textParam("myParameterName")
```
Simple usage (taking advantage of default description)
```groovy
// In this case the description will be set to ''
textParam("myParameterName", "my default textParam value")
```
Full usage
```groovy
textParam("myParameterName", "my default textParam value", "my description")
```
 