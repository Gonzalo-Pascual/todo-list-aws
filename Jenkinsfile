pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_SESSION_TOKEN     = credentials('AWS_SESSION_TOKEN')
        AWS_DEFAULT_REGION    = 'us-east-1'
    }

    stages {
        stage('Prueba de Conexion') {
            steps {
                echo '¡Hola! Jenkins ha descargado el código con éxito de GitHub.'
                
                echo 'Verificando conexión con AWS Academy...'
                sh 'aws sts get-caller-identity'
            }
        }
    }
}