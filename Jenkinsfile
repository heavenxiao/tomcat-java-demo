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
            sh """
            cat pw.txt | docker login --username lizhenliang --password-stdin ${registry}
            echo $PWD
            docker build -t ${image_name} $PWD
            docker push ${image_name}
            """
          }
      }
    }
  }
}
