# Demo project for Jenkinsfile Runner Action for GitHub Actions in GSoC 2022

This is the demo project for the original [POC project](https://github.com/Cr1t-GYM/jenkins-action-poc). 
It teaches the users how to use Jenkinsfile-runner Actions.
All the source files are generated by [Spring Initializr](https://start.spring.io/).

## How you can access Jenkinsfile-runner actions in your project?
Reference these actions in your workflow definition.
1. Cr1t-GYM/jenkins-action-poc/jenkins-plugin-installation-action@master
2. Cr1t-GYM/jenkins-action-poc/jenkinsfile-runner-action@master
3. Cr1t-GYM/jenkins-action-poc/jfr-container-action@master
4. Cr1t-GYM/jenkins-action-poc/jfr-static-image-action@master

## Step by step usage
1. Prepare a Jenkinsfile in your repository. You can check [the basic syntax of Jenkins pipeline definition](https://www.jenkins.io/doc/book/pipeline/syntax/).
2. Prepare a workflow definition under the `.github/workflows` directory. You can check [the official manual](https://docs.github.com/en/actions) for more details.
3. In your GitHub Action workflow definition, you need to follow these steps when calling other actions in sequence:
    1. Use a ubuntu runner for the job.
   ```Yaml
   jobs:
      job-name:
        runs-on: ubuntu-latest   
   ```
    2. If you use jfr-container-action, you need to declare using the `jenkins/jenkinsfile-runner` or any image extended it. If you use jfr-static-image-action, you can skip this step.
   ```Yaml
   jobs:
      job-name:
        runs-on: ubuntu-latest
        container:
          image: jenkins/jenkinsfile-runner             
   ```   
    3. Call the `actions/checkout@v2` to pull your codes into the runner.
    4. If you use jfr-container-action, you need to call `Cr1t-GYM/jenkins-action-poc/jfr-container-action@master` and give necessary inputs. If you use jfr-static-image-action, you need to call `Cr1t-GYM/jenkins-action-poc/jfr-static-image-action@master` and give necessary inputs. See the [examples](#workflow-explanation) for these two actions.

## Workflow Explanation
You can find the workflow definition in the [.github/workflows/ci.yml](.github/workflows/ci.yml).
### jenkins-static-image-pipeline
This job pulls your repository first, and the workspace will be mapped to the container provided by 
`jfr-static-image-action`. By using the `Cr1t-GYM/jenkins-action-poc/jfr-static-image-action@master` and
passing the necessary inputs, you can start the pipeline. Please note that most of the GitHub Actions will become
invalid because of the isolation between the host machine and the container. You still can use `actions/checkout@v2`
to prepare your sources.
```yaml
  jenkins-static-image-pipeline:
    runs-on: ubuntu-latest
    name: jenkins-static-image-pipeline-test
    steps:
      - uses: actions/checkout@v2
      - name: Jenkins pipeline with the static image
        id: jenkins_pipeline_image
        uses:
          Cr1t-GYM/jenkins-action-poc/jfr-static-image-action@master
        with:
          command: run
          jenkinsfile: Jenkinsfile
          pluginstxt: plugins.txt
          jcasc: jcasc.yml
```
### jenkins-container-pipeline
This job pulls the `jenkins/jenkinsfile-runner`, pulls your repository, setup maven and run the Jenkins pipeline finally.
Please note that the declaration of `jenkins/jenkinsfile-runner` is necessary to use the `jfr-container-action` action.
By using the `Cr1t-GYM/jenkins-action-poc/jfr-container-action@master` and
passing the necessary inputs, you can start the pipeline.
```yaml
  jenkins-container-pipeline:
    runs-on: ubuntu-latest
    name: jenkins-prebuilt-container-test
    container:
      image: jenkins/jenkinsfile-runner
    steps:
      - uses: actions/checkout@v2
      - name: Set up Maven
        uses: stCarolas/setup-maven@v4.3
        with:
          maven-version: 3.6.3
      - name: Jenkins pipeline in the container
        id: jenkins_pipeline_container
        uses:
          Cr1t-GYM/jenkins-action-poc/jfr-container-action@master
        with:
          command: run
          jenkinsfile: Jenkinsfile
          pluginstxt: plugins.txt
          jcasc: jcasc.yml
```
### jenkins-runtime-pipeline
This job runs all the GitHub Actions including Jenkinsfile-runner actions inside the host machine.
It pulls your repository first, install the plugins by using `jenkins-plugin-installation-action`,
and run the pipeline by using `jenkinsfile-runner-action`. The installation of plugins and the entrypoint of pipeline
are decoupled in this job. You need to call the `Cr1t-GYM/jenkins-action-poc/jenkins-plugin-installation-action@master`
first and then call the `Cr1t-GYM/jenkins-action-poc/jenkinsfile-runner-action@master`.
```yaml
  jenkins-runtime-pipeline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Jenkins plugins download
        id: jenkins_plugins_download
        uses:
          Cr1t-GYM/jenkins-action-poc/jenkins-plugin-installation-action@master
        with:
          pluginstxt: plugins_min.txt
      - name: Run Jenkins pipeline
        id: run_jenkins_pipeline
        uses:
          Cr1t-GYM/jenkins-action-poc/jenkinsfile-runner-action@master
        with:
          command: run
          jenkinsfile: Jenkinsfile
          jcasc: jcasc_runtime.yml
```