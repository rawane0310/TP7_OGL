pipeline {
    agent any

    stages {
        // ========================================
        // 2.1 LA PHASE TEST
        // ========================================
        stage('Test') {
            steps {
                echo '========== PHASE TEST =========='
                echo '1. Lancement des tests unitaires...'
                bat './gradlew test'
            }
            post {
                always {
                    echo '2. Archivage des résultats des tests unitaires...'
                    junit '**/build/test-results/test/*.xml'

                    echo '3. Génération des rapports de tests Cucumber...'
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

        // ========================================
        // 2.2 LA PHASE CODE ANALYSIS
        // ========================================
        /*stage('Code Analysis') {
            steps {
                echo '========== PHASE CODE ANALYSIS =========='
                echo 'Analyse de la qualité du code avec SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    bat './gradlew sonarqube'
                }
            }
        }*/

        // ========================================
        // 2.3 LA PHASE CODE QUALITY (Quality Gate)
        // ========================================
        /*stage('Code Quality') {
            steps {
                echo '========== PHASE CODE QUALITY =========='
                echo 'Vérification du Quality Gate de SonarQube...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }*/

        // ========================================
        // 2.4 LA PHASE BUILD
        // ========================================
        stage('Build') {
            steps {
                echo '========== PHASE BUILD =========='

                echo '1. Génération du fichier JAR...'
                bat './gradlew jar'

                echo '2. Génération de la documentation...'
                bat './gradlew javadoc'
            }
            post {
                always {
                    echo '3. Archivage du fichier JAR et de la documentation...'
                    archiveArtifacts artifacts: 'build/libs/*.jar', allowEmptyArchive: false
                    archiveArtifacts artifacts: 'build/docs/javadoc/**/*', allowEmptyArchive: false

                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'build/docs/javadoc',
                        reportFiles: 'index.html',
                        reportName: 'Javadoc Documentation',
                        reportTitles: 'Project Documentation'
                    ])
                }
            }
        }

        // ========================================
        // 2.5 LA PHASE DEPLOY
        // ========================================
        stage('Deploy') {
            steps {
                echo '========== PHASE DEPLOY =========='
                echo 'Déploiement du fichier JAR vers mymavenrepo.com...'
                bat './gradlew publish'
            }
        }

        // ========================================
        // 2.6 LA PHASE NOTIFICATION (SUCCESS)
        // ========================================
        stage('Notification - Success') {
            steps {
                echo '========== NOTIFICATION DE SUCCÈS =========='

                // Notification par Email
                emailext(
                    subject: "Déploiement réussi - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        <h2> Déploiement réussi !</h2>
                        <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build :</strong> #${env.BUILD_NUMBER}</p>
                        <p><strong>Status :</strong> SUCCESS</p>
                        <p><strong>Date :</strong> ${new Date()}</p>
                        <br>
                        <p>Le fichier JAR a été déployé avec succès sur mymavenrepo.com</p>
                        <p><a href="${env.BUILD_URL}">Voir le build</a></p>
                    """,
                    to: 'aitkaciazzousarah@gmail.com',
                    mimeType: 'text/html'
                )

                // Notification Slack
                /*slackSend(
                    color: 'good',
                    message: "Déploiement réussi !\nProjet: ${env.JOB_NAME}\nBuild: #${env.BUILD_NUMBER}\nStatus: SUCCESS\n<${env.BUILD_URL}|Voir le build>"
                )*/
            }
        }
    }

    // ========================================
    // NOTIFICATIONS EN CAS D'ÉCHEC
    // ========================================
    post {
        failure {
            echo '========== NOTIFICATION D\'ÉCHEC =========='

            // Notification par Email en cas d'échec
            emailext(
                subject: "Échec du build - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2> Le build a échoué !</h2>
                    <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build :</strong> #${env.BUILD_NUMBER}</p>
                    <p><strong>Status :</strong> FAILURE</p>
                    <p><strong>Date :</strong> ${new Date()}</p>
                    <br>
                    <p>Une erreur s'est produite lors de l'exécution du pipeline.</p>
                    <p><a href="${env.BUILD_URL}console">Voir les logs</a></p>
                """,
                to: 'aitkaciazzousarah@gmail.com',
                mimeType: 'text/html'
            )

            // Notification Slack en cas d'échec
           /* slackSend(
                color: 'danger',
                message: "Échec du build !\nProjet: ${env.JOB_NAME}\nBuild: #${env.BUILD_NUMBER}\nStatus: FAILURE\n<${env.BUILD_URL}console|Voir les logs>"
            )*/
        }

        success {
            echo 'Pipeline terminé avec succès !'
        }

        always {
            echo 'Pipeline terminé.'
        }
    }
}