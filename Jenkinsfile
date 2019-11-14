def projectName = 'repository1'
def namespace='dev'

pipeline {
  environment {
    DOCKER = credentials("docker")
  }
  agent {
    kubernetes {
      label 'app'
      defaultContainer 'jnlp'
      yaml """
          apiVersion: v1
          kind: Pod
          metadata:
            labels:
              component: ci
          spec:
            serviceAccount: jenkins
            containers:
              - name: docker
                image: karima/repository:ubuntu-kubectl-docker
                imagePullPolicy: Always
                command:
                  - cat
                tty: true                
                volumeMounts:
                  - name: dockersock
                    mountPath: /var/run/docker.sock
                  - name: daemon
                    mountPath: /etc/docker/daemon.json 
            volumes:
            - name: dockersock
              hostPath:
                path: /var/run/docker.sock
            - name: daemon
              hostPath:
                path:  /etc/docker/daemon.json               
        """
    }
  }
  stages {
    stage('build') {
      steps {
        container('docker') { 
          sh("echo yes")
          sh("#docker build --network=host -t $DOCKER_USR/${projectName}:nginx-1 .")
        }
      }
    }
    stage('push') {
      steps {
        container('docker') {        
          sh("#docker login -u $DOCKER_USR -p $DOCKER_PSW")
          sh("#docker push $DOCKER_USR/${projectName}:nginx-1")
        }
      }
    }    
    stage('deploy') {
      steps {
        container('docker') {        
          sh("kubectl get pod --namespace=${namespace}")
          sh("kubectl apply -f deployment.yml --namespace=${namespace}")
          sh("sleep 5 && kubectl get pod --namespace=${namespace}")
        }
      }
    }   
    stage('test') {
      steps {
        container('docker') {        
          sh """
           echo this is a second branch
           ip=\$(kubectl get svc helloworld --namespace=${namespace} -o json | jq -r .status.loadBalancer.ingress[0].hostname)
           while [ "\$(curl --insecure -s -o /dev/null -w "%{http_code}" \$ip)" != "200" ]; do sleep 3; done
          """

        }
      }
    }     
  }
}

