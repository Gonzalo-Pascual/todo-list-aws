pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_SESSION_TOKEN     = credentials('aws-session-token')
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Gonzalo-Pascual/todo-list-aws.git'
                sh 'ls -la'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region ${AWS_DEFAULT_REGION}
                    sam deploy \
                        --stack-name todo-list-aws-production \
                        --region ${AWS_DEFAULT_REGION} \
                        --capabilities CAPABILITY_IAM \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset \
                        --parameter-overrides Stage=production
                '''
            }
        }

        stage('Rest Test') {
            steps {
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
                    echo "API URL: ${env.BASE_URL}"
                }

                sh '''
                    export BASE_URL=${BASE_URL}
                    python3 -m pytest test/integration/todoApiTest.py \
                        -k "listtodos or gettodo" \
                        --junitxml=result-rest-production.xml \
                        -v
                '''
            }
            post {
                always {
                    junit 'result-rest-production.xml'
                }
            }
        }
    }
}