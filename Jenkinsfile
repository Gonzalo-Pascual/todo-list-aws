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
                sh 'whoami ; hostname ; hostname -I ; uname -a'
                git branch: 'master',
                    url: 'https://github.com/Gonzalo-Pascual/todo-list-aws.git'
                echo "WORKSPACE: ${env.WORKSPACE}"
                sh 'git rev-parse --abbrev-ref HEAD'
                sh 'ls -la'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam delete \
                        --stack-name todo-list-aws-production \
                        --no-prompts \
                        --region ${AWS_DEFAULT_REGION} || true

                    sam build

                    sam deploy \
                        --config-file samconfig.toml \
                        --config-env production \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset | tee deploy_output.txt
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    BASE_URL=$(awk '/Key *BaseUrlApi/{getline; getline; print $2}' deploy_output.txt)
                    export BASE_URL=${BASE_URL}
                    echo "API URL: ${BASE_URL}"
                    python3 -m pytest test/integration/todoApiTest.py \
                        -k "test_api_listtodos or test_api_gettodo" \
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

    post {
        always {
            cleanWs()
        }
    }
}