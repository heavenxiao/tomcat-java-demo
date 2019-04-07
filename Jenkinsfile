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
    // 第一步：拉取代码
    stage('Checkout') {
      git branch: 'master', url: 'https://github.com/lizhenliang/demo.git'
      // 生成镜像标签，格式：commit号-更新时间
      tag = sh(returnStdout: true, script: "date +%Y%m%d%H%M|tr -d '\n';echo -|tr -d '\n';git rev-parse --short HEAD").trim()
      project = "blog"
      app_name = "demo"
      namespace = "default"
      registry = "192.168.31.61"
      image_name = "${registry}/${project}/${app_name}:${tag}"
      // 第二步：代码编译
      container('maven') {
          stage('Maven Build') {
              sh 'mvn clean install -DskipTests'
          }
      }
      // 第三步：构建镜像
      container('docker') {
          stage('Build Docker Image') {
          withCredentials([usernamePassword(credentialsId: '74a933e5-d7cf-4369-8ef6-7b8d882b3cb7', passwordVariable: 'password', usernameVariable: 'username')]) {
            sh """
            echo '
              FROM lizhenliang/tomcat 
              MAINTAINER lizhenliang
              RUN rm -rf /usr/local/tomcat/webapps/*
              ADD target/*.war /usr/local/tomcat/webapps/ROOT.war 
            ' > Dockerfile
            docker login -u ${username} -p '${password}' ${registry}
            docker build -t ${image_name} .
            docker push ${image_name}
            """
            }
          }
      }
      // 第四步：部署
      stage('Deploy to Kubernetes') {
        sh "sed -i 's#\$IMAGE_NAME#${image_name}#' deploy.yml"
        sh "sed -i 's#\$KUBERNETES_SECRET_NAME#registry-pull-secret#' deploy.yml"
        sh "cat deploy.yml"
        kubernetesDeploy configs: 'deploy.yml', 
        kubeconfigId: 'fad95334-37ee-427b-b4f0-ac11d03a2d19', 
        secretName: 'registry-pull-secret'
        //sh "echo $KUBERNETES_SECRET_NAME"
      }
    }
  }
}
