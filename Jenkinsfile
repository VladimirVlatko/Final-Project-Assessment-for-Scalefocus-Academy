pipeline {
    agent any

    stages {
        // Check if wp namespace exists, create it if not
        stage('Check and create wp namespace') {
            steps {
                script {
                    def wpNamespace = sh(returnStatus: true, 
                        script: 'kubectl get namespace wp --no-headers=true -o name')
                    if (wpNamespace != 0) {
                        echo 'Creating wp namespace'
                        sh('kubectl create namespace wp')
                    } else {
                        echo 'wp namespace already exists'
                    }
                }
            }
        }

        // Check if WordPress Helm chart is installed, install it if not
        stage('Check and install WordPress') {
            steps {
                script {
                    def wpDeployment = sh(returnStatus: true, script: '''
                        helm list -n wp --short \
                        --kubeconfig "C:/Windows/System32/config/systemprofile/AppData/Local/MicroK8s/config" \
                        | grep final-project-wp-scalefocus
                    ''')
                    if (wpDeployment != 0) {
                        echo 'Installing WordPress Helm chart'
                        // Build chart dependencies
                        sh('helm dependency build "C:/Users/V&M/Desktop/final/wordpress"')
                        // Install WordPress Helm chart using Helm command
                        sh('helm install final-project-wp-scalefocus "C:/Users/V&M/Desktop/final/wordpress" ' +
                           '-n wp -f "C:/Users/V&M/Desktop/final/wordpress/values.yaml" ' +
                           '--kubeconfig "C:/Windows/System32/config/systemprofile/AppData/Local/MicroK8s/config"')
                    } else {
                        echo 'WordPress Helm chart already installed'
                    }
                }
            }
        }
        
        // Port Forwarding
        stage('Port Forwarding') {
            steps {
                script {
                    // Wait for WordPress deployment to be ready
                    sleep(time: 1, unit: 'MINUTES')
                    
                    // Run port forwarding command
                    sh('kubectl port-forward --namespace wp svc/final-project-wp-scalefocus-wordpress 8081:80')
                }
            }
        }
    }
}
