# Overview

A closure is passed to withXml to transform the Jenkins configuration XML. This provides very low level access to the underlying XML using groovy.util.Node. All standard documentation for Node applies here, the delegate and object passed in is a Node representing the Jenkins root object, <project/>. Transforming XML via Node is no fun, and quite ugly. The general groovy use-cases are to consume this structure or build it up (via NodeBuilder). Use the samples below to help navigate the Jenkins XML, but keep in mind that the actual syntax is Groovy syntax: http://groovy.codehaus.org/Updating+XML+with+XmlParser. Things to keep in mind:

* All queries (methodMissing or find) assume that a NodeList is being returned, so if you need a single Node get the first element, e.g. [0]
* plus() adds siblings, so once we have one node, you can keep adding siblings, but will need an initial peer to add to.
* Children can be easily accessed if they exist, if they don't exist you have to append them. This means accessing deep trees

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
    withXml {
        // Put Sample here
    }
}
```

## Add permissions

withXml:
```
// Ideal: def matrix = properties.'hudson.security.AuthorizationMatrixProperty' + { permission('hudson.model.Item.Configure:jryan') }
def props = (project.properties?:project.appendNode('properties'))[0]
def matrix = props.'hudson.security.AuthorizationMatrixProperty'?:props.appendNode('hudson.security.AuthorizationMatrixProperty')
matrix.appendNode('permission', 'hudson.model.Item.Configure:jryan') + {
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
      <permission>hudson.model.Item.Configure:jryan</permission>
      <permission>hudson.model.Item.Configure:jill</permission>
      <permission>hudson.model.Item.Configure:jack</permission>
      <permission>hudson.model.Run.Delete:jryan</permission>
    </hudson.security.AuthorizationMatrixProperty>
  </properties>
</project>
```

## Configurate Log Rotator Plugin

withXml:
```groovy
// This doesn't take into account an existing node
NodeBuilder b = NodeBuilder.newInstance();
project.append (b.logRotator {
    daysToKeep(-1)
    numToKeep(10)
    artifactDaysToKeep(-1)
    artifactNumToKeep(-1)
})
```
Result:
```XML
<project>  
  <logRotator>
    <daysToKeep>-1</daysToKeep>
    <numToKeep>10</numToKeep>
    <artifactDaysToKeep>-1</artifactDaysToKeep>
    <artifactNumToKeep>-1</artifactNumToKeep>
  </logRotator>
</project>
```