pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git 'https://github.com/jlcalleja/unir1-4'
            }
        }

        stage('SAM Deploy') {
            steps {
                sh 'sam build'
                sh '''
                    sam deploy --no-confirm-changeset \
                        --no-fail-on-empty-changeset \
                        --stack-name todo-list-aws-production \
                        --capabilities CAPABILITY_IAM \
                        --parameter-overrides Stage=production \
                        --resolve-s3
                '''
            }
        }

        stage('Test Rest'){
            steps{
                script {
                    // Obtener el valor de BASE_URL desde CloudFormation
                    def baseUrlCommand = "aws cloudformation describe-stacks --stack-name todo-list-aws-staging --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --region us-east-1 --output text"
                    // Asignar el valor de BASE_URL a la variable de entorno de Jenkins
                    env.BASE_URL = sh(script: baseUrlCommand, returnStdout: true).trim()
                    
                    // Verificar el valor de BASE_URL
                    echo "BASE_URL: ${env.BASE_URL}"
                    
                    // Ejecutar pytest pasando BASE_URL como variable de entorno
                    sh "BASE_URL=${env.BASE_URL} pytest -m read_only test/integration/todoApiTest.py"
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
