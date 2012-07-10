***********************************
***********************************
**In Progress as of July 10, 2012**
***********************************
***********************************



This page capture the most common use cases, of which we'll try our best to make into actual DSL methods to greatly simplify use them. Please add your own uses cases, there is a high likelihood that if you provide an sample here, it'll get made into a DSL method.

* Set Perforce View Spec
* Create template from scratch
* Maintain Jenkins jobs for each branch of a project on Github or SVN
* Set up a chain of builds - compile & unit test, integration test, static analysis; each passing the build results of the former to the next step in the chain
* Add Git Plugin 
```<scm class="hudson.plugins.git.GitSCM">
    <configVersion>2</configVersion>
    <userRemoteConfigs>
      <hudson.plugins.git.UserRemoteConfig>
        <name></name>
        <refspec></refspec>
        <url></url>
      </hudson.plugins.git.UserRemoteConfig>
    </userRemoteConfigs>
    <branches>
      <hudson.plugins.git.BranchSpec>
        <name>**</name>
      </hudson.plugins.git.BranchSpec>
    </branches>
    <recursiveSubmodules>false</recursiveSubmodules>
    <doGenerateSubmoduleConfigurations>false</doGenerateSubmoduleConfigurations>
    <authorOrCommitter>false</authorOrCommitter>
    <clean>false</clean>
    <wipeOutWorkspace>false</wipeOutWorkspace>
    <pruneBranches>false</pruneBranches>
    <buildChooser class="hudson.plugins.git.util.DefaultBuildChooser"/>
    <gitTool>Default</gitTool>
    <submoduleCfg class="list"/>
    <relativeTargetDir></relativeTargetDir>
    <excludedRegions></excludedRegions>
    <excludedUsers></excludedUsers>
    <gitConfigName></gitConfigName>
    <gitConfigEmail></gitConfigEmail>
    <skipTag>true</skipTag>
    <scmName></scmName>
  </scm>
```

* Add Gradle Build Step
```XML
  <builders>
    <hudson.plugins.gradle.Gradle>
      <description></description>
      <switches></switches>
      <tasks>build</tasks>
      <rootBuildScriptDir></rootBuildScriptDir>
      <buildFile></buildFile>
      <gradleName>1.0</gradleName>
      <useWrapper>false</useWrapper>
    </hudson.plugins.gradle.Gradle>
  </builders>
```
* Maven build