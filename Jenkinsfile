def label = "jenkins-slave-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'slave1', image: 'gcr.io/sentrifugo/jenkins-slave:v1', ttyEnabled: true, command: 'cat')
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
    node(label) {
        def APP_NAME = "node-app"
        def tag = "dev"
            stage("clone code") {
                container('slave1') {                  
  stage 'checkout  repository'
  checkout([$class: 'GitSCM',
        branches: [[name: '*/master']],
        doGenerateSubmoduleConfigurations: false,
        extensions: [],
        submoduleCfg: [],
        userRemoteConfigs: [[
            
            url: 'https://github.com/durgaprasad444/node-app.git'
    ]]])

                }
            }
        
        stage("build & publish") {
            container('slave1') {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'nexus_cred',
usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                   sh """
                      cd /home/jenkins/workspace/node-app
                      echo "registry=http://35.196.136.112:8081/repository/npm-group/" >> .npmrc
                      echo -n '${USERNAME}:${PASSWORD}' | openssl base64 >> .npmrc
                      sed -i '2 s/^/_auth=/' .npmrc
                      echo -e "email=nexus@gmail.com\nalways-auth=true" >> .npmrc
                      npm install
                      sed -i 's/npm-group/npm-private/g' .npmrc
                      npm publish
                    """
}
}
}
stage('Build image') {
            container('slave1') {
                sh """
                cd /home/jenkins/workspace/node-app
                docker build -t gcr.io/sentrifugo/${APP_NAME}-${tag}:$BUILD_NUMBER .
                """
                
  
}
}
stage('Push image') {
    container('slave1') {
  docker.withRegistry('https://gcr.io', 'gcr:sentrifugo') {
      sh "docker push gcr.io/sentrifugo/${APP_NAME}-${tag}:$BUILD_NUMBER"
    
    
  }
    }
}


  stage("deploy on kubernetes") {
            container('slave1') {
                sh "kubectl set image deployment/node-app node-app-dev-sha256=gcr.io/sentrifugo/${APP_NAME}-${tag}:$BUILD_NUMBER"
            }
        }  
                }
            }
