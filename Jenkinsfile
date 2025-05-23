pipeline {
  agent any
  stages {
    stage('Clean Reports')
    {
      steps{
        echo '********* Cleaning Workspace Stage Started **********'
        sh 'rm -rf test-reports'
        echo '********* Cleaning Workspace Stage Finished **********'
      }
    }
    
    stage('Build Stage') {
      steps {
        echo '********* Build Stage Started **********'
        sh 'pip3 install -r requirements.txt'
        sh '/opt/homebrew/bin/pyinstaller --onefile app.py'
        echo '********* Build Stage Finished **********'
        }
    }
    stage('Testing Stage') {
      steps {
        echo '********* Test Stage Started **********'
        sh 'python3 test.py'
        echo '********* Test Stage Finished **********'
      }   
    }
    stage('Configure Artifactory'){
      steps{
        script {
          
          echo '********* Configure Artifactory Started **********'
             def userInput = input(
             id: 'userInput', message: 'Enter password for Artifactory', parameters: [
             
             [$class: 'TextParameterDefinition', defaultValue: 'password', description: 'Artifactory Password', name: 'password']])
             
             sh '/opt/homebrew/bin/jfrog config add artifactory-demo --url=https://34.68.191.118:8081/artifactory --user=admin --password=admin --interactive=false'

          echo '********* Configure Artifactory Finished **********'
        }
       }
    }
    stage('Sanity check') {
            steps {
                input "Does the staging environment look ok?"
            }
     }
stage('Deployment Stage'){
            steps{
                input "Do you want to Deploy the application?"
                echo '********* Deploy Stage Started **********'
                timeout(time : 1, unit : 'MINUTES')
                {
                sh 'python3 app.py'
                }
                echo '********* Deploy Stage Finished **********'
            }
    }
  }
  post {
        always {
            echo 'We came to an end!'
            archiveArtifacts artifacts: 'dist/*.exe', fingerprint: true
            junit 'test-reports/*.xml'
          script{
            if(currentBuild.currentResult=='SUCCESS')
            {
              echo '********* Uploading to Artifactory is Started **********'
              /*bat 'jfrog rt u "dist/*.exe" generic-local'*/
              sh 'pwsh -ExecutionPolicy RemoteSigned -File build_script.ps1'
              echo '********* Uploading Finished **********'
            }
          }
          
            
            deleteDir()

         }
        success {
          echo 'Build Successfull!!'
    }
        failure {
        echo 'Sorry mate! build is Failed :('
    }
        unstable {
            echo 'Run was marked as unstable'
        }
        changed {
            echo 'Hey look at this, Pipeline state is changed.'
        }
    }
}
