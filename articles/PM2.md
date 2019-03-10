## PM2

PM2 是一个带有负载均衡功能的Node应用的进程管理器。由于 node 本身是单线程的，特点就是所有的方法串联执行，并且 node 没有能力像 java 一样，能够单独创建一个新的线程来实现异步的操作，对于现代 CPU 越来越多核话来说，是十分不利的。

安装：

```javascript
npm install -g pm2
```

#### 集群模式

集群模式使得 node 应用不需要修改任何代码就能够覆盖所有的CPU，

#### Process File

PM2 是一个进程管理器，它管理着你的应用的所有状态，所以，你可以开始（start）、结束（stop）、重启（restart）或删除（delete）这些进程。

```javascript
// 开始
pm2 start app.js --name "my-test"
// 停止(仅仅是停止这个进程，进程并没有消失，状态变更： online -> stopped)
pm2 stop my-test
// 重启(从PM2 2.1.x开始，重启是及时的，这意味着维护升级并不影响你的进程)
pm2 restart my-test
// 删除（应用从 PM2进程列表中删除）
pm2 delete my-test
// 注意：从 PM2 2.4.0 开始，你可以使用正则的方式来操作（restart、delete、stop、reload），但是，仅只能restart http-1、http-2，http-3暂时还不可以。这些正则表达式只能用来测试应用名称，不能来判定id
pm2 restart /http-[1, 2]/
```

进程列表：

```javascript
// 进程列表
pm2 [list|ls|l|status]
// 获取进程详细信息
pm2 show <id|name>
// 进程排序
pm2 list --sort [name|id|pid|memory|cpu|status|uptime][:asc|desc]
```

更多：

```javascript
# Fork mode
pm2 start app.js --name my-api # Name process

# Cluster mode
pm2 start app.js -i 0        # Will start maximum processes with LB depending on available CPUs
pm2 start app.js -i max      # Same as above, but deprecated.
pm2 scale app +3             # Scales `app` up by 3 workers
pm2 scale app 2              # Scales `app` up or down to 2 workers total

# Listing

pm2 list               # Display all processes status
pm2 jlist              # Print process list in raw JSON
pm2 prettylist         # Print process list in beautified JSON

pm2 describe 0         # Display all informations about a specific process

pm2 monit              # Monitor all processes

# Logs

pm2 logs [--raw]       # Display all processes logs in streaming
pm2 flush              # Empty all log files
pm2 reloadLogs         # Reload all logs

# Actions

pm2 stop all           # Stop all processes
pm2 restart all        # Restart all processes

pm2 reload all         # Will 0s downtime reload (for NETWORKED apps)

pm2 stop 0             # Stop specific process id
pm2 restart 0          # Restart specific process id

pm2 delete 0           # Will remove process from pm2 list
pm2 delete all         # Will remove all processes from pm2 list

# Misc

pm2 reset <process>    # Reset meta data (restarted time...)
pm2 updatePM2          # Update in memory pm2
pm2 ping               # Ensure pm2 daemon has been launched
pm2 sendSignal SIGUSR2 my-app # Send system signal to script
pm2 start app.js --no-daemon
pm2 start app.js --no-vizion
pm2 start app.js --no-autorestart
```

详见[PM2官网](http://pm2.keymetrics.io/docs/usage/quick-start/)

#### ecosystem 文件

ecosystem.config.js 文件回聚合应用所有的配置及环境变量。这个文件将导出项目所有的配置。

```javascript
pm2 ecosystem
```

这里以 这里以**PM2 3.3.1** 为例，低版本的可见[Generate configuration](http://pm2.keymetrics.io/docs/usage/application-declaration/#generate-configuration)]([http://pm2.keymetrics.io/docs/usage/application-declaration/](http://pm2.keymetrics.io/docs/usage/application-declaration/))

它主要有两部分组成：

```javascript
module.exports = {
  apps: [{}, {}],
  deploy: {}
}
```

1. apps
2. 这里简要介绍几个主要配置项：
```javascript
module.exports = {
    apps: [{
        {
            name: 'my-test',  // 项目名称
            script: './bin/www', // pm2 启动脚本路径
            cwd: __dirname, // 项目启动目录
        	args: 'one two', // 包含所有通过CLI脚本传递的参数
    		instances: 1, // 启动的应用程序实例数量
    		autorestart: true, // 默认为 true，如果为false，PM2 在项目崩溃或结束时不会重启
    		max_memory_restart: '1G', // 超过指定的内存时，项目将会重启
            watch: true, // 监听，当项目代码变更时，项目将会重启
            output : "/wls/applogs/rtlog/out.log", // 错误日志路径
            error : "/wls/applogs/rtlog/err.log", // 输出日志路径
            env: { // 项目环境变量
                "PORT": 30001,
                "NODE_ENV": "development",
        		"ID": 42
            },
           // 格式为^env_\S*$，当重启时，注入PM2
           // 例如：pm2 start ecosystem.config.js --watch --env production
    		env_production: {
           		"PORT": 30002,
      			"NODE_ENV": 'production'
    		} 
        }
    }]
}
```

1. deploy
2. 部署环境
3. 主要配置：
```javascript
module.exports = {
  deploy : {
    production : {
      user : 'node', // 登录远程服务器的用户名
      host : 'X.X.X.X', // 远程服务器的IP或hostname，此处可以是数组同步部署多个服务器
      ref  : 'origin/master', // 远端名称及分支名
      repo : 'git@github.com:repo.git', // git仓库地址
      path : '/var/www/production', // 远程服务器部署目录，需要填写user具备写入权限的目录
      'post-deploy' : 'npm install && pm2 reload ecosystem.config.js --env production' // 部署后需要执行的命令
    }
    staging: {},
    development: {}
  }
};
```


更多配置详见 [Ecosystem file reference](https://pm2.io/doc/en/runtime/reference/ecosystem-file/)

#### 特点

* 内建负载均衡（使用Node cluster 集群模块、子进程，可以参考朴灵的《深入浅出node.js》一书第九章）
* 线程守护，keep alive
* 0秒停机重载，维护升级的时候不需要停机.
* 现在 Linux (stable) & MacOSx (stable) & Windows (stable).多平台支持
* 停止不稳定的进程（避免无限循环）
* 控制台检测
* 提供 HTTP API
* 远程控制和实时的接口API ( Nodejs 模块,允许和PM2进程管理器交互 ）

**更多内容后续整理**