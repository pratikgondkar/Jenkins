# Assignment 4

## Problem Statement

1. Perform CI Checks On Below Apis Using Declarative Pipeline
    (eg: clean, clone, cred scan, unit test, code coverage, dependency check etc. ).
    {Mandatory} - Java:- https://github.com/OT-MICROSERVICES/salary-api
    {Optional}  - GoLang:- https://github.com/OT-MICROSERVICES/employee-api
    {Optional}  - Python:- https://github.com/OT-MICROSERVICES/attendance-api
3. Should store Reports of ALL CI checks.
4. Artifact should be stored locally or remote. 
4. Notification should be sent email/slack/telegram.
 Optional:- 
    - Try Same with scripted pipeline.

## Plugin's  used.

[![Git Plugin](https://img.shields.io/badge/Git-green.svg)](https://plugins.jenkins.io/git/)
[![Git Plugin](https://img.shields.io/badge/BlueOcean-red.svg)](https://plugins.jenkins.io/blueocean/)


## In Jenkkins Dashboard select New Item and create a pipeline Job.
![image](https://github.com/Kiran-dehlikar/test/assets/104997588/c844011f-dabc-42df-88b3-700c534c8d1a)

## Under configuration enter the below pipeline script.

## Pipeline Script


```bash
  pipeline {
    agent any
    stages {
        stage('Code clone and compile') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'java7', url: 'https://github.com/OT-MICROSERVICES/salary-api.git'
                sh "mvn compile"
            }
        }
        
        stage('cred Scan') {
            steps {
                sh "gitleaks detect -v"
            }
        }
        stage('Unit test') {
            steps {
                sh "mvn checkstyle:checkstyle"
                sh "mvn pmd:pmd"
            }
        }
        stage('Dependency check') {
            steps {
                sh "mvn org.owasp:dependency-check-maven:9.0.8:check"
            }
        }
        stage('Code Coverage') {
            steps {
                sh "mvn jacoco:prepare-agent"
            }
        }
        stage('Generate Artifact') {
            steps {
                sh "mvn package -Dmaven.test.skip=true"
            }
        }
        stage('Generate Artefact & Report') {
            steps {
                //Archive Artifacts
                archiveArtifacts artifacts: 'archiveArtifacts artifacts: \'target/*.jar, target/*.war\'', followSymlinks: false
                // Archive the Reports
                archiveArtifacts artifacts: 'checkstyle-result.xml', fingerprint: true
                archiveArtifacts artifacts: 'target/pmd.xml', fingerprint: true
                
            }
        }
    }
    post {
        always {
                recordIssues(tools: [checkStyle(), pmdParser()],
                aggregatingResults: true
                )
                jacoco()
        }
        success {
            slackSend channel: '#pipeline-dec', color: 'good', message: "Job '${env.JOB_NAME}' Success!", teamDomain: 'kiran-workspacegroup', tokenCredentialId: 'df4374b7-51c5-4d43-8297-a5e1971ef969	'
        }
        failure {
            // Notify on failure 'Secret text'
            slackSend channel: '#pipeline-dec', color: 'danger', message: "Job '${env.JOB_NAME}' Failed!", teamDomain: 'kiran-workspacegroup', tokenCredentialId: 'df4374b7-51c5-4d43-8297-a5e1971ef969'
        }
    }
}
```

![Screenshot from 2024-01-23 20-04-36](https://github.com/Kiran-dehlikar/test/assets/104997588/2d9c3eb5-b281-4948-a058-eace9b9abcef)

## Enter the above pipeline script then save and apply.
![Screenshot from 2024-01-23 20-04-56](https://github.com/Kiran-dehlikar/test/assets/104997588/5822394a-1e3d-4e38-9a94-6483338dee30)

## Run the pipeline by clicking Build now button.
![Screenshot from 2024-01-23 20-00-21](https://github.com/Kiran-dehlikar/test/assets/104997588/7cfb8aaa-bf31-4954-8528-4755163fa72a)

## You will see the Pipeline overview during running time with the help of blue ocean plugin.
![Screenshot from 2024-01-23 19-59-17](https://github.com/Kiran-dehlikar/test/assets/104997588/7d11b3f8-03a1-4c12-9c7e-3d6da5039662)
