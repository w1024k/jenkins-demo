def label = "mypod-${UUID.randomUUID().toString()}"

podTemplate(label: label, cloud: 'kubernetes', 
    volumes: [
      hostPathVolume(mountPath: '/root/.kube', hostPath: '/root/.kube'),
      hostPathVolume(mountPath: '/usr/bin/kubectl', hostPath: '/usr/bin/kubectl'),
      hostPathVolume(mountPath: '/usr/bin/docker', hostPath: '/usr/bin/docker'),
      hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
    ]) {

    node(label) {
        stage('Prepare') {
            echo "1.Prepare Stage"
            checkout scm
            script {
                build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                if (env.BRANCH_NAME != 'master') {
                    build_tag = "${env.BRANCH_NAME}-${build_tag}"
                }
            }
        }
        stage('Test') {
          echo "2.Test Stage"
        }
        stage('Build') {
            echo "3.Build Docker Image Stage"
            sh "docker build -t cnych/jenkins-demo:${build_tag} ."
        }
        stage('Push') {
            echo "4.Push Docker Image Stage"
            withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
                sh "docker push cnych/jenkins-demo:${build_tag}"
            }
        }
        stage('Deploy') {
            echo "5. Deploy Stage"
            if (env.BRANCH_NAME == 'master') {
                input "确认要部署线上环境吗？"
            }
            sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
            sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
            sh "kubectl apply -f k8s.yaml --record"
        }
    }

}
