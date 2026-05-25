pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_SESSION_TOKEN     = credentials('aws-session-token')
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {

        // ETAPA 1: GET CODE
        stage('Get Code') {
            steps {
                sh 'whoami ; hostname ; hostname -I ; uname -a'
                git branch: 'master',
                    url: 'https://github.com/Gonzalo-Pascual/todo-list-aws.git'
                echo "WORKSPACE: ${env.WORKSPACE}"
                sh 'git rev-parse --abbrev-ref HEAD'
                sh 'ls -la'
            }
        }

        // ETAPA 2: DESPLIEGUE EN PRODUCCIÓN
        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region ${AWS_DEFAULT_REGION}
                    sam deploy \
                        --config-file samconfig.toml \
                        --config-env production \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset
                '''
                script {
                    env.BASE_URL = sh(
                        script: '''
                            aws cloudformation describe-stacks \
                                --stack-name todo-list-aws-production \
                                --region ${AWS_DEFAULT_REGION} \
                                --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                                --output text
                        ''',
                        returnStdout: true
                    ).trim()
                    echo "BASE_URL: ${env.BASE_URL}"
                }
            }
        }

        // ETAPA 3: PRUEBAS DE INTEGRACIÓN (solo lectura)
        stage('Rest Test') {
            steps {
                sh """
                    echo "API URL: ${env.BASE_URL}"
                    python3 -m pytest test/integration/todoApiTest.py \
                        -k "test_api_listtodos or test_api_gettodo" \
                        --junitxml=result-rest-production.xml \
                        -v
                """
            }
            post {
                always {
                    junit 'result-rest-production.xml'
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
