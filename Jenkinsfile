
def run(cmd) {
    if (isUnix()) {
        sh cmd
    } else {
        bat cmd
    }
}

def awsRun(cmd) {
    withCredentials([
        string(credentialsId: 'aws-key-id', variable: 'AWS_ACCESS_KEY_ID'),
        string(credentialsId: 'aws-secret', variable: 'AWS_SECRET_ACCESS_KEY')
    ]) {
        run(cmd)
    }
}

def ecrLogin() {
    awsRun(
        "aws ecr get-login-password --region ${env.REGION} | " +
        "docker login --username AWS --password-stdin ${env.REGISTRY}"
    )
}

def deploy() {
    if (!env.INSTANCE?.trim() ||
        env.INSTANCE.trim().equalsIgnoreCase('none')) {
        error("Invalid EC2 instance ID: ${env.INSTANCE}")
    }

    if (!fileExists('deploy.json')) {
        error("deploy.json not found in Jenkins workspace. Add it to the GitHub repository root.")
    }

    echo "Deploying to EC2 instance: ${env.INSTANCE}"

    awsRun("""
        aws ssm send-command \
          --region ${env.REGION} \
          --instance-ids ${env.INSTANCE} \
          --document-name AWS-RunShellScript \
          --parameters file://deploy.json
    """)
}

pipeline {
    agent any

    environment {
        REGION   = 'ap-south-1'
        ACCOUNT  = '469465348676'
        REPO     = 'jenkins-web'
        INSTANCE = 'i-04e9eecd25ec854ec'

        REGISTRY = "${ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com"
        IMG      = "${REPO}:${BUILD_NUMBER}"
        REMOTE   = "${REGISTRY}/${REPO}:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'pwd && ls -la'
            }
        }

        stage('Build') {
            steps {
                script {
                    run "docker build -t ${IMG} ."
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    run "docker run --rm ${IMG} nginx -t"
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    ecrLogin()
                    run "docker tag ${IMG} ${REMOTE}"
                    run "docker push ${REMOTE}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    deploy()
                }
            }
        }
    }
}
