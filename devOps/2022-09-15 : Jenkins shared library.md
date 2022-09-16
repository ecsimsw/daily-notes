## Jenkins Pipeline Library


### Configure Jenkins pipeline library
Manage Jenkins » Configure System » Global Pipeline Libraries

Set name and default version.
Retrieval method : Modern SCM
Source code management : Git
Set project repository, Credential

![image](https://user-images.githubusercontent.com/46060746/190597219-884aa4f6-d9b0-4bef-af0d-bd0e2dae1ba3.png)

</br>

### Sample code

#### Tree
Should be placed under `src` or `vars`

```
cicd-library (submodule)
  - src
    - common
      - DockerScript.groovy
    - vars
      - Pod.groovy
```
</br>

#### sample file 

1. Global value
``` groovy
// src/vars/Pod.groovy
class Pod {
    public static def NODE_TEMPLATE = '''
         api version: v1
         kind: Pod
         // ...
     '''

    public def CPU_LIMIT_RESOURCE_SIZE = 39
 ```
 
2. Constructor and method
``` groovy
 class SlackMessageSender {

    private final def workflowScript

    SlackMessageSender(workflowScript) {
        this.workflowScript = workflowScript
    }

    def success() {
        final def env = workflowScript.env
        send('#00FF00', "Build finished successfully. Job: '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

    def failure() {
        final def env = workflowScript.env
        send('#FF0000', "Build failure. Job:'${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

    def send(color, msg) {
        workflowScript.slackSend(color: "${color}", message: "${msg}")
    }
}
```

3. How to use

Import library with @Library(${library_name})
Access variable or method with path

``` groovy
// Load library
@Library('CICD-LIBRARY') _

// Access static value in vars.Pod
def sampleVal = vars.Pod.classValue

// Access instance value in class file
def sampleVal = new vars.Pod().instanceValue

// Call method
def sampleVal = vars.Pod.classFunc()
def sampleVal = new vars.Pod().instanceFunc()

// If you want to import specific directory, use import
import vars.Pod
def sampleVal = Pod.classVlaue
```

</br>

#### Tips : How to hide git info about shared library from Jenkins job page?
system configure → shared library → uncheck Include @Library changes in job recent changes
