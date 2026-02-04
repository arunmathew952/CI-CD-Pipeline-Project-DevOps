

\## Install Plugins in Jenkins



1\. \*\*Eclipse Temurin Installer\*\*:

&nbsp;  - This plugin enables Jenkins to automatically install and configure the Eclipse Temurin JDK (formerly known as AdoptOpenJDK).

&nbsp;  - To install, go to Jenkins dashboard -> Manage Jenkins -> Manage Plugins -> Available tab.

&nbsp;  - Search for "Eclipse Temurin Installer" and select it.

&nbsp;  - Click on the "Install without restart" button.



2\. \*\*Pipeline Maven Integration\*\*:

&nbsp;  - This plugin provides Maven support for Jenkins Pipeline.

&nbsp;  - It allows you to use Maven commands directly within your Jenkins Pipeline scripts.

&nbsp;  - To install, follow the same steps as above, but search for "Pipeline Maven Integration" instead.



3\. \*\*Config File Provider\*\*:

&nbsp;  - This plugin allows you to define configuration files (e.g., properties, XML, JSON) centrally in Jenkins.

&nbsp;  - These configurations can then be referenced and used by your Jenkins jobs.

&nbsp;  - Install it using the same procedure as mentioned earlier.



4\. \*\*SonarQube Scanner\*\*:

&nbsp;  - SonarQube is a code quality and security analysis tool.

&nbsp;  - This plugin integrates Jenkins with SonarQube by providing a scanner that analyzes code during builds.

&nbsp;  - You can install it from the Jenkins plugin manager as described above.



5\. \*\*Kubernetes CLI\*\*:

&nbsp;  - This plugin allows Jenkins to interact with Kubernetes clusters using the Kubernetes command-line tool (`kubectl`).

&nbsp;  - It's useful for tasks like deploying applications to Kubernetes from Jenkins jobs.

&nbsp;  - Install it through the plugin manager.



6\. \*\*Kubernetes\*\*:

&nbsp;  - This plugin integrates Jenkins with Kubernetes by allowing Jenkins agents to run as pods within a Kubernetes cluster.

&nbsp;  - It provides dynamic scaling and resource optimization capabilities for Jenkins builds.

&nbsp;  - Install it from the Jenkins plugin manager.



7\. \*\*Docker\*\*:

&nbsp;  - This plugin allows Jenkins to interact with Docker, enabling Docker builds and integration with Docker registries.

&nbsp;  - You can use it to build Docker images, run Docker containers, and push/pull images from Docker registries.

&nbsp;  - Install it from the plugin manager.



8\. \*\*Docker Pipeline Step\*\*:

&nbsp;  - This plugin extends Jenkins Pipeline with steps to build, publish, and run Docker containers as part of your Pipeline scripts.

&nbsp;  - It provides a convenient way to manage Docker containers directly from Jenkins Pipelines.

&nbsp;  - Install it through the plugin manager like the others.



After installing these plugins, you may need to configure them according to your specific environment and requirements. This typically involves setting up credentials, configuring paths, and specifying options in Jenkins global configuration or individual job configurations. Each plugin usually comes with its own set of documentation to guide you through the configuration process.



\## Configure Above Plugins in Jenkins as per video



\## Pipeline 



```groovy



pipeline {

&nbsp;   agent any

&nbsp;   

&nbsp;   tools {

&nbsp;       jdk 'jdk17'

&nbsp;       maven 'maven3'

&nbsp;   }



&nbsp;   enviornment {

&nbsp;       SCANNER\_HOME= tool 'sonar-scanner'

&nbsp;   }



&nbsp;   stages {

&nbsp;       stage('Git Checkout') {

&nbsp;           steps {

&nbsp;              git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/jaiswaladi246/Boardgame.git'

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Compile') {

&nbsp;           steps {

&nbsp;               sh "mvn compile"

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Test') {

&nbsp;           steps {

&nbsp;               sh "mvn test"

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('File System Scan') {

&nbsp;           steps {

&nbsp;               sh "trivy fs --format table -o trivy-fs-report.html ."

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('SonarQube Analsyis') {

&nbsp;           steps {

&nbsp;               withSonarQubeEnv('sonar') {

&nbsp;                   sh ''' $SCANNER\_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \\

&nbsp;                           -Dsonar.java.binaries=. '''

&nbsp;               }

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Quality Gate') {

&nbsp;           steps {

&nbsp;               script {

&nbsp;                 waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 

&nbsp;               }

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Build') {

&nbsp;           steps {

&nbsp;              sh "mvn package"

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Publish To Nexus') {

&nbsp;           steps {

&nbsp;              withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {

&nbsp;                   sh "mvn deploy"

&nbsp;               }

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Build \& Tag Docker Image') {

&nbsp;           steps {

&nbsp;              script {

&nbsp;                  withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {

&nbsp;                           sh "docker build -t arunmathew952/boardshack:latest ."

&nbsp;                   }

&nbsp;              }

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Docker Image Scan') {

&nbsp;           steps {

&nbsp;               sh "trivy image --format table -o trivy-image-report.html arunmathew952/boardshack:latest "

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Push Docker Image') {

&nbsp;           steps {

&nbsp;              script {

&nbsp;                  withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {

&nbsp;                           sh "docker push arunmathew952/boardshack:latest"

&nbsp;                   }

&nbsp;              }

&nbsp;           }

&nbsp;       }

&nbsp;       stage('Deploy To Kubernetes') {

&nbsp;           steps {

&nbsp;              withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {

&nbsp;                       sh "kubectl apply -f deployment-service.yaml"

&nbsp;               }

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       stage('Verify the Deployment') {

&nbsp;           steps {

&nbsp;              withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {

&nbsp;                       sh "kubectl get pods -n webapps"

&nbsp;                       sh "kubectl get svc -n webapps"

&nbsp;               }

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;       

&nbsp;   }

&nbsp;   post {

&nbsp;   always {

&nbsp;       script {

&nbsp;           def jobName = env.JOB\_NAME

&nbsp;           def buildNumber = env.BUILD\_NUMBER

&nbsp;           def pipelineStatus = currentBuild.result ?: 'UNKNOWN'

&nbsp;           def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'



&nbsp;           def body = """

&nbsp;               <html>

&nbsp;               <body>

&nbsp;               <div style="border: 4px solid ${bannerColor}; padding: 10px;">

&nbsp;               <h2>${jobName} - Build ${buildNumber}</h2>

&nbsp;               <div style="background-color: ${bannerColor}; padding: 10px;">

&nbsp;               <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>

&nbsp;               </div>

&nbsp;               <p>Check the <a href="${BUILD\_URL}">console output</a>.</p>

&nbsp;               </div>

&nbsp;               </body>

&nbsp;               </html>

&nbsp;           """



&nbsp;           emailext (

&nbsp;               subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",

&nbsp;               body: body,

&nbsp;               to: 'jaiswaladi246@gmail.com',

&nbsp;               from: 'jenkins@example.com',

&nbsp;               replyTo: 'jenkins@example.com',

&nbsp;               mimeType: 'text/html',

&nbsp;               attachmentsPattern: 'trivy-image-report.html'

&nbsp;           )

&nbsp;       }

&nbsp;   }

}



}

```



