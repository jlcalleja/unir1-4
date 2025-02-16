pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                git url: 'https://github.com/jlcalleja/unir1-4', branch: 'develop', credentialsId: '4d4a1cc8-ee35-4ff6-8837-90699db6e6f7'
            
                
                sh 'curl -o samconfig.toml https://raw.githubusercontent.com/jlcalleja/todo-list-aws-config/staging/samconfig.toml'
                
                sh 'ls -la' 
                sh 'cat samconfig.toml' 
            }
        }
        
        stage('Static Test') {
            steps {
                sh 'flake8 src/ > flake8_report.txt || true'
                sh 'bandit -r src/ -f html -o bandit_report.html || true'
            }
        }

        stage('SAM Deploy') {
            steps {
                sh 'sam build'
                sh '''
                    sam deploy --no-confirm-changeset \
                        --no-fail-on-empty-changeset \
                        --config-env staging \
                        --resolve-s3 \
                        --parameter-overrides Stage=staging
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
                    sh "BASE_URL=${env.BASE_URL} pytest test/integration/todoApiTest.py"
                }
            }
        }


        stage('Promote') {
            steps {
                script {
                    // Configurar nombre de usuario para Git
                    sh "git config --global user.name 'JLC'"
                    
                    // Asegurarse de estar en la rama correcta
                    sh 'git checkout develop'
                    
                    // Asegura que Jenkins tenga todas las ramas
                    sh 'git fetch --all'  
                    
                    sh 'git checkout master'
                    sh 'git pull origin master'

                    // Realizar el merge desde 'develop' a 'master'
                    def mergeResult = sh(script: 'git merge develop', returnStatus: true)
                    
                    if (mergeResult != 0) {
                        echo "Merge conflict detected, proceeding to resolve..."
                        
                        // Resolver el conflicto: Mantener la versión de 'master' para Jenkinsfile y Jenkinsfile_agentes
                        sh 'git checkout --ours Jenkinsfile'
                        sh 'git checkout --ours Jenkinsfile_agentes'
                        sh 'git add Jenkinsfile Jenkinsfile_agentes'
                        sh 'git commit -m "Resolved merge conflict: keeping Jenkinsfile and Jenkinsfile_agentes from master"'


                    }
        
                    // Usar las credenciales de tipo 'Username with password' directamente
                    withCredentials([usernamePassword(credentialsId: '4d4a1cc8-ee35-4ff6-8837-90699db6e6f7', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
                        // Hacer el git push con el nombre de usuario y el token
                        sh "git push https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/jlcalleja/unir1-4.git master"
                    }
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
