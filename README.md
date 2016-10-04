##### Table of Contents  
[Jenkins Job DSL](#jenkins-jobdsl)
[Testing](#testing)
[Bootstrapping the seed jobs](#seed-jobs)
[Creating and Removing Pipelines for Projects](#pipelines)
**[Chef Pipelines](#chef-pipelines)**
**[Apps](#apps)** 
**[.jenkins.yml (public API)](#api)**


## Jenkins Job DSL

Generate the jobs and views needed for Continuous Delivery (CD) pipelines
in Jenkins using the [Groovy DSL plugin for Jenkins Jobs](https://github.com/jenkinsci/job-dsl-plugin).

## Testing
While the Job DSL Plugin can run outside Jenkins in maven, you'll find this method
often fails for any DSL blocks that require interaction with the Jenkins API.

We currently test branches of this repo using the "canary" projects.  These canaries
are "hello, world" apps that can be promoted at will with no impact on real servers.

* [App Pipelines Canaries](https://jenkins.thomascook.io/job/dsl~canary~apps/)
* [Chef Pipeline Canaries](https://jenkins.thomascook.io/job/dsl~canary~chef/)

Another interesting project is the [Jenkins Job DSL Playground](http://job-dsl.herokuapp.com/),
a CodePen-like UI for rendering DSL into the project XML:

![image](https://cloud.githubusercontent.com/assets/3165077/5448546/6469b318-84b0-11e4-8b00-36a58342c32a.png)

## Bootstrapping the seed jobs

The `./jobs` folder contains the "seed jobs" used to execute the DSL scripts
that will generate other jobs.

The Jenkins Web UI doesn't support uploading an existing config.xml file to create
a new job.  However, the HTTP API for Jenkins can create a new job by POST'ing the
config.xml file.

Below example creates the "dsl~chef" seed job using curl:

``` shell
# set the name of the job to create in Jenkins
job='dsl~chef'

# supply your Jenkins User ID, as reported by http://jenkins.eceit.net/me/
jenkins_userid="steve-jansen"

# supply your the Jenkins API key provided at http://jenkins.eceit.net/me/configure
# you will need to click the "Show API Token" button
jenkins_api_token="0123456789abcdef"

# create/update the job
curl -i \
	-X POST \
     --user "$jenkins_userid:$jenkins_api_token" \
     -H "Content-Type: text/xml" \
     --data @"./jobs/$job.xml" \
     "http://jenkins.eceit.net/createItem?name=$job"
```

The output of the above example should resemble:

```
HTTP/1.1 100 Continue

HTTP/1.1 200 OK
Content-Length: 0
Server: Jetty(8.y.z-SNAPSHOT)
```

## Creating and Removing Pipelines for Projects

> **WARNING:** deleting a pipeline will delete the Jenkins jobs and assocated job history from Jenkins.
> Re-adding a cookbook or app for will automatically restore the jobs, but, not the job history.
> The job history can be restored using the [Jenkins recovery process](https://github.com/ThomasCookOnline/wiki/blob/master/DevOps/Jenkins.md#backup--recovery)
> for the job history that is stored in the [ThomasCookOnline/jenkins-config](https://github.com/ThomasCookOnline/jenkins-config) repo.


### Chef Pipelines
The [DSL seed job for Chef](jobs/dsl~chef.xml) runs when a commit is pushed to
the master branch of [ThomasCookOnline/chef-repo](https://github.com/ThomasCookOnline/chef-repo).
The DSL will create a pipeline for newly added site-cookbooks, and remove pipelines for deleted site-cookbooks.


### Apps

The [DSL seed job for apps](jobs/dsl~apps.xml) requires an entry in the [projects.txt](projects.txt) file for
each app that requires a continuous delivery pipeline.  The [DSL seed job for apps](jobs/dsl~apps.xml) runs when
a new commit is pushed to the master branch of this repo.  The job will create pipelines for new entries in the
[projects.txt](projects.txt) file and remove pipelines for removed entries.


> **TIP:**   The [dsl/apps.dsl.groovy](dsl/apps.dsl.groovy) script could be refactored to support auto-discovery
> of repos and pipeline settings.  The script would need to [spider the list of repos in the ThomasCookOnline org](https://developer.github.com/v3/repos/#list-organization-repositories) rather than parsing the
> [projects.txt](projects.txt) file.
> Spidering the list of repos would be a mild challenge, as this list is paged 20 records at a time using
> [HAL](http://stateless.co/hal_specification.html) links.  GitHub currently does not offer webhook notifications
> for new repos or deleted repos in an org, so the spider would have to be scheduled to run every X hours.

## .jenkins.yml (public API)

The DSL scripts use a `.jenkins.yml` file to imitate the very powerful and simple [.travis.yml build config used by TravisCI](http://docs.travis-ci.com/user/build-configuration/#.travis.yml-file%3A-what-it-is-and-how-it-is-used).
The `.jenkins.yml` file supports app pipelines for:
1) NodeJS apps
2) Java web apps
3) Scala apps

Jenkins job @DSL_Apps uses `.jenkins.yml` file in the root of your repository to learn about your project and how you want your builds to be executed

[The settings helper class](dsl/helpers/settings.groovy) is responsible for parsing `.jenkins.yml`.

``` yaml
GITHUB_ORG*:
GITHUB_PROJECT*:
pipeline*:									# (*) main section that define pipeline
  type: nodejs-githubflow					# (*)
  title: Canary/NodeJS
  group: Canary Apps
  validateCommit:
  customLabel:
  notifyJira: true
target_domain*:
chef*:
  app_name*: tc-canary
  role*: canary-nodejs
git:
  clean:
  wipe:
cdn:
  enabled: true
  origin: 127.0.0.1
  path: /tmp/starfish/canary
grunt:
  enabled: false
  target:
gulp:
  enabled:
  target:
code_climate:
  repo_token:
nodejs:
  enabled:
  version: "4.2.1"
  devtools: "/opt/rh/devtoolset-2/root"
npm:
  publish:
jdk:
maven:
  group: com.thomascook
  artifact: hello-world
  artifact_folder: .
  packaging: war
  envopts: -Xmx1g -XX:MaxPermSize=768m
  full_deploy:
  full_output:
  version:
  snapshot:
    goals: clean org.jacoco:jacoco-maven-plugin:0.7.4.201502262128:prepare-agent deploy
    envopts: -Xmx1g -XX:MaxPermSize=768m
sbt:
  version:
  actions:
  jvm_flags:
  pull_request:
  publish:
  release:
  version_bump:
skip_sonar:
sonar:
  pr_dryrun_enable_sonar:
  pr_dryrun_sonar_project_suffix:
restart_service:
e2e_tests:
  test_job_name:
  target_environments:
post_tests:
  suites:
        - name: 'uk-npm'
          after: 'integration'
          npm: 'run fail'
          cucmber:
                  archivePath: 'target/protractor/cucumber/reports/**/*'
                  reportPath: 'target/protractor/cucumber/reports'
                  include: '**/*.json'
notifications:
  build_fail:
  deploy:
    integration:
    qa:
    staging:
    production:
enable_metrix_callback:
```


Contact GitHub API Training Shop Blog About


### .jenkins.yml for NodeJS apps

``` yaml
--- # continuous delivery pipeline settings used by the jenkins-jobdsl jobs
pipeline:
  type: nodejs           # required
  title: Canary/NodeJS   # optional; display name for the pipeline view; defaults to the repo name
  group: DSL             # optional; name of the pipeline collection/view; defaults to 'App Pipelines'
  notifyJira:            # optional; will pick up Jira issue numbers from commit comments in the form [ISSUE-002] and update those issues with pipeline progress
chef:
  app_name: tc-canary	 # required; name of the node module, used as a key in the chef environment apps element
  role: canary-nodejs    # required; name of the chef role for servers running this app; defaults to repo name
grunt:
  enabled: true          # optional; boolean to enable/disable grunt builds for client-side code; defaults to false
code_climate:
  repo_token: abc123     # optional; CodeClimate.com api key to enable code coverage reporting; defaults to nil
npm:
  publish: tarball       # optional; if value is `tarball`, publishing will push a tarball of the static content to npm; defaults to nil to publish a regular npm module; grunt builds should be enabled when this option is used
sonar:
  pr_dryrun_enable_sonar: true      # optional; enable SonarQube for Pull Request Dry Run job. defaults to false  
  pr_dryrun_sonar_project_suffix:   # optional; suffix name for sonar project, used to devide master and Pull Request Dry Run projects on Sonar side. Default value : 'PR'
post_tests:
  nodejsInstallation: NodeJS 4.4.3   # specifies a NodeJS version (must be one Jenkins is configured to use)
  suites:
        - name: 'uk-quick'
          after: 'integration'
          grunt: 'runTests phantomJS --market=uk --env=integration'  # the grunt args
          xvfb: false                     # turns off xvfb management. This defaults to on, as this will probably be the common case
        - name: 'nl-smoke'
          after: 'integration'
          grunt: 'runTests smoke firefox --market=nl --env=integration'
        - name: 'performance'
          after: 'qa'
          grunt: 'runTests performance --env=qa'
          github: 'ThomasCookOnline/web-foo-perftests'  # this will check out code from the specified repo to run the tests from
```

### .jenkins.yml for Java web apps

``` yaml
--- # continuous delivery pipeline settings used by the jenkins-jobdsl jobs
pipeline:
  type: java                        # required
  title: Canary/Java                # optional; display name for the pipeline view; defaults to the repo name
  group: DSL                        # optional; name of the pipeline collection/view; defaults to 'App Pipelines'
  notifyJira:                       # optional; will pick up Jira issue numbers from commit comments in the form [ISSUE-002] and update those issues with pipeline progress
chef:
  role: canary-java                 # required; name of the chef role for servers running this app; defaults to repo name
  app_name: canary-java             # required; key name fort this app in the chef envrionment `apps`; defaults to repo name
maven:
  group: com.thomascook             # required; maven group to use for publishing to Nexus
  artifact: hello-world             # required; name of the artifact to publish to Nexus
  artifact_folder: .                # optional; relative path of the directory for the artifact to publish to Nexus; defaults to `./$artifact/`
  packaging: war                    # required; the file extension used for the artifact when publishing to Nexus, likely `war` or `zip`
sonar:
  pr_dryrun_enable_sonar: true      # optional; enable SonarQube for Pull Request Dry Run job. defaults to false  
  pr_dryrun_sonar_project_suffix:   # optional; suffix name for sonar project, used to devide master and Pull Request Dry Run projects on Sonar side. Default value : 'PR'

```

### Specifying where a build should run

The jenkins-jobdsl project allows pipelines to be targeted at Jenkins slaves with specific labels, based upon their underlying technology.
For example, a Maven build will run on slaves labelled 'java'. Sometimes this may not be enough control, so it is now possible to further
customise the labels against which a pipeline can run.

``` yaml
--- # continuous delivery pipeline settings used by the jenkins-jobdsl jobs
pipeline:
  type:             java            
  title:            Canary/Java     
  group:            DSL      
  customLabel:     java java-one # this will replace the platform default label with these
chef:
  role: canary-java     
  app_name: canary-java
maven:
  group: com.thomascook  
  artifact: hello-world  
  artifact_folder: .   
  packaging: war
```

Note that this configuration affects only pull request jobs and build master jobs. Everything else
behaves as before.

### .jenkins.yml for Chef cookbooks

The [dsl/chef.dsl.groovy](dsl/apps.dsl.groovy) script does not currently support the `.jenkins.yml` file,
as the Chef DSL assumes we are using a Chef "megarepo" where all wrapper cookbooks are stored in a single Git repo.
This simplifies discovery of the wrapper cookbooks, as we can just list the contents of the `site-cookbooks` folder
in the repo.

The other model for Chef source control is to use a single Git repo per wrapper cookbook.  This model has several
advantages (and a few disadvantages) over megarepos  However, single repos would require moving to Berkshelf 3.0
and running an internal [Berkshelf API server](https://github.com/berkshelf/berkshelf-api) to discover
the locations of reusable wrapper cookbooks like `tc-nodejs`, `tc-java`, and `tc-nginx`.
Should ThomasCook.com move to the single repo per cookbook model, it would be wise to implement support
for `.jenkins.yml` in the Chef DSL scripts.

## Configure Sonar Gateway

To start failing Pull Request Dry Run Jenkins builds on Sonar violations we need to implement three steps:

- Add 'pr_dryrun_enable_sonar' option to jenkins.yml file. It will add SonarQube for Pull Request Dry Run Jenkins job. For the master job it's enabled by default.
- Create or configure Quality Gates on Sonar side. http://docs.sonarqube.org/display/SONARQUBE45/Quality+Gates
- Map new project to Sonar Quality Gate.

## Configure Email Notifications
To enable email notifications for custom recipient list in jenkins, you need to add in .jenkins.yaml this section:

``` yaml
notifications:
  build_fail: build1@example.com, build2@example.com 	# optional; comma-separated list of recipient addresses. E-mail will be sent when build with type "Build + Test + Publish" get status: failure, unstable, fixed, regression etc (without success status)
  deploy:
    integration: integration@example.com 						# optional; comma-separated list of recipient addresses. E-mail will be sent when any build (with success status also) with type "Deploy" will run on INTEGRATION environment
    qa: qa@example.com  								# optional; comma-separated list of recipient addresses. E-mail will be sent when any build (with success status also) with type "Deploy" will run on QA environment
    staging: staging@example.com 						# optional; comma-separated list of recipient addresses. E-mail will be sent when any build (with success status also) with type "Deploy" will run on STAGING environment
    production: production@example.com 					# optional; comma-separated list of recipient addresses. E-mail will be sent when any build (with success status also) with type "Deploy" will run on PRODUCTION environment
```

## Updating Metrix dashboard
To update Metrix dashboard for your project after each Sonar run you need to add in .jenkins.yaml this section:

``` yaml
enable_metrix_callback: true # optional; boolean; enable updating Metrix dashboard after each SonarQube analysis run
```

## NewRelic deployment markers
To put deployment markers on newrelic's graphs you need to add these settings in your .jenkins.yaml:
``` yaml
newrelic:
  app_name: canary-java		# optional, project name in newrelic
  deployment_marker: true
```
Note: environment name is concatenated to project name, example: canary-java-integration. 

## Release Approvers

The [Continuous Delivery](https://github.com/ThomasCookOnline/wiki/blob/master/DevOps/ContinuousDelivery.md#flow) flow
allows any member of the GitHub organization to trigger the build/test/publish task, promote a build to the
integration environment, and promote a build to the QA environment.

Only named release approvers whose GitHub username is listed in the [approvers.txt](approvers.txt) file are allowed
to promote a build to the staging environment or the production environemnt.  This list of named approvers should
be carefully limited to a small group who understand fully how to deconflict releases and understand how to
revert a production release.

> **TIP:**   The [dsl/helpers/approvers.groovy](dsl/helpers/approvers.groovy) script could be refactored to
[list the members of a GitHub organization team like `Release Approvers`](https://developer.github.com/v3/orgs/teams/#get-team-membership) rather than parsing the
[approvers.txt](approvers.txt) file.  MDL felt a text file offered better traceability, visibility,
and access control than using a GitHub team.  Especially since membership in GitHub teams tends to
expand very rapidly.
