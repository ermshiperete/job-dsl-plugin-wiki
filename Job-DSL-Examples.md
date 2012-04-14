```
job {
   name = "${JOBNAME}-tests"
   configure { project ->
       properties {
             'hudson.security.AuthorizationMatrixProperty' {
                add {
                    permission { 'hudson.model.Item.Configure:jryan' }
                }
                permission { hudson.model.Run.Delete:jryan } // Not sure what to do here, since parent is a list, maybe we need a << syntax
            }
       }
       assignedNode { remove() } // doing it this way, so that we first identify node, then remove
       triggers(class:'vector') {
           add { generateTrigger('*/10 * * * *') }        
       }
       configureScm(project)
   }
}

// Replaces trigger
def generateTrigger(String cron) {
        'hudson.triggers.SCMTrigger' {
            spec { cron }
         }
}

def configureScm(project) {
  project.with {
    scm(class:'hudson.plugins.git.GitSCM') {
        userRemoteConfigs {
            'hudson.plugins.git.UserRemoteConfig' {
               name { 'origin' }
               refspec { '+refs/heads/*:refs/remotes/origin/* }
               url 'git://github.com/jenkinsci/analysis-collector-plugin.git' // I'm getting tired of typing brackets
            }
        }
        branches {
           'hudson.plugins.git.BranchSpec {
               name { 'origin/master' }
           }
        }
        recursiveSubmodules { false } // Should support native boolean and other native types.
     }
   }
}
```