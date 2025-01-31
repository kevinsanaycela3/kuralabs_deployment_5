pipeline {
  agent any
  environment {
    dockerhub = credentials('DOCKER_CREDS')
  }
  stages {
    stage ('Build') {
      steps {
        sh '''#!/bin/bash
        python3 -m venv test3
        source test3/bin/activate
        pip install pip --upgrade
        pip install -r requirements.txt
        export FLASK_APP=application
        flask run &
        '''
         }
     }

    stage ('Test') {
        steps {
            sh '''#!/bin/bash
            sudo apt install python3.10-venv -y
            python3 -m venv test3
            source test3/bin/activate
            py.test --verbose --junit-xml test-reports/results.xml
            '''
        }
        post{
            always {
                junit 'test-reports/results.xml'
                }
            }
    }
    stage ('Create Image') {
        agent{label 'dockerAgent'}
        steps {
            sh '''#!/bin/bash
            sudo apt update
            sudo apt -y install git
            sleep 3
            git clone https://github.com/kevinsanaycela3/kuralabs_deployment_5.git
            sleep 2
            cd kuralabs_deployment_5/
            tar --exclude='./Documentation' --exclude='./Jenkinsfile' --exclude='./READEME.md' --exclude='./intTerraform/' --exclude='./test_app.py' --exclude='./dockerfile' -cvf url_app.tar.gz .
            sudo docker pull python
            sudo docker build -t kos44/kura_apps .
            '''

        }
    }
    stage ('Push to Docker'){
        agent { label 'dockerAgent' }
        steps {
            sh '''#!/bin/bash
            echo $dockerhub_PSW | sudo docker login -u $dockerhub_USR --password-stdin
            sudo docker push kos44/kura_apps
            docker logout
            '''
            
        }
    }
    stage ('Deploy to ECS') {
    agent{label 'terraformAgent'}
    steps {
        withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'), string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
            dir('intTerraform') {
                sh '''#!/bin/bash
                terraform init
                terraform plan -out plan.tfplan -var="aws_access_key=$aws_access_key" -var="aws_secret_key=$aws_secret_key"
                terraform apply plan.tfplan
                sleep 300
                '''
                }
            }

        }
    }




   }
}


   







 