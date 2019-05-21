
// 公共
def registry = "192.168.31.64"
// 项目
def project = "welcome"
def app_name = "demo"
def image_name = "${registry}/${project}/${app_name}:${BUILD_NUMBER}"
def git_address = "git@192.168.31.64:/home/git/java-demo.git"
// 认证
def secret_name = "registry-pull-secret"
def docker_registry_auth = "881f76f7-ea94-4397-8006-23c300feae9a"
def git_auth = "b3e33c8b-c7e0-47b9-baee-d7629d71f154"
def k8s_auth = "854ae16f-dfa2-46ee-a214-a4c4aed0768e"

podTemplate(label: 'jenkins-slave', cloud: 'kubernetes', containers: [
    containerTemplate(
        name: 'jnlp', 
        image: "${registry}/library/jenkins-slave-jdk:1.8"
    ),
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    hostPathVolume(mountPath: '/usr/bin/docker', hostPath: '/usr/bin/docker')
  ],
) 
{
  node("jenkins-slave"){
      // 第一步
      stage('拉取代码'){
         checkout([$class: 'GitSCM', branches: [[name: '${Branch}']], userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]])
      }
      // 第二步
      stage('代码编译'){
          sh "mvn clean package -Dmaven.test.skip=true"
      }
      // 第三步
      stage('构建镜像'){
          withCredentials([usernamePassword(credentialsId: "${docker_registry_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
            sh """
              echo '
                FROM lizhenliang/tomcat 
                RUN rm -rf /usr/local/tomcat/webapps/*
                ADD target/*.war /usr/local/tomcat/webapps/ROOT.war 
              ' > Dockerfile
              docker build -t ${image_name} .
              docker login -u ${username} -p '${password}' ${registry}
              docker push ${image_name}
            """
            }
      }
      // 第四步
      stage('部署到K8S平台'){
          sh """
          sed -i 's#\$IMAGE_NAME#${image_name}#' deploy.yml
          sed -i 's#\$SECRET_NAME#${secret_name}#' deploy.yml
          """
          kubernetesDeploy configs: 'deploy.yml', kubeconfigId: "${k8s_auth}"
      }
  }
}


