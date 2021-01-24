node {
    // config 
    def to = emailextrecipients([
            [$class: 'CulpritsRecipientProvider'],
            [$class: 'DevelopersRecipientProvider'],
            [$class: 'RequesterRecipientProvider']
    ])
    def commit_id

    // job
    try {
        stage('Preparation') {
            checkout scm
            sh "git rev-parse --short HEAD > .git/commit-id"                        
            commit_id = readFile('.git/commit-id').trim()
        }
        stage('Test') {
            nodejs(nodeJSInstallationName: 'nodejs') {
                sh 'npm install --only=dev'
                sh 'npm test'
            }
        }
        stage('Build') {
            def customImage = docker.build("nodejs-app:${commit_id}")
        }
        stage('Deploy') {
            docker.image('mysql:5').withRun('-p 8080:8080') {
                /* do things */
            }
        }

    } catch(e) {

        // mark build as failed
        currentBuild.result = "FAILURE";
        // set variables
        def subject = "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} ${currentBuild.result}"
        def content = '${JELLY_SCRIPT,template="html"}'

        // send email
        if(to != null && !to.isEmpty()) {
            emailext(body: content, mimeType: 'text/html',
                    replyTo: '$DEFAULT_REPLYTO', subject: subject,
                    to: to, attachLog: true )
        }

        // mark current build as a failure and throw the error
        throw e;
    }
}
