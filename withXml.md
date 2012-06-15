# Overview

withXml is a method inside the Job DSL to give direct access to underlying XML of the Jenkins config.xml. The method is provided a closure to manipulate the groovy.util.Node that is passed in. All standard documentation for Node applies here, the object passed in represents the Jenkins root object<project/>. 

# Caveat

Transforming XML via Node is no fun, and quite ugly. The general groovy use-cases are to consume this structure or build it up via NodeBuilder. Use the samples below to help navigate the Jenkins XML, but keep in mind that the actual syntax is Groovy syntax: http://groovy.codehaus.org/Updating+XML+with+XmlParser. Things to keep in mind:

* All queries (methodMissing or find) assume that a NodeList is being returned, so if you need a single Node get the first element, e.g. [0]
* plus() adds siblings, so once we have one node, you can keep adding siblings, but will need an initial peer to add to.
* Children can be easily accessed if they exist, if they don't exist you have to append them. This means accessing deep trees

# Add-ons

To ease navigation, two key operators have been overridden. Try to use them as much as possible:

* div() - finds a child node by name, always returning the first child. If no child exists, one will be created
* leftshift() - appends as a child. If a Node is provided, it is directly added. A string is created as a node. A closure is processed like a NodeBuilder, allowing many nodes to be appended.

# Specification

* withXml can be stated multiple times
* execution order is maintained
* Closure (fragments) can be passed in
* If no template is specified via using() then a simple project structure is provided. 
* withXml blocks are run before any DSL commands are run. 

# Samples

All samples are given in the context of a withXml block, e.g.

```
job {
    name = "Job-Name"
    withXml { project ->
        // Put Sample here
    }
}
```

## Add permissions

withXml:
```
def matrix = project / builders / 'hudson.security.AuthorizationMatrixProperty'
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

## Configure Log Rotator Plugin

withXml:
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

withXml:
```groovy
```
Result:
```XML

```

## Configure Checkstyle, Findbugs and PMD - TBC

withXml:
```groovy
```
Result:
```XML

```

## Configure

withXml:
```groovy
```
Result:
```XML

```

## Configure Downstream Build - TBC

withXml:
```groovy
```
Result:
```XML

```


## Configure SVN - TBC

withXml:
```groovy
```
Result:
```XML

```