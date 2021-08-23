node{
    stage('checkout scm from Bitbucket'){
        git branch: 'main', url: 'https://sanjeev0407@bitbucket.org/sanjeev0407/java-source-code-maven-iwaq.git'
    }
    stage('code analysis'){
        sh '''mvn sonar:sonar \
        -Dsonar.projectKey=devops \
        -Dsonar.host.url=http://35.170.168.238:9000 \
        -Dsonar.login=1715790819ec9e16f67fdf7be54950204623fb9d'''
   }
    stage('build war file'){
        sh 'mvn clean install'
    }
    stage('artifact'){
        sh '''cp target/iwayQApp-1.0-RELEASE.war target/iwayQApp-1.0-RELEASE-$BUILD_ID.war
        curl -u admin:password -T target/iwayQApp-1.0-RELEASE-$BUILD_ID.war "http://35.170.168.238:8081/artifactory/example-repo-local"'''
    }
    stage('Deployment tomcat'){
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://35.170.168.238:8084')], contextPath: 'sample', war: '**/*.war'
    }
}
