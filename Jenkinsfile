pipeline {
    agent {
		docker {
			image 'docker.1ms.run/crystallang/osc'
		}
	}
    triggers {
        cron('H 0 * * *')
    }
    stages {
        stage('Trigger Remote Services') {
            steps {
                script {
                    // 定义所有要处理的项目和包组合
                    def projects = [
                        'openEuler:OpenJDK:24.03',
                        'openEuler:OpenJDK:24.03:SP1',
                        'openEuler:OpenJDK:24.03:Next',
                        'openEuler:OpenJDK:master'
                    ]
                    
                    def packages = [
                        "hello-world"
                    ]
                    // def packages = [
                    //     'openjdk-11',
                    //     'openjdk-17',
                    //     'openjdk-21',
                    //     'openjdk-latest'
                    // ]

                    // 创建并行任务集合
                    def parallelTasks = [:]

                    projects.each { project ->
                        packages.each { pkg ->
                            // 为每个组合创建唯一任务名
                            // 提取版本标识部分
                            def parts = project.split(':')
                            def version = (parts.size() >= 3) ? parts[2..-1].join(':') : project
                            
                            // 生成符合要求的任务名称
                            String taskName = "${version}-${pkg}"

                            // 使用闭包捕获当前变量值
                            parallelTasks[taskName] = {
                                // 使用 Jenkins 凭证管理
                                withCredentials([file(credentialsId: 'peeweep-oscrc', variable: 'OSCRC')]) {
                                    sh "cat /etc/os-release"
                                    sh "osc --config $OSCRC service remoterun home:peeweep:${project} ${pkg}"
                                }
                            }
                        }
                    }

                    // 执行所有并行任务
                    parallel parallelTasks
                }
            }
        }
    }
}
