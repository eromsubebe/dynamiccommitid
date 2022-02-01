pipeline {
  environment {
    TAG = 'blue'
  }
  options {
    ansiColor('xterm')
  }

  // def branch = env.BRANCH_NAME
  // def commit = env.GIT_COMMIT

  // Usage 
  // sh 'echo $BRANCH_NAME'
  // sh 'echo $GIT_COMMIT'

  agent {
    kubernetes {
      yamlFile 'builder.yaml'
    }
  }

  stages {

    stage(testenv) {
      steps {
        script {
          def fields = env.getEnvironment()
          fields.each {
            key, value --> println("${key} = ${value}");
          }
          println(env.PATH)
        }
      }
    }

    stage('Kaniko Build & Push Image') {
      steps {
        container('kaniko') {
          script {
            sh '''
            /kaniko/executor --dockerfile `pwd`/Dockerfile \
                             --context `pwd` \
                             --destination=eromski/mybuildnumber:${BUILD_NUMBER}
            '''
          }
        }
      }
    }

    stage('Deploy App to Kubernetes') {     
      steps {
        container('kubectl') {
          withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
            sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" myweb.yaml'
            sh 'kubectl apply -f myweb.yaml'
          }
        }
      }
    }
  
  }
}
