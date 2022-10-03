# pipeline 高级方法

## Pipeline Basic Steps

[https://jenkins.io/doc/pipeline/steps/workflow-basic-steps/](https://jenkins.io/doc/pipeline/steps/workflow-basic-steps/)

基础的 pipeline 组件，默认自动安装

### deleteDir

没有参数，清空 worksspace。可以用在 post 中进行的清理工作

```groovy
stage('Clear Workspace'){
    steps{
        script{
            sh "ls -al ${env.WORKSPACE}"
            deleteDir()
            sh "ls -al ${env.WORKSPACE}"
        }
    }
}
```

### dir

改变当前的工作目录，在 dir 语句块里执行的其他路径或者相对路径，都是和 dir 设置的路径相关

```groovy
pipeline{

  agent any
  stages{
    stage("dir") {
      steps{
          println env.WORKSPACE
          dir("${env.WORKSPACE}/testdata"){
            sh "pwd"
          }
      }
    }
  }
}
```

### echo error

echo 是打印 info debug 级别的日志

如果执行 error("message") 的方法，那么该任务就会退出并显示失败

### fileExists

返回 true 文件存在

```groovy
stages{
    stage("fileExists") {
        steps{
            script {
                json_file = "${env.WORKSPACE}/testdata/test_json.json"
                if(fileExists(json_file) == true) {
                    echo("json file is exists")
                }else {
                    error("here haven't find json file")
                }
            }
        }
    }
}

```

### isUnix

判断 node 环境是 linux/mac 还是 windows

### pwd

可以在 windows 上调用该方法，效果和 linux 的 pwd 一样

### mail

配置好 smtp 后，通过该方法可以发送邮件

必选参数可以由 subject、body，可选参数有：bcc，cc，charset，from，mimeType，replyTo，to

```groovy
steps{
    script {
        mail to: '570375381@qq.com',
        subject: "Running Pipeline: ${currentBuild.fullDisplayName}",
        body: "Something is wrong with ${env.BUILD_URL}"
    }
}
```

### readFile

从 workspace 路径下读取一个文件，返回文件的字符串

参数 file，encoding

```groovy
steps{
    script {
        json_file = "${env.WORKSPACE}/testdata/test_json.json"
        file_contents = readFile json_file
        println file_contents
    }
}
```

### retry sleep

写法 retry(3){...}  sleep 2

retry 单位是次数，sleep 单位是秒

retry 只能出现在异常的地方，否则只能跑一边就结束

```groovy
steps{
    script{

        try {
            retry(3) {
                println "here we are test retry fuction"
                sleep 5
                println 10/0

            }
        }catch (Exception e) {
            println e
        }
    }
}
```

### timeout

如果超过时间，stage 就会被取消，状态为 aborted

```groovy
steps{
    script {
        timeout(1) {
            sh('java -version')
            //sleep 61
        }
    }
}
```

### waitUntil

代码会无限循环执行，直到返回 true

```groovy
steps{
    script {
        timeout(5) {
            waitUntil {
                script {
                    // 监控某一个服务是否启动完成代码
                }
            }
        }
    }
}
```

### withEnv

添加多个环境变量

```groovy
stage("init") {
    steps{
        withEnv(['java_home=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-3.b13.el6_10.i386/jre']) {
            sh("$java_home/bin/java -version")
        }
        script {
            println "test with withEnv feature"
        }
    }
}
```

### writeFile

```groovy
stage("writeFile demo") {
    steps{
        script {
            write_file_path = "${env.WORKSPACE}/testdata/write.txt"
            file_contents = "Hello Anthony!! 这是一个测试例子"
            //write into write.txt
            writeFile file: write_file_path, text: file_contents, encoding: "UTF-8"
            // read file and print it out
            fileContents = readFile file: write_file_path, encoding: "UTF-8"
            println fileContents
        }
    }
}
```

### git scm 插件

通常第一个 stage 的名称有一个 step 是 Declarative: Checkout SCM，这个是 pipeline 给自动加上的。如果需要检出其他仓库的代码，就需要使用相关的 git 命令

```groovy
stage("git checkout") {
    steps{
        script {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']],
                      userRemoteConfigs: [[credentialsId: '6f4fa66c-eb02-46dc-a4b3-3a232be5ef6e',
                                           url: 'https://github.com/Anthonyliu86/HelloWorld.git']]])
        }
    }
}
```

## Pipeline Utility Steps

该插件提供常用方法

### findFiles

在工作目录下，根据字符串规则去查找文件，如果有匹配的，返回的是一个 file 数组对象

可选指令 blob 参数是字符串，字符串为 ant 类型的匹配模式

```groovy
//找出左右 .log 结尾的文件
def files = findFiles(glob:'**/*.log')
echo files[0].name
```

### readYaml writeYaml

```groovy
def read_yaml_file(yaml_file) {
  def datas = ""
  if(yaml_file.toString().endsWith(".yml")){
    datas = readYaml file : yaml_file

  }else {
    datas = readYaml text : yaml_file
  }
  datas.each {
    println ( it.key + " = " + it.value )
   }
}

def write_to_yaml(map_data, yaml_path) {
  writeYaml file: yaml_path , data: map_data
}
```

### readJSON writeJSON

```groovy
def read_json_file(file_path) {
  def propMap = readJSON file : file_path
  propMap.each {
      println ( it.key + " = " + it.value )
  }
}

def read_json_file2(json_string) {
  def propMap = readJSON text : json_string
  propMap.each {
    println ( it.key + " = " + it.value )
  }
}


def write_json_to_file(input_json, tofile_path) {
  def input = ''
  if(input_json.toString().endsWith(".json")) {
    input = readJSON file : input_json
  }else {
    def jsonOutput = new JsonOutput()
        def new_json_object = jsonOutput.toJson(input_json)
    input = new_json_object
  }
  writeJSON file: tofile_path, json: input
}
```

### readProperties

```groovy
def read_properties(properties_file) {
   def props = readProperties interpolate: true, file: properties_file
   props.each {
    println ( it.key + " = " + it.value )
   }
}
```

### touch

```groovy
steps{
    script{
        touch_file = env.WORKSPACE + "/testdata/"+ env.BUILD_NUMBER +".log"
        touch touch_file
    }
}
```

## 串联多个 job

```groovy
post {
    always {
        script {
            println "Do some actins when always need."
        }
    }
    failure {
        script {
            println "Do some actins when build failed."
        }
    }
    success {
        script {
            println "Here we kickoff run job B"
            jobB = build job: 'ProjectB-pipeline-demo', propagate: false, wait: true, parameters: [
                string(name:'INPUT_JSON', value: "${json_file}")
            ]
            println jobB.getResult()
        }
    }
}

```
