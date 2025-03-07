pipeline{
      // 定义groovy脚本中使用的环境变量
      environment{
        // 将构建任务中的构建参数转换为环境变量
		IMAGE = sh(returnStdout: true,script: 'echo registry.$image_region.aliyuncs.com/$image_namespace/$image_reponame:$image_tag').trim()
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
        stage('Git'){
          steps{
            git branch: '${BRANCH}', credentialsId: '', url: 'https://github.com/Huangys1992/jenkins-demo.git'
          }
        }

        // 添加第二个stage， 运行源码打包命令
        stage('Package'){
           steps{
               container("composer") {
                   sh "composer install"
               }
           }
         }


        // 添加第三个stage, 运行容器镜像构建和推送命令， 用到了environment中定义的groovy环境变量
         stage('Image Build And Publish'){
           steps{
               container("kaniko") {
                   sh "kaniko -f `pwd`/Dockerfile -c `pwd` --destination=${IMAGE} --skip-tls-verify"
               }
           }
         }

        // 添加第四个stage, 部署应用到指定k8s集群
        stage('Deploy to Kubernetes') {
          steps {
            container('kubectl') {
			  sh "sed -i  's/IMAGE/${IMAGE}/g' application-demo.yaml"
			  sh "kubectl apply -f  application-demo.yaml"
            }
          }
        }
      }
}
