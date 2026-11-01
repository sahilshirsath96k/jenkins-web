
def run(cmd) {
    if (isUnix()) {
        sh cmd
    } else {
        bat cmd
    }
}

def awsRun(cmd) {
    withCredentials([
        string(
            credentialsId: 'aws-key-id',
            variable: 'AWS_ACCESS_KEY_ID'
        ),
        string(
            credentialsId: 'aws-secret',
            variable: 'AWS_SECRET_ACCESS_KEY'
        )
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
    awsRun(
        "aws ssm send-command --region ${env.REGION} " +
        "--instance-ids ${env.INSTANCE} " +
        "--document-name AWS-RunShellScript " +
        "--parameters file://deploy.json"
    )
}
