# 🐳 Jenkins Pipeline Setup with Docker (Python Example)

本文件教你如何在 Windows 上使用 **Docker** 快速部署 Jenkins，並運行 Python Pipeline。

---

## 🚀 Jenkins 安裝與啟動

### 🧩 PowerShell 版本

```powershell
docker run -d `
  -p 8081:8080 `
  -p 50001:50000 `
  -v "D:\docker\jenkins_home:/var/jenkins_home" `
  -v "//var/run/docker.sock:/var/run/docker.sock" `
  --name jenkins_new `
  jenkins/jenkins:lts-jdk17
```

### 💻 CMD 版本

```cmd
mkdir D:\docker\jenkins_home

docker run -d -p 8081:8080 -p 50001:50000 ^
-v D:\docker\jenkins_home:/var/jenkins_home ^
-v //var/run/docker.sock:/var/run/docker.sock ^
--name jenkins_new jenkins/jenkins:lts-jdk17
```

| 參數 | 功能 |
| ---- | ---- |
| `-p 8081:8080` | Jenkins Web 介面 Port |
| `-p 50001:50000` | Agent 連線 Port |
| `-v D:\docker\jenkins_home:/var/jenkins_home` | Jenkins 資料持久化 |
| `-v //var/run/docker.sock:/var/run/docker.sock` | 讓 Jenkins 控制宿主 Docker |
| `--name jenkins_new` | 容器名稱 |
| `jenkins/jenkins:lts-jdk17` | Jenkins LTS 官方映像 |

---

## 🌐 Jenkins 初次登入

啟動後開啟瀏覽器：  
👉 [http://localhost:8081](http://localhost:8081)

初始密碼位置：  
```
D:\docker\jenkins_home\secrets\initialAdminPassword
```

登入後選擇：  
🔹 安裝建議插件（Install suggested plugins）  
🔹 建立 admin 帳號

---

## 🔌 安裝必要插件

進入：**Manage Jenkins → Manage Plugins → Available**  
搜尋並安裝：

- Docker Pipeline  
- Docker  
- Git  
- Pipeline  
- (可選) Blue Ocean

---

## 🧰 常用 Docker / Jenkins 指令

| 指令 | 功能 |
| ---- | ---- |
| `docker ps` | 查看正在運行的容器 |
| `docker ps -a` | 查看所有容器（包含停止的） |
| `docker stop jenkins_new` | 停止 Jenkins 容器 |
| `docker rm jenkins_new` | 刪除 Jenkins 容器 |
| `docker logs jenkins_new` | 查看 Jenkins 日誌 |
| `docker exec -it jenkins_new bash` | 進入 Jenkins 容器 |
| `docker restart jenkins_new` | 重啟 Jenkins |

---

## 🧪 Jenkinsfile (Python 測試範例)

```groovy
pipeline {
    agent {
        docker {
            image 'python:3.11'
            args '-u root'
        }
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/cba542/jenkins-pipeline-demo.git'
            }
        }

        stage('Install dependencies') {
            steps {
                sh '''
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install pytest
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    python --version
                    pytest --maxfail=1 --disable-warnings -q
                '''
            }
        }
    }

    post {
        success {
            echo '✅ All tests passed successfully!'
        }
        failure {
            echo '❌ Tests failed.'
        }
    }
}
```

---

## ✅ 驗證安裝成功

```bash
docker ps
```
應看到：

```
CONTAINER ID   IMAGE                       PORTS
xxxxxx         jenkins/jenkins:lts-jdk17   0.0.0.0:8081->8080/tcp
```

開啟 [http://localhost:8081](http://localhost:8081)  
看到 Jenkins 介面代表成功！ 🎉
