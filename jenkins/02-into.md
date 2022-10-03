# pipeline 语法

## agent

```bash
pipeline{
  agent any
}
```

agent 指定整个 pipeline 或特定 stage 在 jenkins 环境中执行的位置，具体取决于该 agent 的位置；必须在 pipeline 块内的顶层定义，stage 级的定义是可选的。

- agent any 代表可以在任何节代理上执行 pipeline 或 stage

- agent none 代表不会为整个 pipeline 分配代理，各 stage 需要指定自己的代理

- agent {label 'name'} 具体指定节点的名称

```bash
pipeline {
  agent {
    label ‘具体一个节点label名称’
  }
}
```

还可通过 node 代码块来分别对不同节点设置不同的参数

```bash
pipeline {
    agent {
        node {
            label ‘xxx-agent-机器’
            customWorkspace "${env.JOB_NAME}/${env.BUILD_NUMBER}"
        }
    }
}
```

agent 还有 docker 和 dockerfile 两个参数，这里不再赘述

## post

post 块将在 pipeline 运行结束时执行，可以放在 pipeline 内的顶层，也可以放在 stage 里面

post 块支持多种条件指令，由 always，changed，failure，success，unstable，aborted

需要按顺序编写：

always - changed - aborted - success - unstable - failure

- always

不论 pipeline 运行的状态如何，都会执行的步骤，例如清除文件、恢复环境等代码

```bash
pipeline {
    agent {
        node {
            label ‘xxx-agent-机器’
            customWorkspace "${env.JOB_NAME}/${env.BUILD_NUMBER}"
        }
    }
    stages {
        stage (‘Build’) {
            bat “dir” // 如果jenkins安装在windows并执行这部分代码
            sh “pwd”  //这个是Linux的执行
        }
    }
    post {
        always {
            script {
                //写相关清除/恢复环境等操作代码
            }
        }
    }
}
```

- changed

当前 pipeline 运行的状态与先前完成的 pipeline 的状态不同时执行。通常是发送通知，例如最近几次都是成功，突然有一次失败

- failure

当前 pipeline 运行状态是失败，通常发送邮件、微信、钉钉等

- success 成功
- unstable 不稳定
- aborted 取消

## stages stage

一个 stages 下至少有一个 steps

stages 内定义 stage，stage 必须指定参数，作为名称

stage 内可以嵌套一个 stages，用来指定串行的子 stage

stage 内可以嵌套一个 parallel，用来指定并行的子 stage

如果由一个模块的代码，需要发布到三个环境下去测试，就可以使用并行。在该 stage 中设置 failFast true，可以指定只要有一个并行的子 stage 失败，就强行停止整个 stage

## environment

该代码块用于指定一系列键值对，当作环境变量使用。如果写在顶层，则所有 stage 都可使用，也可以单独定义在一个 stage 下。通常均使用在顶层，而局部的变量可使用 def 来定义

```bash
pipeline {
    agent any
    environment {
        unit_test = true
    }
    stages {
        stage('Example') {
            steps {
                if(unit_test == true) {
                   // call run unit test methods
                }
            }
        }
    }
}
```

## options

可定义 pipeline 的专用选项，例如 buildDiscarder 等，可由插件提供

```bash
pipeline {
    agent any
    options {
        retry(3)
    }
    stages {
        stage('Example') {
            steps {
                // call some method
            }
        }
    }
}
```

## parameters

可提供触发 pipeline 构建时的参数

```bash
parameters {
  string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '')
  booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '')
  text(name: 'Welcome_text', defaultValue: 'One\nTwo\nThree\n', description: '')
  choice(name: 'ENV_TYPE', choices: ['test', 'dev', 'product'], description: 'test means test env,….')
  name: 'FILE', description: 'Some file to upload')
  password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'A secret password')
}
```

## triggers

定义 pipeline 重新触发的方式，可用 crom pollSCM upStream 三个方式

```bash
triggers {
  pollSCM (‘H H(9-16)/2 * * 1-5)’)
}
```

第一部分 H 表示hash，记住不是表示hour，是一个散列值，含义就是在一个小时之内，会执行一次，但是这次是一个散列值，而且不会并发执行。

第二部分 H(9-16)/2，表示白天在早上9点到下午5点，每间隔2小时执行一次。

第三部分 *，每天执行

第四部分 * 表示每月执行

第五部分 1-5 表示周一到周五执行

## tool

定义自动安装和放置工具的部分 PATH，例如 maven jdk gradle

工具是在管理端中预先定义的配置

```bash
pipeline {
    agent any
    tools {
        jdk 'jdk8'
    }
    stages {
        stage('Example') {
            steps {
                sh 'java -version'
            }
        }
    }
}
```

## input

该指令允许在一个 stage 中显示提示输入等待，在 input 中写一些代码，接受用户输入

- message：在用户输入界面显示的信息
- id：可选，该用户输入的标识
- ok：可选，在 ok 按钮上的文本
- submitter：可选，指定提交者（jenkins中的用户）
- paramters：可选，可定义一些参数

通常用于在管理端的 UI 构建

## when

允许 pipeline 根据条件决定是否执行该 stage

- branch：指定分支条件 when { branch 'master' }
- environment：指定环境变量的条件 when { environment name: 'DEPLOY_TO', value: 'production' }
- experssion：当 groovy 表达式为 true 时执行 when { expression { return params.DEBUG_BUILD } }
- not：当嵌套条件为 false 时 when {not { branch 'master' } }
- allOf：当所有嵌套条件为真时执行，when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } }
- anyOf：当至少一个嵌套条件为真

```groovy
pipeline {
    agent any
    environment {
        quick_test = false
    }
    stages {
        stage('Example Build') {
            steps {
                script {
                    echo 'Hello World'
                }
            }
        }
        stage('Example Deploy') {
            when {
                expression {
                   return  (quick_test == “true” )
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

## 流程控制

pipeline 基于 groovy 开发的，支持在流程控制语句和循环代码

```groovy
pipeline {
    agent any
    stages {
        stage('flow control') {
            steps {
                script {
                    if ( 10 == 10) {
                        println "pass"
                    }else {
                        println "failed"
                    }
                }
            }
        }
    }
}
```

## 工具类

jenkins 定义了一些工具类，方便我们使用

[https://jenkins.io/doc/pipeline/steps/pipeline-utility-steps/](https://jenkins.io/doc/pipeline/steps/pipeline-utility-steps/)
