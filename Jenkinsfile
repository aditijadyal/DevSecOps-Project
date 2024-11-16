pipeline {
    agent any
    
    environment {
        VAGRANT_ENV = 'production' // Set environment variable for Vagrant
    }

    stages {
        stage('Provisioning') {
            steps {
                script {
                    try {
                        sh 'vagrant up'  // Provision the VMs using Vagrant
                    } catch (Exception e) {
                        error "Provisioning failed"
                    }
                }
            }
        }
        
        stage('Run Lynis Audit') {
            steps {
                script {
                    try {
                        sh 'vagrant ssh webserver -c "lynis audit system --quiet > /vagrant/lynis-web-audit.log"' // Run Lynis on Web Server
                        sh 'vagrant ssh dbserver -c "lynis audit system --quiet > /vagrant/lynis-db-audit.log"'   // Run Lynis on DB Server
                    } catch (Exception e) {
                        error "Lynis Audit failed"
                    }
                }
            }
        }

        stage('Rollback') {
            when {
                expression {
                    return currentBuild.result == 'FAILURE'  // Rollback if any stage fails
                }
            }
            steps {
                script {
                    echo 'Rolling back to previous stable state...'
                    sh 'vagrant destroy -f'  // Destroy VMs to rollback
                }
            }
        }
    }
}
