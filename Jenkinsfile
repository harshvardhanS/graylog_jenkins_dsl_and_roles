pipeline {
    agent any
    parameters {
        choice(
            choices: ['create' , 'destroy'],
            description: '',
            name: 'REQUESTED_ACTION')
    }

    environment {
        TERRAFORM_CMD='/opt/terraform/terraform'
    }

    stages {
        stage("Prep") {
            
            steps { 
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'devuser_terraform', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                //sh "cp $tfvars terraform.tfvars"
                sh """
cat <<EOF > terraformKey
aws_access_key = "${AWS_ACCESS_KEY_ID}"
aws_secret_key = "${AWS_SECRET_ACCESS_KEY}"
EOF
"""             }
            }    
        }
        stage('Checkout'){
            steps {
                //checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanBeforeCheckout']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/harshvardhanS/graylog-infra.git']]]    
                git poll: false, url: 'https://github.com/harshvardhanS/graylog-infra.git'
                //sh 'printenv'
            }
        }
        stage('Terraform plan'){
            when {
                // Only say hello if a "greeting" is requested
                expression { params.REQUESTED_ACTION == 'create' }
            }
            steps {
                sh """
                ${TERRAFORM_CMD} init
                ${TERRAFORM_CMD} plan -out=myplan -input=false -var-file=terraformKey
                """
                script {
                  timeout(time: 10, unit: 'MINUTES') {
                    input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
                  }
                }    
            }
        }
        stage('Approval') {
            steps {
                script {
                def userInput = input(id: 'confirm', message: 'Apply Terraform?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Apply terraform', name: 'confirm'] ])
                }
            }
        }
        stage('Terraform apply') {
            when {
                // Only say hello if a "greeting" is requested
                expression { params.REQUESTED_ACTION == 'create' }
            }            
            steps {
                 sh """
                ${TERRAFORM_CMD} apply -var-file=terraformKey -auto-approve
                """
            }
        }
        stage('Terraform destroy') {
            when {
                // Only say hello if a "greeting" is requested
                expression { params.REQUESTED_ACTION == 'destroy' }
            }            
            steps {
                 sh """
                ${TERRAFORM_CMD} destroy -var-file=terraformKey -auto-approve
                """
            }
        }
    }
}
