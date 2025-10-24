# 🧰 Jenkins Python Pipeline Setup Guide (with Docker)

本文件說明如何在 Docker 上安裝 Jenkins、設定 Python Pipeline，並包含 **安裝 Docker CLI 的兩種方法**。

---

## 🚀 Quick Start

### 🧱 Jenkins + Docker 安裝

#### PowerShell

```powershell
docker run -d `
  -p 8081:8080 `
  -p 50001:50000 `
  -v "D:\docker\jenkins_home:/var/jenkins_home" `
  -v "//var/run/docker.sock:/var/run/docker.sock" `
  --name jenkins_new `
  jenkins/jenkins:lts-jdk17
```

#### CMD

```cmd
mkdir D:\docker\jenkins_home

docker run -d -p 8081:8080 -p 50001:50000 ^
-v D:\docker\jenkins_home:/var/jenkins_home ^
-v //var/run/docker.sock:/var/run/docker.sock ^
--name jenkins_new jenkins/jenkins:lts-jdk17
```

| 參數 | 說明 |
|------|------|
| `-p 8081:8080` | Jenkins 主網頁 Port |
| `-p 50001:50000` | Jenkins Agent 連線 Port |
| `-v D:\docker\jenkins_home:/var/jenkins_home` | Jenkins 資料持久化 |
| `-v //var/run/docker.sock:/var/run/docker.sock` | 掛載宿主 Docker |
| `--name jenkins_new` | 容器名稱 |
| `jenkins/jenkins:lts-jdk17` | Jenkins LTS 映像 |

---

## 🌐 登入 Jenkins

開啟瀏覽器 → [http://localhost:8081](http://localhost:8081)  
初次登入密碼位於：  
`D:\docker\jenkins_home\secrets\initialAdminPassword`

---

## 🔧 安裝插件

在 Jenkins 網頁 → **Manage Jenkins → Manage Plugins → Available**

建議安裝：
- Docker Pipeline
- Docker
- Git
- Pipeline
- (可選) Blue Ocean

---

## 🧩 常用 Docker / Jenkins 指令

| 指令 | 功能 |
|------|------|
| `docker ps -a` | 查看容器運行狀況 |
| `docker rm jenkins_new` | 刪除容器 |
| `docker logs jenkins_new` | 查看 Jenkins 日誌 |
| `docker exec -it jenkins_new bash` | 進入 Jenkins 容器 |
| `docker restart jenkins_new` | 重新啟動 Jenkins |

---

## 🧱 使用自建 Dockerfile (內含 Docker CLI)

若要在 Jenkins 容器內使用 `docker` 指令，有兩種方式：

---

### 方法一：進入容器安裝 docker.io

1. 進入 Jenkins 容器：
   ```bash
   docker exec -u 0 -it jenkins_new bash
   ```

2. 安裝 Docker CLI：
   ```bash
   apt-get update
   apt-get install -y docker.io
   ```

3. 驗證是否安裝成功：
   ```bash
   docker --version
   ```

> ⚠️ 注意：這種方式在 **容器刪除後會遺失設定**，僅適合測試用途。

---

### 方法二：自建 Jenkins + Docker CLI 映像

建立一個 `Dockerfile`：

```dockerfile
FROM jenkins/jenkins:lts-jdk17
USER root
RUN apt-get update && apt-get install -y docker.io
USER jenkins
```

建立映像：

```bash
cd D:\docker\jenkins_custom_3
docker build -t myjenkins-docker-3 .
```

啟動 Jenkins：

#### PowerShell

```powershell
docker run -d `
  -p 8082:8080 `
  -p 50002:50000 `
  -v "D:\docker\jenkins_home_3:/var/jenkins_home" `
  -v "//var/run/docker.sock:/var/run/docker.sock" `
  --name jenkins_custom_3 `
  myjenkins-docker-3
```

#### CMD

```cmd
mkdir D:\docker\jenkins_home_3

docker run -d -p 8082:8080 -p 50002:50000 ^
-v D:\docker\jenkins_home_3:/var/jenkins_home ^
-v //var/run/docker.sock:/var/run/docker.sock ^
--name jenkins_custom_3 myjenkins-docker-3
```

✅ **優點**：Docker CLI 會預先安裝，重啟不會遺失設定。

---

## ⚙️ Jenkinsfile 範例 (Python 測試)

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
                    python main.py
                '''
            }
        }
    }

    post {
        success { echo '✅ All tests passed successfully!' }
        failure { echo '❌ Tests failed.' }
    }
}
```

---

## 🧩 常見錯誤排查

| 問題 | 解決方式 |
|------|----------|
| `docker: not found` | 確認是否掛載 `/var/run/docker.sock` 或使用自建映像 |
| `Permission denied` | 在 Dockerfile 中用 `USER root` 安裝，再切回 `USER jenkins` |
| `apt-get: are you root?` | 容器內需切換為 root 才能安裝軟體 |
| `Couldn't find any revision to build` | 確認 Jenkins job 指向正確分支（main/master） |
| `pytest: not found` | 在 Jenkinsfile 的 `Install dependencies` 階段中安裝 pytest |
| `Invalid agent type "docker" specified` | 確認 Jenkins 有安裝 **Docker Pipeline** 與 **Docker** 插件，安裝後請重啟 Jenkins 再執行。 |

---

## ✅ 最終驗證步驟

1. 開啟 Jenkins → 新建 Pipeline 專案
2. 選擇「Pipeline script from SCM」
3. 指定 Git URL：`https://github.com/cba542/jenkins-pipeline-demo.git`
4. 運行 Pipeline → 應顯示 `✅ All tests passed successfully!`

---

📦 **完成！**  
你的 Jenkins 現在可以自動從 Git 下載專案、安裝 Python 依賴、並執行自動化測試 🚀
