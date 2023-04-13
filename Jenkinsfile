#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        NEW_VERSION = '1.3'
    }

    
    parameters {
        choice(name: 'VERSION', choices:['1','2', '3'], description: '')
        booleanParam(name: 'executeTest', defaultValue : true, description: '')
    }
    
    tools{
        maven 'maven-3.9.0'
    }

    stages {

        // stage('Checkout') {
        //     steps {
        //         git branch: 'master', credentialsId: 'git-credentials', url: 'https://github.com/learnwithparth/springboot-jenkins.git'
        //     }
        // }
        stage('init'){
            steps{
                script{
                    gv = load "script.groovy"
                    //sh "git clone https://github.com/learnwithparth/springboot-jenkins.git"
                }
            }
        }
        stage('config'){
            steps{
                script{
                    gv.config()
                }
            }
        }
        stage('build') {
            
            steps {
                script{
                    echo 'building the application'
                    echo "Software version is ${NEW_VERSION}"
                    sh 'mvn build-helper:parse-version versions:set -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.nextMinorVersion}.\\\${parsedVersion.incrementalVersion}\\\${parsedVersion.qualifier?}' 
                    sh 'mvn clean package'
                    def version = (readFile('pom.xml') =~ '<version>(.+)</version>')[0][2]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    sh "docker build -t utsavdesai26/spring-boot:${IMAGE_NAME} ."
                    }
            }
        }
      stage('test') {
          when{  
             expression{
                 params.executeTest
             }
          }
            steps {
                script{echo 'testing the application...'
                sh 'mvn test'}
            }
        }
      stage('deploy') {
//           sshagent(['Production']) {
//               // some block
//           }
        input{
            message "Select the environment to deploy"
            ok "done"
            parameters{
                choice(name: 'Type', choices:['Dev','Test','Deploy'], description: '')
            }

        }
            steps {
                script{echo 'deploying the application...'
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
                    sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
                    sh "docker push utsavdesai26/spring-boot:${IMAGE_NAME}"
                }}

             }
        }

//         stage('commit and push'){
//             steps{
//                 script{
//                     withCredentials([usernamePassword(credentialsId: 'git-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
//                         //def encodedPassword = URLEncoder.encode("$PASSWORD",'UTF-8')
//                         sh 'git config --global user.email "learnwithparth.in@gmail.com"'
//                         sh 'git config --global user.name "learnwithparth"'
//
//                         sh 'git status'
//                         sh 'git branch'
//                         sh 'git config --list'
//
//                         //sh "git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/learnwithparth/springboot-jenkins.git"
//
//                         sh 'git add .'
//                         sh 'git status'
//                         sh 'git commit -m "version change updated"'
//                         //sh 'git push origin HEAD:master'
//                         sh "git push -u origin master"
//                         //sh "git push https://${USERNAME}:${PASSWORD}@github.com/learnwithparth/springboot-jenkins.git"
//                         }
//                 }
//             }
//         }
    }
    post{
        always{
            echo 'Executing always....'
        }
        success{
            echo 'Executing success'
        }
        failure{
            echo 'Executing failure'
        }
    }
}
