`docker run -d -p 8080:8080 -p 50000:50000 -v D:\docker\jenkins_home_2:/var/jenkins_home -v //var/run/docker.sock:/var/run/docker.sock --name jenkins jenkins/jenkins:lts-jdk17`

| 參數                                              | 功能               |
| ----------------------------------------------- | ---------------- |
| `-p 8080:8080`                                  | Jenkins 主介面 Port |
| `-p 50000:50000`                                | Agent 連線 Port    |
| `-v D:\docker\jenkins_home_2:/var/jenkins_home` | Jenkins 資料持久化    |
| `-v //var/run/docker.sock:/var/run/docker.sock` | 宿主 Docker 掛載     |
| `--name jenkins`                                | 容器名稱             |
| `jenkins/jenkins:lts-jdk17`                     | Jenkins LTS 映像   |


開啟瀏覽器 → http://localhost:8080

找密碼：

\secrets\initialAdminPassword


安裝建議插件

建立 admin 帳號

⚙️ 安裝插件

Manage Jenkins → Manage Plugins → Available

安裝：

Docker Pipeline

Docker

Git

Pipeline

(可選) Blue Ocean


🧰 常用指令
| 指令                             | 用途              |
| ------------------------------ | --------------- |
| `docker ps`                    | 查看 Jenkins 是否運行 |
| `docker logs jenkins`          | 查看 Jenkins log  |
| `docker exec -it jenkins bash` | 進入 Jenkins 容器   |
| `docker restart jenkins`       | 重新啟動 Jenkins    |
| `docker rm jenkins`            | 刪除 Jenkins 容器   |

