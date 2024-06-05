pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 9999999
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  - name: helm-kubectl
    image: dtzar/helm-kubectl:latest
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 9999999
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: docker-credentials
          items:
            - key: .dockerconfigjson
              path: config.json
'''
        }
    }
    environment {
        IMAGE_NAME = "${{ values.name }}"
        SONAR_PROJECT_KEY = "${{ values.name }}"
        // below needed?
        // GH_CREDS = credentials('gh-creds')
        // use GH token
        // GITHUB_TOKEN = credentials('github-token')
        SONAR_HOST_URL = credentials('sonarqube-host')
        SONAR_TOKEN = credentials('sonarqube-token')
    }
    stages {
        // stage('Run Tests') {
        //     steps {
        //         echo 'Running tests...'
        //         // insert tests
        //         sh 'echo tests successful'
        //     }
        // }
        // stage('Scan') {
        //     steps {
        //         script {
        //             def scannerHome = tool 'sonar-scanner'
        //             withSonarQubeEnv(installationName: 'sq1') {
        //                 sh "${scannerHome}/bin/sonar-scanner \
        //                 -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
        //                 -Dsonar.sources=.   "
        //             }
        //         }
        //     }
        // }
        // sonarqube webhook for quality gate
        // quality gate
        // stage('Quality Gate') {
        //     steps {
        //                 waitForQualityGate abortPipeline: true
        //     }
        // }
        stage('Build with Kaniko') {
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
            /kaniko/executor --context `pwd` --destination eastyler/${{ values.name }}:latest
          '''
                }
            }
        }
                // withKubeCredentials([
                //     [credentialsId: 'kubeconfig']
                // ]) {
                    // set up helm? kubectl?
                    // use kc-playground  - minikube for dev

        // }
        // stage('Helm setup') {
        //     steps {
        //         // install helm and kubectl
        //     }
        // }
        // stage('Helm setup') {
        //     steps {
        //         container('kaniko') {
        //             script {
        //                 // Install Helm
        //                 sh '''
        //                 curl -fsSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
        //                 '''
        //                 // Install kubectl
        //                 sh '''
        //                 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
        //                 chmod +x kubectl
        //                 mv kubectl /usr/local/bin/
        //                 '''
        //             }
        //         }
        //     }
        // }
        stage('Helm chart deployment') {
            steps {
                container('helm-kubectl') {
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh '''
                        helm upgrade --install ${IMAGE_NAME} ./helm/generic \
                        --namespace ${NAMESPACE} \
                        --create-namespace
                        '''
                    }
                }
            }
        }
}
}
// pipeline {
//     agent {
//         kubernetes {
//             yaml '''
// kind: Pod
// spec:
//   containers:
//   - name: kaniko
//     image: gcr.io/kaniko-project/executor:debug
//     imagePullPolicy: Always
//     command:
//     - sleep
//     args:
//     - 9999999
//     volumeMounts:
//       - name: jenkins-docker-cfg
//         mountPath: /kaniko/.docker
//   volumes:
//   - name: jenkins-docker-cfg
//     projected:
//       sources:
//       - secret:
//           name: docker-credentials
//           items:
//             - key: .dockerconfigjson
//               path: config.json
// '''
//         }
//     }
//     environment {
//         IMAGE_NAME = "${{ values.name }}"
//         SONAR_PROJECT_KEY = "${{ values.name }}"
//         // below needed?
//         // GH_CREDS = credentials('gh-creds')
//         // use GH token
//         // GITHUB_TOKEN = credentials('github-token')
//         SONAR_HOST_URL = credentials('sonarqube-host')
//         SONAR_TOKEN = credentials('sonarqube-token')
//     }
//     stages {
//         // stage('Run Tests') {
//         //     steps {
//         //         echo 'Running tests...'
//         //         // insert tests
//         //         sh 'echo tests successful'
//         //     }
//         // }
//         // stage('Scan') {
//         //     steps {
//         //         script {
//         //             def scannerHome = tool 'sonar-scanner'
//         //             withSonarQubeEnv(installationName: 'sq1') {
//         //                 sh "${scannerHome}/bin/sonar-scanner \
//         //                 -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
//         //                 -Dsonar.sources=.   "
//         //             }
//         //         }
//         //     }
//         // }
//         // sonarqube webhook for quality gate
//         // quality gate
//         // stage('Quality Gate') {
//         //     steps {
//         //                 waitForQualityGate abortPipeline: true
//         //     }
//         // }
//         stage('Build with Kaniko') {
//             steps {
//                 container(name: 'kaniko', shell: '/busybox/sh') {
//                     sh '''#!/busybox/sh
//             /kaniko/executor --context `pwd` --destination eastyler/${{ values.name }}:latest
//           '''
//                 }
//             }
//         }
//                 // withKubeCredentials([
//                 //     [credentialsId: 'kubeconfig']
//                 // ]) {
//                     // set up helm? kubectl?
//                     // use kc-playground  - minikube for dev

//         // }
//         // stage('Helm setup') {
//         //     steps {
//         //         // install helm and kubectl
//         //     }
//         // }
//         // stage('Helm setup') {
//         //     steps {
//         //         container('kaniko') {
//         //             script {
//         //                 // Install Helm
//         //                 sh '''
//         //                 curl -fsSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
//         //                 '''
//         //                 // Install kubectl
//         //                 sh '''
//         //                 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
//         //                 chmod +x kubectl
//         //                 mv kubectl /usr/local/bin/
//         //                 '''
//         //             }
//         //         }
//         //     }
//         // }
//         stage('Helm chart deployment') {
//             steps {
//                 withKubeConfig([credentialsId: 'kubeconfig']) {
//                     sh '''
//                         helm upgrade --install ${{ values.name }} ./helm/generic \
//                         --namespace ${{ values.namespace }} \
//                         --create-namespace
//                     '''
//                 }
//             }
//     // }
//     // stage('Helm chart deployment') {
//     //     steps {
//     //         container('kaniko') {
//     //             withKubeConfig([credentialsId: 'kubeconfig']) {
//     //                 sh '''
//     //                 helm upgrade --install ${IMAGE_NAME} ./helm/generic \
//     //                 --namespace ${values.namespace} \
//     //                 --create-namespace
//     //                 '''
//     //             }
//     //         }
//     //     }
//     // }
//     }
// }
// }
