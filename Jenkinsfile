def label = "jenkins-slave"
podTemplate(
  label: label, 
  cloud: 'kubernetes', 
  containers: [
    containerTemplate(name: 'docker', image: 'docker:18', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    hostPathVolume(mountPath: '/root/.m2', hostPath: '/tmp/m2'),
  ],
) 
{
  node(label) {
    stage('Checkout SCM') {
      git branch: 'master', url: 'https://github.com/lizhenliang/demo.git'
      tag = sh(returnStdout: true, script: "git rev-parse --short HEAD|tr -d '\n';echo -|tr -d '\n';date +%Y%m%d%H%M")
      project = "blog"
      app_name = "demo"
      namespace = "default"
      registry = "192.168.31.61"
      image_name = "${registry}/${project}/${app_name}:$tag"
      container('maven') {
          stage('Maven Build') {
              sh "mvn clean install -DskipTests"
          }
      }
      container('docker') {
          stage('Build Docker Image') {
          withCredentials([usernamePassword(credentialsId: '74a933e5-d7cf-4369-8ef6-7b8d882b3cb7', passwordVariable: 'password', usernameVariable: 'username')]) {
            sh """
            echo '
            FROM lizhenliang/tomcat 
            MAINTAINER www.ctnrs.com123
            RUN rm -rf /usr/local/tomcat/webapps/*
            ADD target/*.war /usr/local/tomcat/webapps/ROOT.war 
            ' > Dockerfile
            cat ${env.WORKSPACE}/Dockerfile

            docker login -u ${username} -p '${password}' ${registry}
            docker build -t 192.168.31.61/blog/demo:2c6abb1-201904061959 -f /home/jenkins/workspace/test/Dockerfile /home/jenkins/workspace/test 
            #docker build -t ${image_name} -f ${env.WORKSPACE}/Dockerfile ${env.WORKSPACE} 
            docker push ${image_name}
            """
            }
          }
      }
    }
  }
}
