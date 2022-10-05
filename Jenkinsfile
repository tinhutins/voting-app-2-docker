pipeline {
    agent none
    stages {
        stage('worker-build!!') {
        agent {
            docker {
            image 'maven:3.8.1-adoptopenjdk-11'
            args  '-v $HOME/.m2:/root/.m2'
        }
       }
       when {
         changeset "**/worker/**"
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
       when {
         changeset "**/worker/**"
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
        changeset "**/worker/**"
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
        when {
          changeset "**/vote/**"
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
        when {
          changeset "**/vote/**"
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
                docker.withRegistry('https://registry-1.docker.io/v2/', 'dockerlogin'){
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
        when {
          changeset "**/result/**"
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
          when {
            changeset "**/result/**"
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
                docker.withRegistry('https://registry-1.docker.io/v2/', 'dockerlogin'){
                def workerImage = docker.build("hutinskit/result:v${env.BUILD_ID}", "./result")
                workerImage.push()
                workerImage.push("${env.BRANCH_NAME}")
                }
            }

            }
        }

        stage('Sonarqube') {
        agent any



        environment{
          sonarpath = tool 'SonarScanner' // the namdse you have given the sonarscanner installation in Global Tool Configuration
        }

        steps {
              echo 'Running Sonarqube Analysis..'
              withSonarQubeEnv('sonar-voting-app-2-docker') {
                sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
              }
        }
      }


        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('deploy to dev!'){
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
