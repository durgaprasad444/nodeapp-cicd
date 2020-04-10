def label = "jenkins-slave-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'slave1', image: 'durgaprasad444/jenmine:1.1', ttyEnabled: true, command: 'cat')
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
    node(label) {
        def APP_NAME = "nodeapp-cicd"
        def tag = "dev"
        def gitBranch = env.BRANCH_NAME
           
          stage("Checking Branch") {
                container('slave1') {  
                    sh """
                    echo "branch is"  ${gitBranch}          
                    """
                }
  }
                    stage('checkout scm') {
                        container('slave1') {
                               checkout scm
                            
 }
                    }
                    
                    stage("build & publish") {
            container('slave1') {
                   sh """
                   cd /home/jenkins/workspace/nodeapp-cicd
                   npm install
                    """
                    }

}
             stage('Build image') {
            container('slave1') {
                sh """
                cd /home/jenkins/workspace/nodeapp-cicd
                docker build -t gcr.io/kube-cluster-237706/${APP_NAME}-${tag}:$BUILD_NUMBER .
                """
                
  
}
}
stage('Push image') {
    container('slave1') {
      sh """
      cd /home/jenkins/workspace/nodeapp-cicd
      docker login -u username -p password
      docker push gcr.io/kube-cluster-237706/${APP_NAME}-${tag}:$BUILD_NUMBER
    
      """
  
    }
}


  stage("deploy on kubernetes") {
            container('slave1') {
                sh "cd /home/jenkins/workspace/nodeapp-cicd"
                sh "kubectl apply -f node-app.yaml"
                sh "kubectl set image deployment/node-app node-app=gcr.io/kube-cluster-237706/${APP_NAME}-${tag}:$BUILD_NUMBER"
            }
        }  

                }
            }
