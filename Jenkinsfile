pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }

    stages{    

        stage("sonar quality check"){
           agent {
              docker {
                 image 'openjdk:11'
                }
            }
           steps {
               script{
                sh 'chmod +x gradlew'
                sh "./gradlew build   |  tee output.log"
                 }
              }


        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                             sh '''
                                docker build -t 172.16.16.11:5000/springapp:${VERSION} .
                                docker login -u admin -p $docker_password 172.16.16.11:5000
                                docker push  172.16.16.11:5000/springapp:${VERSION}
                                docker rmi 172.16.16.11:5000/springapp:${VERSION}
                            '''
                    }
                }
            }
        }

        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="34.125.214.226:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
                    }
               }
            }
        }

        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                         sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'

                     }
                }
            }
        }
    }


}
