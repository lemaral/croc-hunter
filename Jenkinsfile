#!/usr/bin/groovy
slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
podTemplate(
    label: 'jenkins-pipeline',
    containers: [
        containerTemplate(name: 'jnlp', image: 'jenkinsci/jnlp-slave:2.62', args: '${computer.jnlpmac} ${computer.name}', workingDir: '/home/jenkins', resourceRequestCpu: '200m', resourceLimitCpu: '200m', resourceRequestMemory: '256Mi', resourceLimitMemory: '256Mi'),
        containerTemplate(name: 'docker', image: 'docker:1.12.6', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'golang', image: 'golang:1.7.5', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:v2.4.1', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.4.8', command: 'cat', ttyEnabled: true)
    ],
    volumes:[
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    ])
    {

      node('jenkins-pipeline') {
        def pwd = pwd()
        def chart_dir = "${pwd}/charts/croc-hunter"
        def app_name = 'croc-hunter'
        def imageTag = 'latest'
        def replicas = '2'
        def cpu = '1'
        def memory = '512Mi'
        def namespace = app_name
        def helm_args = "--install ${app_name} ${chart_dir} --set imageTag=${imageTag},replicas=${replicas},cpu=${cpu},memory=${memory} --namespace=${namespace}"
        def reg_name = '930379479477.dkr.ecr.eu-west-1.amazonaws.com'
        def reg_cred = 'ecr:eu-west-1:demo-ecr-push'
        def reg_repo = 'demo-ecr'

        //def git_repo = 'https://github.com/lemaral/croc-hunter'
        //git git_url
        checkout scm

        stage('build') {
          container('golang') {
            sh "go test -v -race ./..."
            sh "make bootstrap build"
          }
        }

        stage('test') {
          container('helm') {
            sh "helm version"
            sh "helm lint ${chart_dir}"
            sh "helm init"
            sh "helm upgrade --dry-run ${helm_args}"
          }
        }

        stage('push') {
          container('docker') {
            docker.withRegistry("https://${reg_name}", reg_cred) {
              def img_name = "${reg_name}/${reg_repo}:${imageTag}"
              //def img = docker.build(img_name)
              //echo "Id=${img.Id}"
              sh "docker build -t ${img_name}  ."
              def img = docker.image(img_name)
              img.push()
            }
          }
        }

        stage('deploy') {
          container('helm') {
            sh "helm upgrade ${helm_args}"
            sh "helm test ${app_name} --cleanup"
          }
        }
      }

    }
