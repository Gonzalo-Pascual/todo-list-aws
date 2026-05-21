pipeline {
    agent any

    environment {
        // Inyectamos tus credenciales temporales de AWS Academy guardadas en Jenkins
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_SESSION_TOKEN     = credentials('AWS_SESSION_TOKEN')
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {
        // ETAPA 1: DESCARGA CÓDIGO FUENTE
        stage('Get Code') {
            steps {
                echo "Descargando el código del proyecto..."
                git branch: 'develop', url: 'https://github.com/Gonzalo-Pascual/todo-list-aws.git'
            }
        }

        // ETAPA 2: PRUEBAS ESTÁTICAS
        stage('Static Test') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    echo "Ejecutando Flake8..."
                    sh 'flake8 --format=pylint --exit-zero src > flake8.out'
                    
                    echo "Ejecutando Bandit..."
                    sh 'bandit --exit-zero -r src -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"'
                }
            }
        }

        // ETAPA 3: DESPLIEGUE EN STAGING (Inspirada en tu ejemplo)
        stage('Deploy') {
            steps {
                echo "Compilando y Desplegando en Staging con SAM..."
                sh '''
                sam build
                sam deploy \
                    --config-file samconfig.toml \
                    --config-env staging \
                    --no-confirm-changeset \
                    --no-fail-on-empty-changeset | tee deploy_output.txt
                '''
                
                // Verificamos que se creó el log del despliegue
                sh 'cat deploy_output.txt'
            }
        }
    }
}