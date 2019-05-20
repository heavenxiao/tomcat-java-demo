
// 公共
def registry = "10.206.240.188"
def git = "10.206.240.189"
// 项目
def project = "project"
def app_name = "demo"
def image_name = "${registry}/${project}/${app_name}"
def git_address = "git@${git}:/home/git/solo.git"
// 认证
def secret_name = "registry-pull-secret"
def docker_registry_auth = "75df606e-e706-4749-a21c-86df7ed4b5b2"
def git_auth = "75df606e-e706-4749-a21c-86df7ed4b5b2"
def k8s_auth = "9f15ff8f-c5e4-4523-8bfe-cc02d921984a"

podTemplate(label: 'jenkins-slave', cloud: 'kubernetes', containers: [
    containerTemplate(
        name: 'jnlp', 
        image: '10.206.240.188/library/jenkins-slave-jdk:1.8'
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
         checkout([$class: 'GitSCM', branches: [[name: '${Tag}']], userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]])
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
