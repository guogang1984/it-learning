jenkins_pipeline.md


````yaml
pipeline {
    agent none

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    stages {
        stage('Source') {
            // agent any
            agent {
                label 'master'
            }
            steps {
                echo "ok"
                git branch: 'master', credentialsId: '1', url: 'http://dev.topflames.com/sfjd/sfjd-service.git'
            }

        }
        stage('Build') {
            agent {
                docker {
                    /*
                     * Reuse the workspace on the agent defined at top-level of Pipeline but run inside a container.
                     * In this case we are running a container with maven so we don't have to install specific versions
                     * of maven directly on the agent
                     */
                    reuseNode true
                    image 'g127/java'
                    // args '-v $HOME:/tmp/jenkins-home'
                }
            }
            options {
                timeout(time: 30, unit: 'MINUTES')
            }
            steps {
                echo "ok"
                sh 'MAVEN_OPTS="-Duser.name=jenkins -Duser.home=/var/jenkins_home" mvn  -U -B -DskipTests clean package'
            }
        }


        stage('Quality Analysis') {
            parallel {
                // run Sonar Scan and Integration tests in parallel. This syntax requires Declarative Pipeline 1.2 or higher
                stage('Integration Test') {
                    agent any //run this stage on any available agent
                    steps {
                        echo 'Run integration tests here...'
                    }
                }
                stage('Sonar Scan') {
                    agent {
                        docker {
                            // we can use the same image and workspace as we did previously
                            reuseNode true
                            image 'g127/java'
                            args '-v $HOME:/tmp/jenkins-home'
                        }
                    }
                    environment {
                        //use 'sonar' credentials scoped only to this stage
                        SONAR = credentials('sonar')
                    }
                    steps {
                        sh 'mvn sonar:sonar -Dsonar.login=64f8001af070a772539d515ba702075ef10e6fed'
                    }
                }
            }
        }

        stage('Build and Publish Image') {
            when {
                branch 'master' //only run these steps on the master branch
            }
            steps {
                /*
                 * Multiline strings can be used for larger scripts. It is also possible to put scripts in your shared library
                 * and load them with 'libaryResource'
                 */
                sh """
                  echo ok
                  #docker build -t ${IMAGE} .
                  # docker tag ${IMAGE} ${IMAGE}:${VERSION}
                  # docker push ${IMAGE}:${VERSION}
                """
            }
        }
    }
}


pipeline {
    agent {
        docker {
            image 'g127/javac'
            args '-v /Users/gg/DevProjectFiles/SupportLibrary/.m2:/root/.m2'
        }
    }
    stages {
        stage('Source') {
            steps {
                echo "ok"
                git branch: 'master', credentialsId: '1', url: 'http://dev.topflames.com/sfjd/sfjd-service.git'
            }
            
        }
        stage('Build') {
            steps {
                echo "ok"
                sh 'mvn -B -DskipTests clean package'
            }
        }
    }
}




pipeline {

  /*
   * Run everything on an existing agent configured with a label 'docker'.
   * This agent will need docker, git and a jdk installed at a minimum.
   */
  agent {
    node {
      label 'docker'
    }
  }

  // using the Timestamper plugin we can add timestamps to the console log
  options {
    timestamps()
  }

  environment {
    //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
    IMAGE = readMavenPom().getArtifactId()
    VERSION = readMavenPom().getVersion()
  }

  stages {
    stage('Build') {
      agent {
        docker {
          /*
           * Reuse the workspace on the agent defined at top-level of Pipeline but run inside a container.
           * In this case we are running a container with maven so we don't have to install specific versions
           * of maven directly on the agent
           */
          reuseNode true
          image 'maven:3.5.0-jdk-8'
        }
      }
      steps {
        // using the Pipeline Maven plugin we can set maven configuration settings, publish test results, and annotate the Jenkins console
        withMaven(options: [findbugsPublisher(), junitPublisher(ignoreAttachments: false)]) {
          sh 'mvn clean findbugs:findbugs package'
        }
      }
      post {
        success {
          // we only worry about archiving the jar file if the build steps are successful
          archiveArtifacts(artifacts: '**/target/*.jar', allowEmptyArchive: true)
        }
      }
    }

    stage('Quality Analysis') {
      parallel {
        // run Sonar Scan and Integration tests in parallel. This syntax requires Declarative Pipeline 1.2 or higher
        stage ('Integration Test') {
          agent any  //run this stage on any available agent
          steps {
            echo 'Run integration tests here...'
          }
        }
        stage('Sonar Scan') {
          agent {
            docker {
              // we can use the same image and workspace as we did previously
              reuseNode true
              image 'maven:3.5.0-jdk-8'
            }
          }
          environment {
            //use 'sonar' credentials scoped only to this stage
            SONAR = credentials('sonar')
          }
          steps {
            sh 'mvn sonar:sonar -Dsonar.login=$SONAR_PSW'
          }
        }
      }
    }

    stage('Build and Publish Image') {
      when {
        branch 'master'  //only run these steps on the master branch
      }
      steps {
        /*
         * Multiline strings can be used for larger scripts. It is also possible to put scripts in your shared library
         * and load them with 'libaryResource'
         */
        sh """
          docker build -t ${IMAGE} .
          docker tag ${IMAGE} ${IMAGE}:${VERSION}
          docker push ${IMAGE}:${VERSION}
        """
      }
    }
  }

  post {
    failure {
      // notify users when the Pipeline fails
      mail to: 'team@example.com',
          subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
          body: "Something is wrong with ${env.BUILD_URL}"
    }
  }
}



pipeline {
    agent none 
    environment {
        APP_NAME = 'sfjd-service'
    }
    stages {
        stage('Win Build') {
            agent { label 'win' } 
            steps {
                echo "Running Win Build ${env.APP_NAME}"
                echo "Running Win Build ${APP_NAME}"
            }
        }
    }
}


pipeline {
    agent none 
    environment {
        APP_NAME = 'devops'
        BACKEND = '.'
        FRONTEND = 'ruoyi-ui'
        GATEWAY = ''
    }
    pra
    triggers {
        // http://dev:114cec77c215c0eac696115bfb9e728fab@ci.topflames.com:8080/generic-webhook-trigger/invoke
        // http://dev:114cec77c215c0eac696115bfb9e728fab@ci.topflames.com:8080/generic-webhook-trigger/invoke
        // see https://wiki.jenkins.io/display/JENKINS/Generic+Webhook+Trigger+Plugin
        GenericTrigger(
            // gitlab config
            genericVariables: [
              [key: 'ref', value: '$.ref'],
              [
               key: 'project',
               value: '$.project.name',
               expressionType: 'JSONPath', //Optional, defaults to JSONPath
               regexpFilter: '', //Optional, defaults to empty string
               defaultValue: '' //Optional, defaults to empty string
              ]
            ],
            token: '' ,
            causeString: ' Triggered on $ref_$project' ,
            printContributedVariables: true,
            printPostContent: true,
            regexpFilterText: '$ref_$project',
            // regexpFilterExpression: 'refs/heads/' + BRANCH_NAME
            regexpFilterExpression: '^(refs/heads/master)_${env.APP_NAME}'
        )
    }
    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    stages {
        stage('Source') {
            // agent any
            agent { label 'win' } 
            steps {
                echo "ok"
                //
                git branch: 'devops-master', credentialsId: '1', url: 'http://dev.topflames.com/devops-cce/devops.git'
            }
        }
        stage('Pkg nodejs') {
          // when { 
          //   anyOf { 
          //     changeset pattern: "ruoyi-ui/*", caseSensitive: true 
          //   } 
          // }
          agent { label 'win' } 
          options {
              timeout(time: 30, unit: 'MINUTES')
          }
          steps {
              echo "ok"
              bat """
                cd ${env.FRONTEND}
                npm i
              """
              bat """
                cd ${env.FRONTEND}
                npm run build:prod
              """

              bat "shx rm -rf              D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui"
              bat "shx mkdir -p            D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui"
              
              bat """
                cd ${env.FRONTEND}
                shx cp -r dist/*           D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui
              """
          }
        }
        stage('Pkg mvn') {
          // when { 
          //   anyOf { 
          //     changeset pattern: "ruoyi/*", caseSensitive: true
          //   } 
          // }
          agent { label 'win' } 
          options {
              timeout(time: 30, unit: 'MINUTES')
          }
          steps {
              bat """
                mvn  -U -B -DskipTests -Dmaven.test.skip=true clean package -Pprod
              """
              bat "shx rm -rf              D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/api"
              bat "shx mkdir -p            D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/api"
              //
              bat """
                shx cp -r **/target/*.jar      D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/api
              """
          }
        }

        stage('Deploy static resource') {
          // when { 
          //   anyOf { 
          //     changeset pattern: "ruoyi-ui/*", caseSensitive: true 
          //   } 
          // }
          agent { label 'win' } 
          steps {
              bat "shx rm -rf    D:/DevProjectFiles/ws-root/www/${env.APP_NAME}.topflames.com"
              bat "shx mkdir -p  D:/DevProjectFiles/ws-root/www/${env.APP_NAME}.topflames.com"
              bat """
                shx cp -r D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui/*  D:/DevProjectFiles/ws-root/www/${env.APP_NAME}.topflames.com
              """
          }
        }
        // stage('Deploy Tomcat') {
        //   // when { 
        //   //   anyOf { 
        //   //     changeset pattern: "ruoyi/*", caseSensitive: true 
        //   //   } 
        //   // }
        //   agent { label 'win' } 
        //   steps {
        //       bat "shx rm -rf    D:/DevProjectFiles/ws-root/webapps-18081/${env.APP_NAME}"
        //       bat "shx mkdir -p  D:/DevProjectFiles/ws-root/webapps-18081/${env.APP_NAME}"
        //       bat """
        //         shx cp -r D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/api/*  D:/DevProjectFiles/ws-root/webapps-18081/${env.APP_NAME}
        //       """
        //   }
        // }

        // stage('Deploy:prod static resource ') {
        //   agent { label 'master' } 
        //   steps {
        //       sh "echo processing prod..."
        //       sh " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //           rsync --no-iconv -avzP --progress --delete \
        //                 --password-file=/home/dev/DevProjectFiles/ws-conf/rsync/downloadUser.pas \
        //                 --exclude-from=/home/dev/DevProjectFiles/ws-conf/rsync/exclude.list \
        //                 rsync://downloadUser@release.topflames.com:873/release/${env.APP_NAME} \
        //                 /home/dev/DevProjectFiles/ws-release/ \
        //       "
        //       sh " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //           rsync --no-iconv -avzP --progress --delete \
        //                 --exclude-from=/home/dev/DevProjectFiles/ws-conf/rsync/exclude.list \
        //                 /home/dev/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui/ \
        //                 /home/dev/DevProjectFiles/ws-root/www/www.app.com/ \
        //       "
        //       sh "echo finish prod..."
        //   }
        // }
        // stage('Deploy:prod Tomcat') {
        //   agent { label 'master' } 
        //   steps {
        //       sh "echo processing prod..."
        //       sh  "ssh -i /var/jenkins_home/.ssh/agent  -p 18022 dev@58.49.165.253 \"docker rm -fv web1 || /bin/true\" "
        //       sh " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //           rsync --no-iconv -avzP --progress --delete \
        //                 --password-file=/home/dev/DevProjectFiles/ws-conf/rsync/downloadUser.pas \
        //                 --exclude-from=/home/dev/DevProjectFiles/ws-conf/rsync/exclude.list \
        //                 rsync://downloadUser@release.topflames.com:873/release/${env.APP_NAME} \
        //                 /home/dev/DevProjectFiles/ws-release/ \
        //       "
        //       sh " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //           rsync --no-iconv -avzP --progress --delete \
        //                 --exclude-from=/home/dev/DevProjectFiles/ws-conf/rsync/exclude.list \
        //                 /home/dev/DevProjectFiles/ws-release/${env.APP_NAME}/target/api/ \
        //                 /home/dev/DevProjectFiles/ws-root/webapps-18081/${env.APP_NAME} \
        //       "
        //       sh  " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //                 docker-compose -f /home/dev/DevProjectFiles/ws-docker/docker-compose-web1.yml up -d \
        //       "
        //       sh "echo finish prod..."
        //   }
        // }
    }
}





pipeline {
    agent none 
    environment {
        APP_NAME = 'devops'
        BACKEND = '.'
        FRONTEND = 'ruoyi-ui'
        GATEWAY = ''
    }
    triggers {
        // http://dev:114cec77c215c0eac696115bfb9e728fab@ci.topflames.com:8080/generic-webhook-trigger/invoke
        // http://dev:114cec77c215c0eac696115bfb9e728fab@ci.topflames.com:8080/generic-webhook-trigger/invoke
        // see https://wiki.jenkins.io/display/JENKINS/Generic+Webhook+Trigger+Plugin
        GenericTrigger(
            // gitlab config
            genericVariables: [
              [key: 'ref', value: '$.ref'],
              [
               key: 'project',
               value: '$.project.name',
               expressionType: 'JSONPath', //Optional, defaults to JSONPath
               regexpFilter: '', //Optional, defaults to empty string
               defaultValue: '' //Optional, defaults to empty string
              ]
            ],
            token: '' ,
            causeString: ' Triggered on $ref_$project' ,
            printContributedVariables: true,
            printPostContent: true,
            regexpFilterText: '$ref_$project',
            // regexpFilterExpression: 'refs/heads/' + BRANCH_NAME
            regexpFilterExpression: '^(refs/heads/master)_${env.APP_NAME}'
        )
    }
    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    stages {
        stage('Source') {
            // agent any
            agent { label 'win' } 
            steps {
                echo "ok"
                //
                git branch: 'develop', credentialsId: '1', url: 'http://dev.topflames.com/devops-cce/devops.git'
            }
        }
        stage('Pkg nodejs') {
          // when { 
          //   anyOf { 
          //     changeset pattern: "ruoyi-ui/*", caseSensitive: true 
          //   } 
          // }
          agent { label 'win' } 
          options {
              timeout(time: 30, unit: 'MINUTES')
          }
          steps {
              echo "ok"
              bat """
                cd ${env.FRONTEND}
                npm i
              """
              bat """
                cd ${env.FRONTEND}
                npm run build:prod
              """

              bat "shx rm -rf              D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui"
              bat "shx mkdir -p            D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui"
              
              bat """
                cd ${env.FRONTEND}
                shx cp -r dist/*           D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui
              """
          }
        }
        stage('Pkg mvn') {
          // when { 
          //   anyOf { 
          //     changeset pattern: "ruoyi/*", caseSensitive: true
          //   } 
          // }
          agent { label 'win' } 
          options {
              timeout(time: 30, unit: 'MINUTES')
          }
          steps {
              bat """
                mvn  -U -B -DskipTests -Dmaven.test.skip=true clean package -Pprod
              """
              bat "shx rm -rf              D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/api"
              bat "shx mkdir -p            D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/api"
              //
              bat """
                shx cp -r **/target/*.jar      D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/api
              """
          }
        }

        stage('Deploy static resource') {
          // when { 
          //   anyOf { 
          //     changeset pattern: "ruoyi-ui/*", caseSensitive: true 
          //   } 
          // }
          agent { label 'win' } 
          steps {
              bat "shx rm -rf    D:/DevProjectFiles/ws-root/www/${env.APP_NAME}.topflames.com"
              bat "shx mkdir -p  D:/DevProjectFiles/ws-root/www/${env.APP_NAME}.topflames.com"
              bat """
                shx cp -r D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui/*  D:/DevProjectFiles/ws-root/www/${env.APP_NAME}.topflames.com
              """
          }
        }
        // stage('Deploy Tomcat') {
        //   // when { 
        //   //   anyOf { 
        //   //     changeset pattern: "ruoyi/*", caseSensitive: true 
        //   //   } 
        //   // }
        //   agent { label 'win' } 
        //   steps {
        //       bat "shx rm -rf    D:/DevProjectFiles/ws-root/webapps-18081/${env.APP_NAME}"
        //       bat "shx mkdir -p  D:/DevProjectFiles/ws-root/webapps-18081/${env.APP_NAME}"
        //       bat """
        //         shx cp -r D:/DevProjectFiles/ws-release/${env.APP_NAME}/target/api/*  D:/DevProjectFiles/ws-root/webapps-18081/${env.APP_NAME}
        //       """
        //   }
        // }

        // stage('Deploy:prod static resource ') {
        //   agent { label 'master' } 
        //   steps {
        //       sh "echo processing prod..."
        //       sh " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //           rsync --no-iconv -avzP --progress --delete \
        //                 --password-file=/home/dev/DevProjectFiles/ws-conf/rsync/downloadUser.pas \
        //                 --exclude-from=/home/dev/DevProjectFiles/ws-conf/rsync/exclude.list \
        //                 rsync://downloadUser@release.topflames.com:873/release/${env.APP_NAME} \
        //                 /home/dev/DevProjectFiles/ws-release/ \
        //       "
        //       sh " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //           rsync --no-iconv -avzP --progress --delete \
        //                 --exclude-from=/home/dev/DevProjectFiles/ws-conf/rsync/exclude.list \
        //                 /home/dev/DevProjectFiles/ws-release/${env.APP_NAME}/target/ui/ \
        //                 /home/dev/DevProjectFiles/ws-root/www/www.app.com/ \
        //       "
        //       sh "echo finish prod..."
        //   }
        // }
        // stage('Deploy:prod Tomcat') {
        //   agent { label 'master' } 
        //   steps {
        //       sh "echo processing prod..."
        //       sh  "ssh -i /var/jenkins_home/.ssh/agent  -p 18022 dev@58.49.165.253 \"docker rm -fv web1 || /bin/true\" "
        //       sh " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //           rsync --no-iconv -avzP --progress --delete \
        //                 --password-file=/home/dev/DevProjectFiles/ws-conf/rsync/downloadUser.pas \
        //                 --exclude-from=/home/dev/DevProjectFiles/ws-conf/rsync/exclude.list \
        //                 rsync://downloadUser@release.topflames.com:873/release/${env.APP_NAME} \
        //                 /home/dev/DevProjectFiles/ws-release/ \
        //       "
        //       sh " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //           rsync --no-iconv -avzP --progress --delete \
        //                 --exclude-from=/home/dev/DevProjectFiles/ws-conf/rsync/exclude.list \
        //                 /home/dev/DevProjectFiles/ws-release/${env.APP_NAME}/target/api/ \
        //                 /home/dev/DevProjectFiles/ws-root/webapps-18081/${env.APP_NAME} \
        //       "
        //       sh  " \
        //           ssh -i /var/jenkins_home/.ssh/agent -p 18022 dev@58.49.165.253 \
        //                 docker-compose -f /home/dev/DevProjectFiles/ws-docker/docker-compose-web1.yml up -d \
        //       "
        //       sh "echo finish prod..."
        //   }
        // }
    }
}

````

````yaml
    post {
        success {
            sendMail('SUCCESSFUL')
        }
        failure {
            sendMail('FAILED')
        }
    }
````

````yaml
@NonCPS
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ""
    for (changeSet in currentBuild.changeSets) {
        for (entry in changeSet.items) {
            // commitId, timestamp, msg, author
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += " - ${truncated_msg} [${entry.author}]\n"
        }
    }
    if (!changeString) {
        changeString = " - No new changes"
    }
    return changeString
}
````

````yaml
def sendNotification(String status = 'SUCCESSFUL') {
    def changeString = getChangeString()
    def statusText = (status == 'SUCCESSFUL')?'【已完成】':'【失败】！'
    def emailSubject = "流水线" + currentBuild.projectName + "("+ currentBuild.fullDisplayName + ")"+ statusText
    def emailBody = """
        <p>更新信息：<a href="${repositoryUrl}/commit/${repositoryCommit}">${changeString}</a>.</p>
    """
    echo """
      subject:${emailSubject}
      body   :${emailBody}
    """
    // def repositoryCommit = ''
    // def repositoryUrl = ''
    // def commitMessage = ''
    // try {
    //     repositoryCommit = "${GIT_COMMIT}" 
    //     repositoryUrl = sh(returnStdout: true, script: 'git config --get remote.origin.url | sed -e "s/.git$//" | xargs')
    //     commitMessage = sh(returnStdout: true, script: 'git log --format=%B -n 1 ${repositoryCommit} | xargs')
    // } catch(Exception e) {
    //     println("${e}")
    // }

    // def emailMessage = ""
    // if(repositoryCommit) {
    //     emailMessage = """
    //         <p>更新信息：<a href="${repositoryUrl}/commit/${repositoryCommit}">${commitMessage}</a>.</p>
    //     """
    // }

    // emailext (
    //     subject: emailSubject,
    //     body: emailMessage,
    //     to: "dev@topflames.com",
    //     recipientProviders: [[$class: 'CulpritsRecipientProvider']],
    //     mimeType: "text/html"
    // )
}
````

