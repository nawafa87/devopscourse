pipeline {
  agent any
  stages {

    stage('SCM Checkout') {
      steps {
        echo '>>> Start getting SCM code'
        git branch: 'main', credentialsId: 'nawafa87', url: 'https://github.com/nawafa87/devopscourse.git'
        echo '>>> End getting SCM code'
      }
    }

    stage('Build JAR') {
      steps {
        echo '>>> Start building using Maven'
        sh 'mvn -Dmaven.test.skip=TRUE install'
        echo '>>> End building using Maven'
      }
    }

    stage('Build Docker Image') {
      steps {
        echo '>>> start clearing old docker images'
        sh label: '', script: '''if docker images -a | grep "pipeline-demo*" | awk \'{print $1":"$2}\' | xargs docker rmi -f; then
         printf \'Clearing old images succeeded\\n\'
        else
          printf \'Clearing old images failed\\n\'
        fi'''
        echo '>>> End clearing old docker images'
        echo '>>> Start building App docker image'
        sh "docker build -t $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER --pull=true $WORKSPACE"
        echo '>>> End building App docker image'
      }
    }

    stage('Upload Docker Image') {
      steps {
        echo '>>> Login to Docker hub'
        withCredentials([string(credentialsId: '04ac1743-9b8f-4f9c-a912-7023fd5aff33', variable: 'DockerHubIDSecret')]) {
            sh "docker login -u $MyDockerAccountName -p $DockerHubIDSecret"
        }
        echo '>>> Start uploading the docker image'
        sh "docker push $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER"
        echo '>>> End uploading the docker image'
      }
    }

    stage('Deploy') {
      steps {
        echo 'Start Killing and Removing old continaer'
        
        sh label: '', script: '''if docker ps -a | grep "Jenkins-pipeline-Demo*" | awk \'{print $1}\' | xargs docker rm -f; then
            printf \'Clearing old containers done succeeded\\n\'
        else
            printf \'Clearing old containers failed\\n\'
        fi'''

        echo 'End Killing and Removing old continaer'
        echo 'Start running a new container of the application'
        sh "docker run -d -p 8090:8090 --name=$MyTagName$BUILD_NUMBER $MyDockerAccountName/$MyDockerReposioryName:$MyTagName$BUILD_NUMBER"
        echo 'End running a new container of the application'
      }
    }

  }
  environment {
    MyDockerAccountName = 'nawafa99docker'
    MyDockerReposioryName = 'demo1-master'
    MyTagName = 'Jenkins-pipeline-Demo'
    emailRecipientIDs='mr.n.a.t.1000@gmail.com'
  }
  post { 
        always { 
            echo 'I will always run, and I can clean up the workspace here......'
        }
        success {
            echo 'I will run in case of failure, and I will send an email in case of failure.'
            mail bcc: '', body: """Dears,
            Congratulations the build # $BUILD_NUMBER for the Job [$JOB_NAME] has successded.
            Below are the Build details:
            Build Number : $BUILD_NUMBER
            Build URL is : $BUILD_URL
            Build Tag : $BUILD_TAG
            Node Name : $NODE_NAME
            Workspace : $WORKSPACE
            Thanks. """, cc: '', from: '', replyTo: '', subject: "Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME] >> Success", to: "$emailRecipientIDs"

            echo 'I will send a success notification on the slack channel'
        }
        failure { 
            echo 'I will run in case of success, and I will send an email in case of success'
            mail bcc: '', body: """Dears,
            Unfortunately the build # $BUILD_NUMBER for the Job [$JOB_NAME] has failed.
            Below are the Build details:
            Build Number : $BUILD_NUMBER
            Build URL is : $BUILD_URL
            Build Tag : $BUILD_TAG
            Node Name : $NODE_NAME
            Executor Number : $EXECUTOR_NUMBER
            Workspace : $WORKSPACE
            Thanks. """, cc: '', from: '', replyTo: '', subject: "Build # [$BUILD_NUMBER] triggered for Job [$JOB_NAME] >> Failed", to: "$emailRecipientIDs"

            echo 'I will send a success notification on the slack channel'
        }
  }
}
