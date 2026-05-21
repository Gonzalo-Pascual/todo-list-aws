pipeline {
    agent any

    stages {
        // ETAPA 1: DESCARGA CÓDIGO FUENTE
        stage('Get Code') {
            steps {
                echo "Descargando el código del proyecto..."
                git branch: 'develop', url: 'https://github.com/Gonzalo-Pascual/todo-list-aws.git'
                sh 'ls -la'
            }
        }

        // ETAPA 2: PRUEBAS ESTÁTICAS
        stage('Static Test') {
            steps {
                // catchError asegura que si hay errores de formato, el pipeline siga adelante
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    echo "Ejecutando Flake8..."
                    sh 'flake8 --format=pylint --exit-zero src > flake8.out'
                    
                    echo "Ejecutando Bandit..."
                    sh 'bandit --exit-zero -r src -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"'
                    
                    // Mostramos el resultado rápido en la consola de Jenkins para verificarlo
                    sh 'cat flake8.out'
                    sh 'cat bandit.out'
                }
            }
        }
    }
}