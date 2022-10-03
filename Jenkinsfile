pipeline {
    agent {
        docker {
        image 'maven:3.8.1-adoptopenjdk-11'
        args  '-v $HOME/.m2:/root/.m2'
    }
   }
    stages {
        stage('build') {
            steps {
                echo 'compiling worker app!'
		dir ('worker'){
                sh ' mvn compile'
		}
            }
        }
        stage('test'){
            steps {
                echo 'Running unit tests'
                dir ('worker'){
                sh 'mvn clean test'
                }  
	  }
        }
        stage('package') {
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
    }
    
    post {
        always {
            echo 'Pipeline for worker is completed.'
        }
    }
    
    
}
