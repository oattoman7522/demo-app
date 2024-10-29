pipeline {
    environment {
        harbor_server = 'dev-harbor.ktam.co.th'
        IMAGE = 'dev-harbor.ktam.co.th/test-srs/frontend/agent:v1'
        git_url = 'https://github.com/oattoman7522/demo-app.git'
        branch_git = dev
    }
    
    agent any

    stages {
        stage('Check out') {
            
             git url: ${git_url}, branch: 'dev', credentialsId: 'test-git'
        }
        
        stage('Docker Login and Pull') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'harbor_dev', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh '''
                        docker login ${harbor_server} -u $USERNAME -p $PASSWORD
                        cd demo-app/
                        ls -al
                        docker build -t ${harbor_server}/test-srs/test:${BUILD_NUMBER} .
                        docker push ${harbor_server}/test-srs/test:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
        
        stage('Write YAML File') {
            steps {
                script {
                    // Define the YAML content as a multi-line string
                    def yamlContent = """
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: test-nginx
  name: test-deployment
  namespace: ktam-dev
spec:
   replicas: 2
   selector:
     matchLabels:
       run: test-nginx
   template:
     metadata:
       labels:
         run: test-nginx
   spec:
     imagePullSecrets:
     - name: harbor-registry
     containers:
     - image: ${harbor_server}/test-srs/test:${BUILD_NUMBER}
       name: test
       imagePullPolicy: IfNotPresent
     dnsPolicy: ClusterFirst
     restartPolicy: Always
                    """

                    // Write the YAML content to a file
                    writeFile file: 'test-deploy.yaml', text: yamlContent

                    // Optionally, print the file content to verify
                    sh 'cat test-deploy.yaml'
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: 'jenkins@kubernetes', credentialsId: 'kubeconfig_jenkins_dev', namespace: '', serverUrl: 'https://192.168.1.217:6443') {
                    sh 'ls -al'
                    sh 'cat test-deploy.yaml'
                    sh 'kubectl replace --force -f test-deploy.yaml -n ktam-dev'
                    sh 'kubectl get po -n ktam-dev'
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace.'
            cleanWs()
        }
    }
}