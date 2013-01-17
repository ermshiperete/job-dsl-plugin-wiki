We are always happy for folks to help us out on this project.  Here's the ways you can help us out:

* Documentation
    * Small Changes - fixes to spelling, inaccuracies, errors - Just do it
    * Larger Changes - Whole new sections / pages - send a mail to the group with your proposal at https://groups.google.com/forum/?fromgroups#!forum/job-dsl-plugin
* New Features
    * Feature Requests - Send a mail to the group with your request at https://groups.google.com/forum/?fromgroups#!forum/job-dsl-plugin - if you can include an example snippet of the config.xml so much the better. 
    * Even better than this is an "New Feature" issue on the [[Jenkins JIRA|https://issues.jenkins-ci.org/secure/Dashboard.jspa]]. Remember and add the "job-dsl-plugin" component. Then, send us a mail to the newsgroup telling us about it.
    * Feature Implementations - Even better than both of these is an implementation.  Simply fork our repo, create a branch (named after the JIRA "New Feature" you created earlier), implement it yourself and submit a Pull Request.  Get started with the (light touch) [[Architecture Docs|Jenkins Job DSL Architecture]] and  [["How to Build"|Building the Jenkins Job DSL]] notes and the "Our Git Protocol" section below
* Bugs
    * Bug Reports - Send a mail to the group with your request at https://groups.google.com/forum/?fromgroups#!forum/job-dsl-plugin - The more information the better
    * Even better than this is a new "Bug" issue on the [[Jenkins JIRA|https://issues.jenkins-ci.org/secure/Dashboard.jspa]]. Remember and add the "job-dsl-plugin" component. Then, send us a mail to the newsgroup telling us about it.
    * Bug Fixes - Even better than both of these is a fix.   Simply fork our repo, create a branch (named after the JIRA "Bug" you created earlier), implement it yourself and submit a Pull Request.  Remember to follow the "Our Git Protocol" section below

## Project Developer Docs
_Not required for users of the DSL. And like any developer docs, they're probably out of date the moment they're written._

## Our Git Protocol
If you want to make a change to the code on jenkinsci/job-dsl-plugin, here's the protocol we follow (you need a github account in order to do this):

1. Fork the jenkinsci/job-dsl-plugin repository to your account
2. On your local machine, clone your copy of the job-dsl-plugin repo
3. Again on your local machine, create a branch, ideally named after a JIRA issue you're created for the work
4. Switch to the local branch and make your changes.  Commit them as you go,m and when you're happy, push them to your repo branch
5. Then, on the github website, find the branch you created for your work, and submit a Pull Request.  This will then poke us and we'll take a look at it.
6. If things are all good, we'll merge the Pull Request straight away.  We might ask you to rebase (if the trunk has moved on and there are some conflicts) or we might suggest some more changes.