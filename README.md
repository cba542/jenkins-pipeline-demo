
# 🧱 Jenkins Python Pipeline + Docker 整合指南 (v7)

本文件涵蓋 Jenkins Pipeline、Docker 部署、完整備份與還原策略，並附常見錯誤排查。

---

## 🚀 環境建置

### PowerShell

```powershell
docker run -d `
  -p 8081:8080 `
  -p 50001:50000 `
  -v "D:\docker\jenkins_home_original:/var/jenkins_home" `
  -v "//var/run/docker.sock:/var/run/docker.sock" `
  --name jenkins_new `
  jenkins/jenkins:lts-jdk17
```

### CMD

```cmd
docker run -d -p 8081:8080 -p 50001:50000 ^
-v D:\docker\jenkins_home_original:/var/jenkins_home ^
-v //var/run/docker.sock:/var/run/docker.sock ^
--name jenkins_new jenkins/jenkins:lts-jdk17
```

| 參數 | 功能 |
|------|------|
| `-p 8080:8080` | Jenkins 主介面 Port |
| `-p 50000:50000` | Agent 連線 Port |
| `-v D:\docker\jenkins_home_original:/var/jenkins_home` | Jenkins 資料持久化 |
| `-v //var/run/docker.sock:/var/run/docker.sock` | 宿主 Docker 掛載 |
| `--name jenkins` | 容器名稱 |
| `jenkins/jenkins:lts-jdk17` | Jenkins LTS 映像 |

---

## 🧱 使用自建 Dockerfile (內含 Docker CLI)

建立 `Dockerfile`：

```dockerfile
FROM jenkins/jenkins:lts-jdk17
USER root
RUN apt-get update && apt-get install -y docker.io
USER jenkins
```

### 建立映像

```bash
docker build -t myjenkins-docker:v1 .
```

### 啟動 Jenkins 容器

```bash
docker run -d -p 8082:8080 -p 50002:50000 ^
  -v D:\docker\jenkins_home_new:/var/jenkins_home ^
  -v //var/run/docker.sock:/var/run/docker.sock ^
  --name jenkins_docker_v1 myjenkins-docker:v1
```

---

## 💾 Jenkins 完整備份與還原指南

### 匯出 Jenkins home 資料夾

```bash
cd D:\docker
tar -cvf jenkins_home_original_backup.tar jenkins_home_original
```

### 匯出 Jenkins 映像

```bash
docker commit jenkins myjenkins:backup
docker save -o D:\docker\myjenkins_backup.tar myjenkins:backup
```

### 匯入備份映像與資料夾

```bash
docker load -i D:\docker\myjenkins_backup.tar
tar -xvf D:\docker\jenkins_home_original_backup.tar -C D:\docker\
```

### 啟動 Jenkins (使用還原資料)

```bash
docker run -d -p 8080:8080 -p 50000:50000 ^
  -v D:\docker\jenkins_home_original:/var/jenkins_home ^
  --name jenkins_restored myjenkins:backup
```

---

## 🔍 查看 Jenkins 掛載目錄

```bash
docker inspect jenkins --format "{{ .Mounts }}"
```

輸出範例：
```
[{bind  D:\docker\jenkins_home_original  /var/jenkins_home  rw  true  rprivate}]
```

代表 Jenkins 的實際設定資料存在：
> D:\docker\jenkins_home_original

---

## 🧹 Docker 清理與刪除映像

### 查看懸空映像

```bash
docker images -f "dangling=true"
```

### 刪除單一 `<none>` 映像

```bash
docker rmi <image-id>
```

### 強制刪除所有未使用映像

```bash
docker image prune -a
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

## 🧰 常用 Docker / Jenkins 指令

| 指令 | 用途 |
|------|------|
| `docker ps -a` | 查看所有容器 |
| `docker logs jenkins` | 檢視 Jenkins 日誌 |
| `docker exec -it jenkins bash` | 進入 Jenkins 容器 |
| `docker restart jenkins` | 重新啟動 Jenkins |
| `docker rm jenkins` | 刪除 Jenkins 容器 |
