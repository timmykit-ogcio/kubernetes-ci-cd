 def label = "kaniko-${UUID.randomUUID().toString()}"
 
 podTemplate(name: 'kaniko', label: label, yaml: """
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /root
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: docker-credentials
          items:
            - key: .dockerconfigjson
              path: .docker/config.json
"""
  )
{
node {

    checkout scm

    env.DOCKER_API_VERSION="1.23"
    
    sh "git rev-parse --short HEAD > commit-id"

    tag = readFile('commit-id').replace("\n", "").replace("\r", "")
    appName = "hello-kenzan"
    registryHost = "127.0.0.1:30400/"
    imageName = "${registryHost}${appName}:${tag}"
    env.BUILDIMG=imageName

    stage "Build" {
    
//        sh "docker build -t ${imageName} -f applications/hello-kenzan/Dockerfile applications/hello-kenzan"
      git 'https://github.com/jenkinsci/docker-jnlp-slave.git'
        container(name: 'kaniko', shell: '/busybox/sh') {
          withEnv(['PATH+EXTRA=/busybox:/kaniko']) {
            sh '''#!/busybox/sh
            /kaniko/executor -f `pwd`/applications/hello-kenzan/Dockerfile -c `pwd` applications/hello-kenzan --insecure-skip-tls-verify --destination=mydockerregistry:5000/myorg/myimage
            '''
          }       
        }
    }   

    stage "Push"

        sh "docker push ${imageName}"

    stage "Deploy"

        kubernetesDeploy configs: "applications/${appName}/k8s/*.yaml", kubeconfigId: 'kenzan_kubeconfig'

}
}
