pipeline {
    agent any
    tools {
        maven 'maven'
    }
    stages {
        stage("Pull SRC") {
            steps {
                git branch: 'main', url: 'https://github.com/bhoomikadk16/terraform-ansible.git'
            }
        }
        stage("Prepare Build") {
            steps {
                sh 'mvn clean package'
            }
        }
        stage("Copy *.war file to ansible") {
            steps {
                sh 'mv target/hash.war .'
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "ssh",
                            transfers: [sshTransfer(sourceFiles: 'hash.war')],
                            verbose: true
                        )
                    ]
                )
            }
        }
        stage("Copying Docker file to ansible") {
            steps {
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "ssh",
                            transfers: [
                                sshTransfer(sourceFiles: 'Dockerfile'),
                                sshTransfer(execCommand: "docker rm -f cont; docker rmi ansi; docker build -t ansi .; docker run -it -d -p 8081:8080 --name cont ansi")
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }
        stage("Build Docker image and push to dockerhub") {
            steps {
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "ssh",
                            transfers: [
                                sshTransfer(sourceFiles: 'Dockerfile'),
                                sshTransfer(execCommand: """
                                docker rm -f cont
                                docker rmi -f bhoomika720/ansible
                                docker build -t bhoomika720/ansible .
                                docker login -u bhoomika720 -p dockeraccount
                                docker push bhoomika720/ansible
                                """)
                            ],
                            verbose: true
                        )
                    ]
                )
            }
            }
         stage("Copy playbook file to ansible and execute") {
            steps {
               
                sshPublisher(
                    continueOnError: false, 
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "ssh",
                            transfers: [
                                sshTransfer(sourceFiles: 'playbook.yml'),
                                sshTransfer(execCommand: "ansible-playbook playbook.yml")
                            ],
                            verbose: true
                        )
                    ]
                )
            
            }
        }
        
    }
}