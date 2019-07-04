pipeline {
    agent {
        kubernetes {
            label 'mypod'
            defaultContainer 'jnlp'
            yamlFile 'podTemplate.yaml'
        }
    }
    triggers {
        eventTrigger simpleMatch('dcanadillas/kaniko-petclinic:latest')
    }
    stages {
        stage('Deploy') {
            steps {
                container('kubectl') {
                    echo 'Deploying k8 app'
                    //sh 'kubectl get pods -n staging'
                    sh 'kubectl get deployments -n staging'
                    // Let's delete deployment petclinic if still deployed
                    // We want to do a fresh deployment of the image
                    sh '''
                    MYDEPLOY=$(kubectl get deployments -n staging | awk '/^petclinic/ {print $1}')
                    if [ "$MYDEPLOY" = "petclinic" ];then kubectl delete deployment -n staging $MYDEPLOY; fi
                    '''
                    sh 'kubectl apply -f test-deploy.yaml -n staging'
                    // Now let's check all the deployments again
                    sh 'kubectl get deployments -n staging'
                    echo 'Spring Petclinic app has been deployed'

                }
            }
        }
        stage('Check') {
            steps {
                //error 'Fake error to force failure in Build'
                container('kubectl') {
                    //copyArtifacts projectName: '../petclinic-kaniko/master'
                    gateConsumesArtifact id: 'kaniko'
                    //error 'Forcing error to check DevOptics'
                    echo 'Check Deployment and Service'
                    sh 'kubectl get deployments,svc -n staging'
                    sh 'kubectl describe deployment petclinic -n staging'
                }
            }
        }
    }
}
