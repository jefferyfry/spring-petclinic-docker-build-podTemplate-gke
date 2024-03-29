
pipeline {
  agent {
    kubernetes {
        label 'docker-build-pod'
        yamlFile 'podTemplate/spring-petclinic-docker-build.yaml'
        idleMinutes 120
    }
  }
  stages {
    stage('Maven Install') {
      steps {
        container('maven') {
          sh 'mvn clean install'
        }
      }
    }
    stage('Docker Build') {
      steps {
        container('docker'){
          sh 'docker build -t jefferyfry/spring-petclinic:latest .'
        }
      }
    }
    stage('Docker Push') {
      steps {
        container('docker'){
          withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
            sh 'docker push jefferyfry/spring-petclinic:latest'
          }
        }
      }
    }
    stage('Staging') {
      steps {
        container('gcloud-kubectl'){
          withCredentials([string(credentialsId: 'gke-access-token', variable: 'TOKEN')]) {
            sh '''
                 kubectl --token=${TOKEN} delete namespace spring-petclinic-docker-build || true
                 sleep 5
                 kubectl --token=${TOKEN} create namespace spring-petclinic-docker-build
                 kubectl --token=${TOKEN} run spring-petclinic-docker-build --image=jefferyfry/spring-petclinic:latest --port 8080 --namespace spring-petclinic-docker-build
                 kubectl --token=${TOKEN} expose deployment spring-petclinic-docker-build --type=LoadBalancer --port 8092 --target-port 8080 --namespace spring-petclinic-docker-build
                 while [ -z "$url" ]; do url=$(kubectl --token=${TOKEN} describe service spring-petclinic-docker-build --namespace spring-petclinic-docker-build | grep 'LoadBalancer Ingress:' | awk '{printf "http://%s:8092",$3;}'); sleep 2; done
                 echo "$url"
                 echo "Spring PetClinic Launched!"
                 
            '''
          }
        }
      }
    }
  }
}