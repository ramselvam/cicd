pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }

    stages{    

        // stage("sonar quality check"){
        //    agent {
        //       docker {
        //          image 'openjdk:11'
        //         }
        //     }
        //    steps {
        //        script{
        //         sh 'chmod +x gradlew'
        //         sh "./gradlew build   |  tee output.log"
        //          }
        //       }


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
                          sh 'kubectl apply -f nginx.yaml'
                          sh 'kubectl apply -f service.yaml'  
                        }
                    }
               }
            }
        }

        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                         sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl 172.16.16.101:32321'

                     }
                }
            }
        }
    }


}
