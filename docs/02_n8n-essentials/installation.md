# 2.1 n8n 安裝

本節將引導您完成 n8n 的安裝過程，讓您能夠在本機環境或伺服器上運行 n8n。

## 2.1.1 安裝方式概述

n8n 提供了多種靈活的安裝方式，以適應不同的使用需求和部署環境。選擇合適的安裝方法是確保 n8n 穩定運行和易於管理的關鍵。本節將概述主要的安裝選項及其適用情況。

主要的 n8n 安裝方式包括：

1.  **使用 Docker 進行安裝：** 這是官方推薦也是最常見的安裝方式，特別適合生產環境。Docker 提供了一個隔離且一致的運行環境，簡化了依賴管理和部署過程。透過 Docker Compose，您可以輕鬆地將 n8n 與資料庫（如 PostgreSQL）和其他必要服務一起部署。
2.  **使用 npm 進行安裝：** 對於本機開發測試或簡單試用，使用 npm（Node.js 的套件管理器）直接安裝 n8n 是一個快速簡便的選擇。但這種方式在生產環境中可能需要額外的配置來確保穩定性和持久性。
3.  **使用雲端服務或一鍵部署：** n8n 支援部署到各種雲端平台，如 Heroku、Render、DigitalOcean 等，或使用官方提供的一鍵部署腳本。這些方式可以快速啟動 n8n 實例，並通常包含自動化的設定步驟。

**如何選擇適合的安裝方式？**

選擇哪種安裝方式通常取決於以下幾個因素：

*   **使用情境：** 是用於生產環境處理關鍵任務，還是僅用於本地開發測試？生產環境更推薦使用 Docker 或穩定的雲端部署方式。
*   **技術熟悉度：** 您對 Docker、npm 或特定雲端平台有多熟悉？選擇您最熟悉的方式可以降低設定難度。
*   **資源限制：** 可用的伺服器資源或雲端預算。Docker 通常需要較多資源，而 npm 安裝可能對本機資源要求較低。
*   **擴展性與可靠性：** 對於需要高可用性和未來可能擴展的應用，基於 Docker 或專業雲端服務的部署會更為適合。

對於大多數生產環境的部署，**使用 Docker 是最推薦的方式**，它提供了最佳的隔離性、可移植性和管理便利性。在後面的小節中，我們將詳細介紹幾種主要的安裝步驟。

## 2.1.2 使用 Docker 安裝 (建議方式)

使用 Docker 是安裝和運行 n8n 的推薦方式，特別是對於生產環境部署。Docker 確保了 n8n 及其依賴（如資料庫）運行在一個隔離、一致的環境中，大大簡化了部署和管理。本節將詳細說明如何使用 Docker Compose 來安裝 n8n。

**先決條件：**

在開始之前，請確保您的系統已經安裝了 Docker 和 Docker Compose。您可以參考 Docker 官方文檔進行安裝：

*   [Docker Engine 安裝指南](https://docs.docker.com/engine/install/)
*   [Docker Compose 安裝指南](https://docs.docker.com/compose/install/)

**安裝步驟：**

1.  **建立專案資料夾：**
    首先，為您的 n8n 部署建立一個專門的資料夾。這個資料夾將用於存放 `docker-compose.yml` 檔案和 n8n 的持久化資料。

    ```bash
mkdir n8n-medical
cd n8n-medical
    ```

2.  **建立 Docker Compose 檔案：**
    在剛建立的 `n8n-medical` 資料夾中，建立一個名為 `docker-compose.yml` 的檔案。這個檔案定義了運行 n8n 和資料庫服務所需的配置。以下是一個包含 n8n 和 PostgreSQL 資料庫的 `docker-compose.yml` 範例：

    ```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - WEBHOOK_URL=${WEBHOOK_URL}
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - ~/.n8n:/home/node/.n8n
    depends_on:
      - postgres

  postgres:
    image: postgres:14
    restart: always
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
    driver: local
    ```

    **注意：** 上述範例使用了多個環境變數（以 `${}` 包裹），這些變數需要在運行 Docker Compose 之前設定。您可以建立一個 `.env` 檔案來定義這些變數，例如：

    ```bash
N8N_BASIC_AUTH_USER=your_n8n_user
N8N_BASIC_AUTH_PASSWORD=your_n8n_password
N8N_HOST=localhost
WEBHOOK_URL=http://localhost:5678/
POSTGRES_USER=your_postgres_user
POSTGRES_PASSWORD=your_postgres_password
POSTGRES_DB=n8n_db
    ```
    請務必替換 `your_n8n_user`, `your_n8n_password`, `your_postgres_user`, `your_postgres_password`, `n8n_db` 以及 `N8N_HOST` 和 `WEBHOOK_URL` 的值。在生產環境中，建議使用更安全的密碼並配置 HTTPS。

3.  **啟動 n8n 服務：**
    在 `n8n-medical` 資料夾中，運行以下指令啟動 n8n 和 PostgreSQL 服務：

    ```bash
docker-compose up -d
    ```
    `-d` 標誌表示在背景運行服務。

**安裝後的驗證：**

服務啟動後，您應該能夠透過瀏覽器訪問 `http://localhost:5678` （如果 `N8N_HOST` 設定為 `localhost`）。您將看到 n8n 的登入介面，使用您在 `.env` 檔案中設定的用戶名和密碼進行登入。

如果無法訪問，您可以使用以下指令查看服務狀態和日誌進行排錯：

```bash
docker-compose ps
docker-compose logs n8n
```

至此，您已經成功使用 Docker Compose 安裝並運行了 n8n。接下來的小節將介紹其他的安裝方式。

## 2.1.3 使用 npm 安裝

如果您只需要在本機環境快速測試 n8n，或者用於開發目的，使用 npm 安裝是一個簡單快速的選項。請注意，這種方式通常不建議用於生產環境，因為它可能不如 Docker 安裝方式穩定和易於管理持久化資料。

**先決條件：**

使用 npm 安裝 n8n 需要您的系統安裝了 Node.js 和 npm。請確保您的 Node.js 版本符合 n8n 的最低要求（通常建議使用 LTS 版本）。您可以從 [Node.js 官方網站](https://nodejs.org/) 下載並安裝。

**安裝步驟：**

1.  **安裝 n8n：**
    打開您的終端或命令提示字元，運行以下 npm 命令來全局安裝 n8n：

    ```bash
npm install n8n -g
    ```
    `-g` 標誌表示全局安裝，這樣您就可以在任何目錄下運行 `n8n` 命令。

2.  **啟動 n8n：**
    安裝完成後，直接在終端中運行以下命令來啟動 n8n 服務：

    ```bash
n8n
    ```

**注意事項：**

*   **資料持久化：** 預設情況下，npm 安裝的 n8n 會將資料（如工作流程）儲存在用戶主目錄下的 `.n8n` 資料夾中。如果需要指定不同的資料夾，可以使用 `N8N_DATA_FOLDER` 環境變數。
*   **背景運行：** 如果您需要讓 n8n 在背景運行，您可能需要使用如 `pm2` 或 `forever` 之類的流程管理器。
*   **更新 n8n：** 要更新通過 npm 安裝的 n8n，只需再次運行 `npm update -g n8n` 命令。

使用 npm 安裝適合快速體驗 n8n 的功能，但對於需要穩定性和可靠性的正式應用，強烈建議考慮 Docker 或其他適合生產環境的部署方式。

## 2.1.4 其他安裝方式 (選填)

除了 Docker 和 npm，n8n 還支援部署到各種雲端平台或使用特定的一鍵部署方案。這些方法可以根據您的具體需求和偏好的雲端服務提供商來選擇。

一些其他的安裝方式包括：

*   **雲端服務提供商的應用市場或容器服務：** 許多雲端平台（如 AWS ECS/EKS, Google Cloud GKE, Azure AKS, DigitalOcean App Platform, Render 等）提供容器託管服務，您可以直接在這些平台上部署 n8n 的 Docker 映像。有些平台還可能在他們的應用市場中提供 n8n 的一鍵部署選項。
*   **基於 Helm Charts (Kubernetes)：** 如果您的組織使用 Kubernetes 進行容器編排，可以使用 n8n 提供的 Helm Chart 進行部署。這種方式適合在大型、複雜的基礎設施中管理 n8n。
*   **一鍵部署腳本或模板：** n8n 社群或第三方可能會提供針對特定環境或雲端提供商的一鍵部署腳本或 Terraform 模板，這些可以幫助您快速自動化部署過程。

選擇這些安裝方式通常需要對特定的雲端平台或技術有一定的了解。具體的部署步驟會因平台而異，建議參考 n8n 官方文檔或相關雲端服務提供商的指南。

## 2.1.5 安裝後的驗證

成功完成 n8n 的安裝後，最後一步是驗證服務是否已正確啟動並可供使用。驗證方法取決於您選擇的安裝方式。

**1. 透過 Web 介面驗證：**

無論您是使用 Docker 還是 npm 安裝，n8n 預設都會啟動一個 Web 服務，通常在連接埠 `5678` 上。開啟您的網頁瀏覽器，並訪問以下網址：

```
http://localhost:5678
```

如果您在安裝時設定了不同的主機名或連接埠，請使用對應的網址。如果一切正常，您應該會看到 n8n 的登入頁面或歡迎介面。如果使用了基礎認證，請輸入您設定的用戶名和密碼。

**2. 透過終端機指令驗證 (僅適用於 Docker)：**

如果您使用 Docker Compose 安裝，可以使用以下指令來檢查 n8n 容器的運行狀態：

```bash
docker-compose ps
```

如果 n8n 容器正在運行，您應該會在輸出中看到類似 `Up` 的狀態標示。您也可以查看 n8n 容器的日誌來確認服務是否正常啟動，例如：

```bash
docker-compose logs n8n
```

如果日誌中沒有顯示錯誤訊息，且容器狀態為 `Up`，則表示 n8n 服務已成功啟動。

**3. 透過終端機指令驗證 (僅適用於 npm 全局安裝)：**

如果您使用 npm 全局安裝，並且在終端機中直接運行 `n8n` 命令啟動服務，只要命令沒有報錯並顯示服務正在監聽特定連接埠的信息，就表示啟動成功。您也可以在新開的終端中嘗試運行 `n8n --version` 來確認是否能正確執行 n8n 命令。

如果驗證失敗，請回到您使用的安裝方法小節，仔細檢查安裝步驟和任何錯誤訊息，並參考 n8n 官方文檔進行排錯。

## 學習重點

完成本節後，您應該能夠：

- 了解安裝 n8n 的不同方式。
- 選擇適合您的安裝方法並成功完成安裝。
- 驗證 n8n 服務是否正常運行。 