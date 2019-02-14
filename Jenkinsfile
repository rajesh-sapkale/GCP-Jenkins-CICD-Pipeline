def slavepod = "slavepod-${UUID.randomUUID().toString()}"

podTemplate(label: slavepod, yaml: """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: sample
spec:
  containers:
  - name: maven
    image: maven:3.6.0-jdk-8-alpine
    command:
    - cat
    tty: true
    volumeMounts:
    - name: context
      mountPath: /home
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /secret
      - name: context
        mountPath: /home
    env:
      - name: GOOGLE_APPLICATION_CREDENTIALS
        value: /secret/kaniko-secret.json

  volumes:
    - name: kaniko-secret
      secret:
        secretName: kaniko-secret
    - name: context
      emptyDir: {}
"""
  ) {
  node(slavepod) {
    def server = Artifactory.server "Artifactory"
    // Create an Artifactory Maven instance.
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo
    container('maven') {
      stage('Git checkout') {
          git 'https://github.com/rajesh-sapkale/spring-petclinic.git'
      }
      stage('Build app') {
          // Set Artifactory repositories for dependencies resolution and artifacts deployment.
          rtMaven.deployer releaseRepo: 'gcpcicd-mvn-release', snapshotRepo: 'gcpcicd-mvn-snapshot', server: server
          // rtMaven.resolver releaseRepo: 'gcpcicd-mvn-release', snapshotRepo: 'gcpcicd-mvn-snapshot', server: server

          withEnv(['MAVEN_HOME=/usr/share/maven']) {
            buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
          }
      }
      stage('Test') {
          junit '**/target/surefire-reports/TEST-*.xml'
      }
      stage('SonarQube quality gate') {
          withSonarQubeEnv('sonarqube') {
            sh 'mvn sonar:sonar -Dsonar.projectKey=rajesh-sapkale_spring-petclinic -Dsonar.organization=rajesh-sapkale-github -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=80805a592d39b2d4e2443fc442cb77dc74532859'
         }
      }
      stage('Push to artifactory') {
            server.publishBuildInfo buildInfo
      }
      stage('context build') {
        sh 'cp **/target/*.jar **/dockerbuild/context'
        sh 'tar -C **/dockerbuild/context -zcvf context.tar.gz .'
      }
    }
  }
}
