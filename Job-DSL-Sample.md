Targeted supported features:
```groovy
job {
    name = "${JOBNAME}-tests"
    configure { project ->
        // Simple block, adds if it doesn't exists
        publishers.'hudson.tasks.JavadocArchiver' {
            // Simple body used in brackets
            javadocDir { 'build/javadoc' }

            // Simple asssignment of text used via equals
            title = 'Java Documentation'

            // Support native boolean and other native types.
            keepAll = true
        }

        // Updating of existing node, e.g. properties
        properties {
            // Further navigation of existing nodes
            'hudson.security.AuthorizationMatrixProperty' {
                // Add a new permission to existing Authorization Matrix
                permission { 'hudson.model.Item.Configure:jryan' } ++

                // Replace all existing permissions since we assume everything is an update
                permission { hudson.model.Run.Delete:jryan } 
            }
        }

        // Remove property from project
        canRoam--

        builders {
            'hudson.tasks.Ant' {
                targets { 'sanitize findbugs build' }
                antName { 'Ant 1.8' }
                buildFile = 'build.xml'
            } ++ // This appends the hudson.tasks.Ant node to builders
       }

       // Append a trigger, using a function
       triggers(class:'vector') + generateTrigger('*/10 * * * *')

       configureGit(project, 'git://github.com/jenkinsci/analysis-collector-plugin.git', 'origin/master')
   }
}

// Convenience Methods, examples

// Function to add trigger, generate a node inside of current context, which in this case is triggers(class:'vector')
def generateTrigger(String cron) {
    'hudson.triggers.SCMTrigger' {
        spec { cron }
     }
}

// Function to configure git, takes Node and work with it
def configureGit(project, url, branch) {
  project.with {
    scm(class:'hudson.plugins.git.GitSCM') {
        userRemoteConfigs {
            'hudson.plugins.git.UserRemoteConfig' {
                // Using bracket to indicate text of node
                name { 'origin' }

                // Using equals to set node text
                refspec = '+refs/heads/*:refs/remotes/origin/*'
                
                // Variable to method, making sure delegates work
                url { url }
            }
        }
        branches {
           'hudson.plugins.git.BranchSpec' {
               name { branch }
           }
        }

        recursiveSubmodules { false } 
     }
   }
}
```
