#!groovy
// def imageVersion
 
pipeline {
   agent any
 


    parameters {
        booleanParam(name: 'buildDockerImage', defaultValue: 'false', description: 'Gibt an ob die Instanz deployed werden soll, falls es nicht der main branch ist')
    }
 
    environment {
                    pom = readMavenPom file: 'pom.xml'
                    imageVersion = "${pom.version}.${env.BUILD_NUMBER}"
    }
 
    options {
        disableConcurrentBuilds()
        timestamps()
//       gitLabConnection('Demo-Gitlab-Connection')
//        gitlabBuilds(builds: ['Build', 'Test', 'Quality'])
    }
 
    stages {
        stage("Init") {
            steps {
                script {
                    currentBuild.description = sh(
                            script: 'git show --name-only',
                            returnStdout: true
                    ).trim()
                }
            }
        }
 
        stage('Build') {
            steps {
                    sh 'echo "mvn -B clean install -f helloworld/pom.xml"'
            }
        }
       
 
        stage('BuildImage') {
            steps {
                // call buildah with Dockerfile
//                sh 'echo "buildah bud -t svd-dockerreg-prod1.svd.local/svd/jbosseap74-hello:${imageVersion} -f helloworld/Dockerfile"'
                  sh 'echo "calling sub tasks DevOpsTasks.buildImage..."'      
                build job: 'DevOpsTasks', parameters: [
                    string(name: 'appname', value: params.appname),
                    string(name: 'environment', value: params.environment)
                ]
            }
 
        stage('PushImage') {
            steps {
                // push image to registry
                withCredentials([usernamePassword(credentialsId: 'registryUser', passwordVariable: 'password', usernameVariable: 'username')]){
                    sh '''
                    echo "buildah login -u ${username} -p ${password} svd-dockerreg-prod1.svd.local"
                    echo "buildah push svd-dockerreg-prod1.svd.local/svd/jbosseap74-hello:${imageVersion} docker://svd-dockerreg-prod1.svd.local/svd/jbosseap74-hello:${imageVersion}"
                    '''
                }
               
            }
           
        }
 

        stage('TriggerGitOps') {
            steps {
           
                withCredentials([usernamePassword(credentialsId: 'configmgmtUser', passwordVariable: 'password', usernameVariable: 'username')]){
 
                sh '''
                echo "git clone https://${username}:${password}@gitlab-prod1.svd.local/svd/cloud/openshift/config-mgmt/helloworld.git cfgmgt"
                echo "cat cfgmgt/env/01-dev/version.yaml.template | sed -e 's/XXXIMAGEVERSIONXXX/'"$imageVersion"'/g' > cfgmgt/env/01-dev/version.yaml"
                echo "cd cfgmgt"
                echo "git add env/01-dev/version.yaml"
                echo "git commit -m 'Update version ${imageVersion} in version.yaml'"
                echo "git push origin main"
                '''
                }
               
            }
        }
 
    }
    post {
        unsuccessful {
            script {
                emailext(
                        body: "Please go to ${env.BUILD_URL}/console for more details.",
                        to: emailextrecipients([developers()]),
                        subject: "Compile-Pipeline Status is ${currentBuild.result}. ${env.BUILD_URL}"
                )
            }
        }
        always {
            script {
                cleanWs()
            }
        }
    }
}
 
 