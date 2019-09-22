pipeline {
  agent {
    label 'maven'
  }
  stages {
    stage('Build App') {
      steps {
        sh "mvn clean package -s src/main/config/settings.xml"
      }
    }
    stage('Integration Test') {
      steps {
        sh "mvn verify -s src/main/config/settings.xml"
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              openshift.startBuild("cart", "--from-file=target/cart.jar").logs("-f")
            }
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              dc = openshift.selector("dc", "cart")
              dc.rollout().latest()
              timeout(10) {
                  dc.rollout().status()
              }
            }
          }
        }
      }
    }
    stage('Component Test') {
      steps {
        script {
          sh "curl -s -X POST http://cart:8080/api/cart/dummy/666/1"
          sh "curl -s http://cart:8080/api/cart/dummy | grep 'Dummy Product'"
        }
      }
    }
  }
}
