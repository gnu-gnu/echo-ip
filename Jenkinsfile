podTemplate(
    serviceAccount: 'jenkins',
    containers: [
        containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
    ],
    volumes: [
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
        hostPathVolume(mountPath: '/usr/bin/kubectl', hostPath: '/usr/bin/kubectl')
    ]
)

{
    node(POD_LABEL) {
        stage('git scm update') {
            git url: 'https://github.com/IaC-Source/echo-ip.git', branch: 'main'
        }
        stage('docker build and push') {
            container("docker") {
                sh'''
                docker build -t 192.168.1.10:8443/echo-ip .
                docker push 192.168.1.10:8443/echo-ip
                '''
            }
        }
        stage('deploy kubernetes') {
            sh'''
            kubectl create deployment pl-bulk-prod --image=192.168.1.10:8443/echo-ip
            kubectl expose deployment pl-bulk-prod --type=LoadBalancer --port=8080 \
                                                   --target-port=80 --name=pl-bulk-prod-svc
            '''
        }
    }
}
