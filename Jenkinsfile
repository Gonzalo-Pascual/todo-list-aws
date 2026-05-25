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
                git branch: 'develop',
                    url: 'https://github.com/Gonzalo-Pascual/todo-list-aws.git'
                echo "WORKSPACE: ${env.WORKSPACE}"
                sh 'git rev-parse --abbrev-ref HEAD'
                sh 'ls -la'
            }
        }

        // ETAPA 2: PRUEBAS ESTÁTICAS
        stage('Static Test') {
            steps {
                sh '''
                    flake8 --exit-zero --format=pylint src > flake8.out
                    cat flake8.out
                '''
                recordIssues(
                    id: 'flake8',
                    tools: [pyLint(name: 'Flake8', pattern: 'flake8.out')]
                )
                sh '''
                    bandit --exit-zero -r src \
                        -f custom \
                        -o bandit.out \
                        --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                    cat bandit.out
                '''
                recordIssues(
                    id: 'bandit',
                    tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')]
                )
            }
        }

        // ETAPA 3: DESPLIEGUE EN STAGING
        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region ${AWS_DEFAULT_REGION}
                    sam deploy \
                        --stack-name todo-list-aws-staging \
                        --region ${AWS_DEFAULT_REGION} \
                        --capabilities CAPABILITY_IAM \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset \
                        --parameter-overrides Stage=staging
                '''
                script {
                    env.BASE_URL = sh(
                        script: '''
                            aws cloudformation describe-stacks \
                                --stack-name todo-list-aws-staging \
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

        // ETAPA 4: PRUEBAS DE INTEGRACIÓN
        stage('Rest Test') {
            steps {
                sh """
                    echo "API URL: ${env.BASE_URL}"
                    python3 -m pytest test/integration/todoApiTest.py \
                        --junitxml=result-rest.xml \
                        -v
                """
            }
            post {
                always {
                    junit 'result-rest.xml'
                }
            }
        }

        // ETAPA 5: PROMOTE
        stage('Promote') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-token',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh '''
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"
                        git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/Gonzalo-Pascual/todo-list-aws.git
                        git fetch origin
                        git checkout master
                        git merge origin/develop --no-ff -m "CI: Merge develop into master [auto]"
                        git push origin master
                        git remote set-url origin https://github.com/Gonzalo-Pascual/todo-list-aws.git
                    '''
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