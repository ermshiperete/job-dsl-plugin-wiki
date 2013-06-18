# Overview

_configure_ is a method inside the Job DSL to give direct access to underlying XML of the Jenkins config.xml. The method is provided a closure to manipulate the groovy.util.Node that is passed in. All standard documentation for Node applies here, the object passed in represents the Jenkins root object <project/>. 

# Caveat

Transforming XML via Node is no fun, and quite ugly. The general groovy use-cases are to consume this structure or build it up via NodeBuilder. Use the samples below to help navigate the Jenkins XML, but keep in mind that the actual syntax is Groovy syntax: http://groovy.codehaus.org/Updating+XML+with+XmlParser. Things to keep in mind:

* All queries (methodMissing or find) assume that a NodeList is being returned, so if you need a single Node get the first element, e.g. [0]
* plus() adds siblings, so once we have one node, you can keep adding siblings, but will need an initial peer to add to.
* Children can be easily accessed if they exist, if they don't exist you have to append them. This means accessing deep trees is laborious.

# Add-ons

To ease navigation, two key operators have been overridden. Try to use them as much as possible:

* div() - finds a child node by name, always returning the first child. If no child exists, one will be created. E.g. project/description will find the description node, creating it if it doesn't exist.  (n.b. "div" stands for the division sign ("/") not the HTML element so named).  And it has a very low precedence in the order of operation, so you need to wrap parenthesis around some operations.
* leftshift() - appends as a child. If a Node is provided, it is directly added. A string is created as a node. A closure is processed like a NodeBuilder, allowing many nodes to be appended. 

# Specification

* _configure_ can be stated multiple times
* execution order is maintained
* Closure (fragments) can be passed in
* If no template is specified via using() then a simple free-style project structure is provided. 
* _configure_ blocks are run in the order they are provided with DSL commands. 

# Samples

Here is an inelegant _configure_ block which may explain what Groovy sees, e.g.
```
configure {
    // "it" is a groovy.util.Node  
    //    representing the job's config.xml's root "project" element.
    // anotherNode is also groovy.util.Node
    //    obtained with the overloaded "/" operator
    //    on which we can call "setValue(...)"
    def aNode = it
    def anotherNode = aNode / 'blockBuildWhenDownstreamBuilding'
    anotherNode.setValue("true")

    // You can chain these steps,
    //    but must add wrapping parenthesis
    //    because the "/" has a very low precedence (lower than the ".")
    (it / 'blockBuildWhenUpstreamBuilding').setValue("true")
}
```

All other samples are given in the context of a _configure_ block, e.g.

```
job {
    name = "Job-Name"
    configure { project ->
        // Put Sample here
    }
}
```

All DSL commands started as samples here. So, hint, hint, if you want a DSL command, add a working sample here. If a DSL sample is provided, that means we've written it directly into the plugin. It's highly recommended to use a DSL command is possible.

## Add permissions

_configure_:
```
def matrix = project / 'properties' / 'hudson.security.AuthorizationMatrixProperty'
matrix << {
    permission('hudson.model.Item.Configure:jill')
    permission('hudson.model.Item.Configure:jack')
}
matrix.appendNode('permission', 'hudson.model.Run.Delete:jryan')
```

Result:
```XML
<project>
  <properties>
    <hudson.security.AuthorizationMatrixProperty>
      <permission>hudson.model.Item.Configure:jill</permission>
      <permission>hudson.model.Item.Configure:jack</permission>
      <permission>hudson.model.Run.Delete:jryan</permission>
    </hudson.security.AuthorizationMatrixProperty>
  </properties>
</project>
```

DSL:
```
job {
    authorization {
        permission('hudson.model.Item.Configure:jill')
        permission('hudson.model.Item.Configure:jack')
    }
}
```

## Configure Log Rotator Plugin

Configure:
```groovy
// Doesn't take into account existing node
project << logRotator {
    daysToKeep(-1)
    numToKeep(10)
    artifactDaysToKeep(-1)
    artifactNumToKeep(-1)
}

// Alters existing value
(project / logRotator / daysToKeep) = -2
```
Result:
```XML
<project>  
  <logRotator>
    <daysToKeep>-2</daysToKeep>
    <numToKeep>10</numToKeep>
    <artifactDaysToKeep>-1</artifactDaysToKeep>
    <artifactNumToKeep>-1</artifactNumToKeep>
  </logRotator>
</project>
```

## Configure Email Notification - TBC

_configure_:
```groovy
def publisher = project/publisher/'hudson.plugins.emailext.ExtendedEmailPublisher'

Closure email() {
    trigger {
        email {
            recipientList = '$PROJECT_DEFAULT_RECIPIENTS'
            subject = '$PROJECT_DEFAULT_SUBJECT'
            body = '$PROJECT_DEFAULT_CONTENT'
            sendToDevelopers = true
            sendToRequester = false
            includeCulprits = false
            sendToRecipientList = true
        }
    }
}

publisher {
      recipientList = 'Engineering@company.com'
      configuredTriggers {
          'hudson.plugins.emailext.plugins.trigger.FailureTrigger' email()
          'hudson.plugins.emailext.plugins.trigger.FixedTrigger' email()
      }
      contentType = 'default'
      defaultSubject = '$DEFAULT_SUBJECT'
      defaultContent = '$DEFAULT_CONTENT'
}

```
Result:
```XML
  <publisher>
    <hudson.plugins.emailext.ExtendedEmailPublisher>
      <recipientList>Engineering@company.com</recipientList>
      <configuredTriggers>
        <hudson.plugins.emailext.plugins.trigger.FailureTrigger>
          <email>
            <recipientList>$PROJECT_DEFAULT_RECIPIENTS</recipientList>
            <subject>$PROJECT_DEFAULT_SUBJECT</subject>
            <body>$PROJECT_DEFAULT_CONTENT</body>
            <sendToDevelopers>true</sendToDevelopers>
            <sendToRequester>false</sendToRequester>
            <includeCulprits>false</includeCulprits>
            <sendToRecipientList>true</sendToRecipientList>
          </email>
        </hudson.plugins.emailext.plugins.trigger.FailureTrigger>
        <hudson.plugins.emailext.plugins.trigger.FixedTrigger>
          <email>
            <recipientList>$PROJECT_DEFAULT_RECIPIENTS</recipientList>
            <subject>$PROJECT_DEFAULT_SUBJECT</subject>
            <body>$PROJECT_DEFAULT_CONTENT</body>
            <sendToDevelopers>true</sendToDevelopers>
            <sendToRequester>false</sendToRequester>
            <includeCulprits>true</includeCulprits>
            <sendToRecipientList>true</sendToRecipientList>
          </email>
        </hudson.plugins.emailext.plugins.trigger.FixedTrigger>
      </configuredTriggers>
      <contentType>default</contentType>
      <defaultSubject>$DEFAULT_SUBJECT</defaultSubject>
      <defaultContent>$DEFAULT_CONTENT</defaultContent>
    </hudson.plugins.emailext.ExtendedEmailPublisher>
  </publishers>
```

## Configure Checkstyle, Findbugs and PMD - TBC

_configure_:
```groovy
```

Result:
```XML
```

## Configure Downstream Build - TBC

_configure_:
```groovy
```

Result:
```XML
```

## Configure Shell Step - TBC

_configure_:
```groovy
project / builders / 'hudson.tasks.Shell' {
    command = 'curl "http://artifacts/gradle/oss.gradle" > ${WORKSPACE}/oss.gradle'
}
```

Result:
```XML
<builders>
  <hudson.tasks.Shell>
    <command>curl "http://artifacts.netflix.com/build-gradle/netflix-oss.gradle" > ${WORKSPACE}/netflix-oss.gradle</command>
  </hudson.tasks.Shell>
</builders>
```

DSL:
```groovy
job {
  steps {
    shell 'curl "http://artifacts/gradle/oss.gradle" > ${WORKSPACE}/oss.gradle'
  }
}
```

## Configure Gradle - TBC

_configure_:
```groovy
project / builders / 'hudson.plugins.gradle.Gradle' {
    description ''
    switches '-Dtiming-multiple=5'
    tasks 'test'
    rootBuildScriptDir ''
    buildFile ''
    useWrapper 'true'
    wrapperScript 'gradlew'
}
```

Result:
```XML
<builders>
  <hudson.plugins.gradle.Gradle>
    <description/>
    <switches>-Dtiming-multiple=5</switches>
    <tasks>clean${Task}</tasks>
    <rootBuildScriptDir/>
    <buildFile/>
    <useWrapper>true</useWrapper>
    <wrapperScript>gradlew</wrapperScript>
  </hudson.plugins.gradle.Gradle>
</builders>
```

DSL:
```groovy
steps {
    gradle('test', '-Dtiming-multiple-5', true) {
        wrapperScript 'gradlew'
    }
}

## Configure SVN - TBC

_configure_:
```groovy
project / scm('hudson.scm.SubversionSCM') {
    locations {
        'hudson.scm.SubversionSCM_-ModuleLocation' {
            remote 'http://svn.apache.org/repos/asf/tomcat/maven-plugin/trunk'
            local '.'
        }
    excludedRegions ''
    includedRegions ''
    excludedUsers ''
    excludedRevprop ''
    excludedCommitMessages ''
    workspaceUpdater(class: "hudson.scm.subversion.UpdateUpdater")
}
```

Result:
```XML
<scm class="hudson.scm.SubversionSCM">
    <locations>
        <hudson.scm.SubversionSCM_-ModuleLocation>
            <remote>http://svn.apache.org/repos/asf/tomcat/maven-plugin/trunk</remote>
            <local>.</local>
        </hudson.scm.SubversionSCM_-ModuleLocation>
    </locations>
    <excludedRegions/>
    <includedRegions/>
    <excludedUsers/>
    <excludedRevprop/>
    <excludedCommitMessages/>
    <workspaceUpdater class="hudson.scm.subversion.UpdateUpdater"/>
</scm>
```

## Configure GIT - TBC

_configure_:
```groovy
```

Result:
```XML

```

## Configure Matrix Job

_configure_:
```groovy
project.name = 'matrix-project'
project / axes / 'hudson.matrix.LabelAxis' {
    name 'label'
    values {
        string 'linux'
        string 'windows'
    }
}
project / executionStrategy(class: 'hudson.matrix.DefaultMatrixExecutionStrategyImpl') {
    runSequentially false
}
```

Result:
```XML
<matrix-project>
    <actions/>
    <description/>
    <keepDependencies>false</keepDependencies>
    <properties/>
    <scm class="hudson.scm.NullSCM"/>
    <canRoam>true</canRoam>
    <disabled>false</disabled>
    <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
    <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
    <triggers class="vector"/>
    <concurrentBuild>false</concurrentBuild>
    <builders/>
    <publishers/>
    <buildWrappers/>
    <axes>
        <hudson.matrix.LabelAxis>
            <name>label</name>
            <values>
                <string>linux</string>
                <string>windows</string>
            </values>
        </hudson.matrix.LabelAxis>
    </axes>
    <executionStrategy class="hudson.matrix.DefaultMatrixExecutionStrategyImpl">
        <runSequentially>false</runSequentially>
    </executionStrategy>
</matrix-project>
```

# Reusable Configure Blocks

To reuse an configure block for many jobs, the configure block can be moved to a helper class.

The following example shows how to refactor the Matrix job sample above into a reusable helper class.

```Groovy
package helpers

class MatrixProjectHelper {
    static Closure matrixProject(Iterable<String> labels) {
        return { project ->
            project.name = 'matrix-project'
            project / axes / 'hudson.matrix.LabelAxis' {
                name 'label'
                values {
                     labels.each { string it }
                }
            }
            project / executionStrategy(class: 'hudson.matrix.DefaultMatrixExecutionStrategyImpl') {
                runSequentially false
            }
        }
    }
}
```

In the job script you can then import the helper method and use it create several matrix jobs.

```Groovy
import static helpers.MatrixProjectHelper.matrixProject

job {
    name 'matrix-job-A'
    configure matrixProject(['label-1', 'label-2'])
}

job {
    name 'matrix-job-B'
    configure matrixProject(['label-3', 'label-4'])
}
```