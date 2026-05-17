pipeline {
    agent any

    stages {

        stage('Build Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t registration-app .'
            }
        }

        stage('Deploy Container') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'docker-host',
                            transfers: [
                                sshTransfer(
                                    execCommand: '''
                                        docker stop registration-app || true
                                        docker rm registration-app || true
                                    '''
                                )
                            ]
                        )
                    ]
                )

                sh '''
                    docker save registration-app > registration-app.tar
                '''

                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'docker-host',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'registration-app.tar',
                                    remoteDirectory: '/home/ubuntu/app',
                                    execCommand: '''
                                        docker load < /home/ubuntu/app/registration-app.tar
                                        docker run -d --name registration-app -p 8080:8080 registration-app
                                    '''
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
}
