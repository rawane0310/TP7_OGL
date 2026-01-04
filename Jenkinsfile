pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the project...'
                bat './gradlew clean build -x test'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                // 1. Lancement des tests unitaires
                bat './gradlew test'
            }
            post {
                always {
                    // 2. Archivage des résultats des tests unitaires
                    junit '**/build/test-results/test/*.xml'

                    // 3. Génération des rapports de tests Cucumber
                    cucumber buildStatus: 'UNSTABLE',
                        reportTitle: 'Cucumber Test Report',
                        fileIncludePattern: '**/reports/*.json',
                        trendsLimit: 10,
                        classifications: [
                            [
                                'key': 'Browser',
                                'value': 'Chrome'
                            ]
                        ]
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished!'
        }
        success {
            echo 'Tests passed successfully!'
        }
        failure {
            echo 'Tests failed!'
        }
    }
}