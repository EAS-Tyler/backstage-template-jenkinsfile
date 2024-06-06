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
        SONAR_HOST_URL = credentials('sonarqube-host')
        SONAR_TOKEN = credentials('sonarqube-token')
    }
    stages {
        stage('Run Tests') {
            steps {
                echo 'Running tests...'
                // insert tests
                sh 'echo tests successful'
            }
        }
        stage('Scan') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv(installationName: 'sq1') {
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=.   "
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                }
        }}
        stage('Build with Kaniko') {
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
            /kaniko/executor --context `pwd` --destination enterpriseautomation/${{ values.name }}:latest
          '''
                }
            }
        }
        stage('Helm chart deployment') {
            steps {
                container('helm-kubectl') {
                    withKubeConfig([credentialsId: 'kubeconfig']) {
                        sh '''
                        helm upgrade --install ${{ values.name }} ./helm/generic \
                        --namespace ${{ values.namespace }} \
                        --create-namespace
                        '''
                    }
                }
            }
        }
        }
    }

// kubeconfig and registry