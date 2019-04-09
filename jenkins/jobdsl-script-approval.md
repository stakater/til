# Job DSL

Job DSL can be used to automatically create Jenkins Jobs.

## Script approval

When we run jobsl scripts from a pipeline, it asks for script approval for every jobdsl script. And if you are creating multiple jobs through it, it will ask for every job to approve it.
So to prevent it and run jobdsl without approvals, Go to `Manage Jenkins -> Configure Global Security -> CSRF Protection` and uncheck `Enable script security for Job DSL scripts`.

## Important points

- Every Jenkins has a different set of plugins installed and some of them are supported by the Job DSL plugin. So to check configurable values for your custom Jenkins go to a url like <http://{JENKINS-URL}/plugin/job-dsl/api-viewer/index.html>.
- If the plugin is not supported then you can configure a pipeline using the UI, check its generated config.xml file which is in the `jobs/job-name` folder and then use Job DSL's configure function to update the generated XML file. e.g for a file like

  ```xml
  <?xml version='1.1' encoding='UTF-8'?>
  <org.jenkinsci.plugins.workflow.multibranch.WorkflowMultiBranchProject plugin="workflow-multibranch@2.20">
    ...
    ...
    <sources class="jenkins.branch.MultiBranchProject$BranchSourceList" plugin="branch-api@2.0.20">
      <data>
        <jenkins.branch.BranchSource>
          <source class="com.cloudbees.jenkins.plugins.bitbucket.BitbucketSCMSource" plugin="cloudbees-bitbucket-branch-source@2.2.15">
            <traits>
              <com.cloudbees.jenkins.plugins.bitbucket.BranchDiscoveryTrait>
                <strategyId>1</strategyId>
              </com.cloudbees.jenkins.plugins.bitbucket.BranchDiscoveryTrait>
            </traits>
          </source>
        </jenkins.branch.BranchSource>
      </data>
    </sources>
  </org.jenkinsci.plugins.workflow.multibranch.WorkflowMultiBranchProject>
  ```

  you can notice that to specify value for `Branch Discovery` drop down, you have to add a block like

  ```xml
  <com.cloudbees.jenkins.plugins.bitbucket.BranchDiscoveryTrait>
    <strategyId>1</strategyId>
  </com.cloudbees.jenkins.plugins.bitbucket.BranchDiscoveryTrait>
  ```
  
  in `'jenkins.branch.OrganizationFolder' -> 'navigators' -> 'com.cloudbees.jenkins.plugins.bitbucket.BitbucketSCMNavigator' -> 'traits'` block. To do this, you can pass the following to configure method in Job DSL script

  ```groovy
  configure { node ->
    // node represents <jenkins.branch.OrganizationFolder>
    def traits = node / 'navigators' / 'com.cloudbees.jenkins.plugins.bitbucket.BitbucketSCMNavigator' / 'traits'
    traits << 'com.cloudbees.jenkins.plugins.bitbucket.BranchDiscoveryTrait' {
        strategyId(1) // Exclude branches that are also filed as PR
    }
  }
  ```
