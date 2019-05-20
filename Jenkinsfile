
// 公共
def registry = "10.206.240.188"
// 项目
def project = "project"
def app_name = "demo"
def image_name = "${registry}/${project}/${app_name}:${BUILD_NUMBER}"
def git_address = "git@10.206.240.189:/home/git/demo.git"
// 认证
def secret_name = "registry-pull-secret"
def docker_registry_auth = "f192bf8b-acb9-4561-a914-50b2c884393c"
def git_auth = "e120e48b-3846-46e9-9401-8e7482c8eee5"
def k8s_auth = "cfa44e97-6ae7-45ba-b1d1-4f927f928f80"

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


