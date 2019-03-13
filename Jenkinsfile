pipeline{
      // 定义groovy脚本中使用的环境变量
      environment{
        // 本示例中使用DEPLOY_TO_K8S变量来决定把应用部署到哪套容器集群环境中，如“Production Environment”， “Staging001 Environment”等
        IMAGE_TAG =  sh(returnStdout: true,script: 'echo $image_tag').trim()
        ORIGIN_REPO =  sh(returnStdout: true,script: 'echo $origin_repo').trim()
        REPO =  sh(returnStdout: true,script: 'echo $repo').trim()
        BRANCH =  sh(returnStdout: true,script: 'echo $branch').trim()
      }

      // 定义本次构建使用哪个标签的构建环境，本示例中为 “slave-pipeline”
      agent{
        node{
          label 'slave-pipeline'
        }
      }

      // "stages"定义项目构建的多个模块，可以添加多个 “stage”， 可以多个 “stage” 串行或者并行执行
      stages{
        // 定义第一个stage， 完成克隆源码的任务
        stage('获取代码'){
          steps{
            git branch: '${BRANCH}', credentialsId: '', url: 'https://github.com/AliyunContainerService/jenkins-demo.git'
          }
        }

        // 添加第二个stage， 运行源码打包命令
        stage('代码编译打包'){
          steps{
              container("maven") {
                  sh "mvn package -B -DskipTests"
              }
          }
        }


        // 添加第四个stage, 运行容器镜像构建和推送命令， 用到了environment中定义的groovy环境变量
        stage('镜像构建推送'){
          steps{
              container("kaniko") {
                  sh "kaniko -f `pwd`/Dockerfile -c `pwd` --destination=${ORIGIN_REPO}/${REPO}:${IMAGE_TAG}"
              }
          }
        }


        stage('应用部署到K8S集群') {
            parallel {
                stage('生产环境') {
                    when {
                        expression {
                            "$BRANCH" == "master"
                        }
                    }
                    steps {
                        container('kubectl') {
                            step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: 'https://kubernetes.default.svc.cluster.local:443', credentialsId:'k8sCertAuth', config: 'deployment.yaml',variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
                        }
                    }
                }
                stage('临时环境') {
                    when {
                        expression {
                            "$BRANCH" == "latest"
                        }
                    }
                    steps {
                        container('kubectl') {
                            step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: 'https://kubernetes.default.svc.cluster.local:443', credentialsId:'k8sCertAuth', config: 'deployment.yaml',variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
                        }
                    }
                }
            }
        }
      }
    }
