# 🧱 Jenkins + Docker + Python CI/CD Pipeline

本文件紀錄如何在 Windows 上使用 Docker 快速架設 Jenkins，並建立一個能自動測試 Python 專案的 CI Pipeline。

---

## 🚀 Jenkins 安裝與啟動

```bash
docker run -d `
  -p 8081:8080 `
  -p 50001:50000 `
  -v "D:\docker\jenkins_home:/var/jenkins_home" `
  -v "//var/run/docker.sock:/var/run/docker.sock" `
  --name jenkins_new `
  jenkins/jenkins:lts-jdk17
```

| 參數 | 功能 |
| ---- | ---- |
| `-p 8080:8080` | Jenkins 主介面 Port |
| `-p 50000:50000` | Jenkins Agent 連線 Port |
| `-v D:\docker\jenkins_home:/var/jenkins_home` | Jenkins 資料持久化 |
| `-v //var/run/docker.sock:/var/run/docker.sock` | 讓 Jenkins 可操作宿主 Docker |
| `--name jenkins` | 容器名稱 |
| `jenkins/jenkins:lts-jdk17` | Jenkins LTS 映像（含 JDK17） |

---

## 🌐 初始設定步驟

1. 開啟瀏覽器 → [http://localhost:8080](http://localhost:8080)
2. 在主機端找到密碼檔案：
   ```
   D:\docker\jenkins_home_2\secrets\initialAdminPassword
   ```
3. 輸入初始密碼登入  
4. 選擇 **Install suggested plugins**  
5. 建立 `admin` 帳號  
6. 完成後登入 Jenkins Dashboard  

---

## ⚙️ 必裝插件清單

| 插件名稱 | 功能說明 |
| -------- | -------- |
| **Docker Pipeline** | 支援在 Jenkinsfile 使用 `agent docker` |
| **Docker plugin** | 讓 Jenkins 能管理 Docker 容器 |
| **Docker Commons** | 提供 Docker 共用功能 |
| **Git plugin** | 支援 Git SCM |
| **Pipeline plugin** | 支援 pipeline 腳本 |
| **(可選) Blue Ocean** | 提供直覺式 pipeline UI |

> 建議安裝完畢後重啟 Jenkins：
> ```bash
> docker restart jenkins
> ```

---

## 🧰 常用 Docker / Jenkins 指令

| 指令 | 用途 |
| ---- | ---- |
| `docker ps` | 查看 Jenkins 是否正在運行 |
| `docker logs jenkins` | 查看 Jenkins log |
| `docker exec -it jenkins bash` | 進入 Jenkins 容器 |
| `docker restart jenkins` | 重新啟動 Jenkins |
| `docker stop jenkins` | 停止 Jenkins |
| `docker rm jenkins` | 刪除 Jenkins 容器 |

---

## 🧩 Jenkinsfile 範例（Docker + Python）

```groovy
pipeline {
    agent {
        docker {
            image 'python:3.11'
            args '-u root'  // 以 root 執行，避免 pip 權限問題
        }
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/cba542/jenkins-pipeline-demo.git'
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
        always {
            echo 'Pipeline finished.'
        }
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

## 🧠 Pipeline 兩種執行方式

| 模式 | 優點 | 缺點 |
| ---- | ---- | ---- |
| **Pipeline script** | 快速測試、立即修改 | 無法版本控管 |
| **Pipeline script from SCM** | 維護方便、隨 Git 自動同步 | 初期設定較多 |

> ✅ 建議：正式專案請使用 **Pipeline from SCM** 模式  
> Jenkinsfile 放在專案根目錄中。

---

## 🛠 常見錯誤與修正

| 錯誤訊息 | 原因 | 解法 |
| -------- | ---- | ---- |
| `docker: not found` | Jenkins 容器未掛載 Docker | 確認有 `-v //var/run/docker.sock:/var/run/docker.sock` |
| `Invalid agent type "docker"` | 未安裝 Docker Pipeline 外掛 | 安裝後重啟 Jenkins |
| `Permission denied` during `pip install` | 預設 Jenkins user 無權限 | 在 Jenkinsfile 加上 `args '-u root'` |
| `pytest: not found` | 未安裝測試套件 | Jenkinsfile 補上 `pip install pytest` |
| `Python not installed` | 未指定 Python image | 改用 `image 'python:3.11'` |

---

## 🗂️ 文件管理建議

1. 將此 SOP 寫成 `README.md` 放入 GitHub 專案。  
2. 每次修改 Jenkinsfile 時，建立對應的 commit 訊息（方便追蹤變更）。  
3. 可用 [Typora](https://typora.io/) 或 [Obsidian](https://obsidian.md/) 編輯筆記。  

---

## ✅ 範例執行結果

```
[Pipeline] sh
+ python --version
Python 3.11.9
+ pytest --maxfail=1 --disable-warnings -q
.                                                      [100%]
✅ All tests passed successfully!
Finished: SUCCESS
```

---

## 📘 參考連結

- [Official Jenkins Pipeline Docs](https://www.jenkins.io/doc/book/pipeline/)
- [Docker Plugin for Jenkins](https://plugins.jenkins.io/docker-plugin/)
- [Python Docker Image](https://hub.docker.com/_/python)

---

💡 **作者筆記**  
本文件實測於：  
- Windows 11  
- Docker Desktop 27.x  
- Jenkins LTS (JDK17)  
- Python 3.11 Image  
