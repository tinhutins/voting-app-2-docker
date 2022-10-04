pipeline {
    agent none
    stages {
        stage('worker-build') {
        agent {
            docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args  '-v $HOME/.m2:/root/.m2'
        }
       }
            steps {
                echo 'compiling worker app!'
        		dir ('worker'){
                sh ' mvn compile'
        		}
            }
        }
        stage('worker-test'){
        agent {
            docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args  '-v $HOME/.m2:/root/.m2'
        }
       }
            steps {
                echo 'Running unit tests'
                dir ('worker'){
                sh 'mvn clean test'
                }
	  }
        }
        stage('worker-package') {
        agent {
            docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args  '-v $HOME/.m2:/root/.m2'
        }
       }
            when {
	      branch 'master'
	    }
            steps {
                echo 'Packaging worker app'
                dir ('worker'){
		sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker-package-docker') {
             agent any
             when {
              branch 'master'
             }
            steps {
                echo 'Packaging worker app with docker '
            script {
                docker.withRegistry('https://registry-1.docker.io/v2/', 'dockerlogin'){
                def workerImage = docker.build("hutinskit/worker:v${env.BUILD_ID}", "./worker")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                }
            }

            }
        }

        stage('vote-build') {
        agent {
        docker { image 'python:2.7.16'
         args '--user root'
        }
        }
            steps {
                echo 'compilling vote app!!'
            dir ('vote'){
                        sh 'pip install -r requirements.txt '
            }
            }
        }
        stage('vote-test') {
        agent {
        docker { image 'python:2.7.16'
         args '--user root'
        }
        }

            steps {
                echo 'Running unit tests on vote app'
                dir ('vote'){
                sh 'pip install -r requirements.txt'
                sh 'nosetests -v'
                }
    }
        }

        stage('vote-package-docker') {
             agent any
             when {
              branch 'master'
             }
            steps {
                echo 'Packaging vote app with docker '
            script {
                docker.withRegistry('https://index.docker.io/v2/', 'dockerlogin'){
                def workerImage = docker.build("hutinskit/vote:v${env.BUILD_ID}", "./vote")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                }
            }

            }
        }

        stage('result-build') {
          agent {
        docker { image 'node:16.13.1-alpine' }
        }

            steps {
                echo 'compiling result app'
            dir ('result'){
                        sh ' npm install'
            }
            }
        }
        stage('result-test') {
         agent {
          docker { image 'node:16.13.1-alpine' }

          }
            steps {
                echo 'Running unit tests on result app'
                dir ('result'){
                sh 'npm install'
                sh 'npm test'
                }
    }
        }

        stage('result-package-docker') {
             agent any
             when {
              branch 'master'
             }
            steps {
                echo 'Packaging result app with docker '
            script {
                docker.withRegistry('https://index.docker.io/v2/', 'dockerlogin'){
                def workerImage = docker.build("hutinskit/result:v${env.BUILD_ID}", "./result")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                }
            }

            }
        }
          stage('deploy to dev'){
           agent any
           when {
            branch 'master'
           }
           steps{
             echo 'Deploy instavote app with docker compose'
             sh 'docker-compose up -d'
           }
        }
    }

    post {
        always {
            echo 'Pipeline for worker is completed.'
        }
    }


}
