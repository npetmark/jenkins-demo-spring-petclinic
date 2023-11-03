pipeline {
    agent any
    
    stages {
        stage("build") {
            steps {
                slackSend message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Link>)"
                sh "./mvnw package"                
            }
        }
        stage("capture") {
            steps {
                archiveArtifacts '**/target/*.jar'
            }
        }
        stage("test & coverage") {
            steps {
                junit '**/target/surefire-reports/TEST*.xml'
                jacoco()
            }
        }

        stage("deploy") {
            agent { 
                docker { 
                    image 'nmark/petclinic'
                }
            }
            steps {
                script{
                    // withEnv(['JENKINS_NODE_COOKIE=dontKillMe']) {
                    $containerId = sh 'docker ps -q --filter "label=org.jenkinsci.plugins.pipeline.modeldefinition.agent.impl.LabelledDockerAgentTemplate"'
                    copyArtifacts filter: '**/*.jar', fingerprintArtifacts: true, projectName: 'scm-declarative', selector: upstream(), target: './target'
                    sh 'docker exec -u 0 petclinic docker cp $containerId:./target/spring-petclinic-3.1.0-SNAPSHOT.jar .'
                    sh "docker exec petclinic java -Dserver.port=9000 -jar ./spring-petclinic-3.1.0-SNAPSHOT.jar &"
                // }
            }
        }
    }
                // copyArtifacts filter: '**/*.jar', fingerprintArtifacts: true, projectName: 'scm-declarative', selector: upstream(fallbackToLastSuccessful: true), target: './target'
                    // sh 'java -Dserver.port=9000 -jar **/spring-petclinic-3.1.0-SNAPSHOT.jar & exit'
                // sh 'docker version'
    }
    post {
        always {
            emailext body: "${env.BUILD_URL}\n${currentBuild.absoluteUrl}",
                to: 'nikolay.markov@technocloudlab.com',
                recipientProviders: [previous()], 
                subject: "${currentBuild.currentResult}: Job ${env.JOB_NAME} [${env.BUILD_NUMBER}]"

            slackSend message: "Build Completed - ${currentBuild.currentResult}: ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Link>)"
        }
    }
}
