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
node(label) {

    stage('Build with Kaniko') {
    
      git 'https://github.com/timmykit-ogcio/kubernetes-ci-cd.git'
        container(name: 'kaniko', shell: '/busybox/sh') {
          withEnv(['PATH+EXTRA=/busybox:/kaniko']) {
            sh '''#!/busybox/sh
            /kaniko/executor -f `pwd`/applications/hello-kenzan/Dockerfile -c `pwd` applications/hello-kenzan --insecure-skip-tls-verify --destination=mydockerregistry:5000/myorg/myimage
            '''
          }       
        }
    }   

}
}
