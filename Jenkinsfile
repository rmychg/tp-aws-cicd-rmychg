pipeline {
    agent any

    options {
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: 'Environnement cible')
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version de artefact')
        booleanParam(name: 'DRY_RUN', defaultValue: true, description: 'Mode simulation')
        string(name: 'CHANGE_REFERENCE', defaultValue: '', description: 'Ticket / demande de changement')
    }

    stages {
        stage('Préparation') {
            steps {
                echo "Environnement : ${params.ENVIRONMENT}"
                echo "Version : ${params.VERSION}"
                echo "Dry run : ${params.DRY_RUN}"
                echo "Changement : ${params.CHANGE_REFERENCE}"
                sh 'git rev-parse --short HEAD'
            }
        }
        stage('Validation') {
            steps {
                script {
                    if (!(params.ENVIRONMENT in ['dev', 'test', 'prod'])) {
                        error("Environnement non autorisé : ${params.ENVIRONMENT}")
                    }
                }
                echo "Paramètres validés."
            }
        }
        stage('Exécution') {
            steps {
                sh '''
                    mkdir -p artifacts
                    printf 'build=%s\\ncommit=%s\\nenv=%s\\nversion=%s\\n' "$BUILD_NUMBER" "$(git rev-parse --short HEAD)" "$ENVIRONMENT" "$VERSION" > artifacts/build-info.txt
                '''
            }
        }
        stage('Post-traitement') {
            steps {
                archiveArtifacts artifacts: 'artifacts/*.txt', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "Pipeline terminé avec succès."
        }
        failure {
            echo "Le pipeline a échoué — vérifier les logs ci-dessus."
        }
        always {
            echo "Fin du build ${env.BUILD_NUMBER} pour ${params.ENVIRONMENT}."
        }
    }
}
