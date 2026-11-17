# litellm-bedrock

這個 repo 分享一套 LiteLLM proxy 的 Docker Compose 設定：LiteLLM + PostgreSQL + Redis。LiteLLM 不直接連 AWS Bedrock，而是**透過一個上游 LiteLLM 路由器**取得模型；Bedrock 的存取與認證由上游路由器負責，本 repo 不需要也不掛載任何 AWS 認證。

## 前置需求

- **上游 LiteLLM 路由器**：請自行準備（本 repo 不提供）。你需要它的網址與一把可呼叫 `/v1/models` 的 API 金鑰。
- Docker 與 Docker Compose
- `bash`、`curl`、`jq`、`python3`（`gen-upstream-models.sh` 會用到）

## 使用步驟

1. 複製範例設定並填入你自己的值（`.env` 已列在 `.gitignore`，絕對不要 commit）：

   ```bash
   cp .env.example .env
   # 編輯 .env，填入 5 個變數：
   # LITELLM_DB_USER、LITELLM_DB_PASSWORD、LITELLM_MASTER_KEY、
   # UPSTREAM_LITELLM_API_BASE、UPSTREAM_LITELLM_API_KEY
   ```

2. 建立外部 volume（PostgreSQL 資料）：

   ```bash
   docker volume create litellm-bedrock-pg
   ```

3. 由上游路由器產生 `config.yaml` 的模型區塊：

   ```bash
   ./gen-upstream-models.sh
   ```

   腳本會讀取 `.env`（或環境變數），向上游 `/v1/models` 查詢，並依 `model-alias-map.json` 與 `model-base-map.json` 更新 `config.yaml` 中 `BEGIN/END upstream-models` 之間的內容。若任何對應的上游模型不存在，腳本會失敗且不修改 `config.yaml`。成功時會顯示 `Updated config.yaml: 4 models`。

4. 啟動服務：

   ```bash
   docker compose up -d
   ```

   之後若更新了模型區塊，執行 `docker compose up -d --force-recreate litellm` 讓設定生效。

## 服務與連接埠

| 服務 | 容器名稱 | 主機連接埠 |
|---|---|---|
| LiteLLM | litellm | 4000 |
| PostgreSQL | litellm-db | 5433（容器內 5432） |
| Redis | litellm-redis | 6380 |

## 範例模型

`config.yaml` 的模型區塊由 `gen-upstream-models.sh` 產生，內容如下（本機名稱是客戶端與 virtual key 使用的穩定名稱）：

| 本機模型名稱 | 上游模型 id | base_model（計價用） |
|---|---|---|
| `claude_sonnet_5_5` | `claude-sonnet-5-5-global` | `global.anthropic.claude-sonnet-5-5` |
| `claude_haiku_5_5` | `claude-haiku-5-5-global` | `global.anthropic.claude-haiku-5-5` |
| `titan_embed_v2` | `titan-embed-v2` | `amazon.titan-embed-text-v2:0` |
| `qwen3_32b` | `qwen3-32b` | `qwen.qwen3-32b-v1:0` |

這 4 個只是範例；要增減模型，請修改 `model-alias-map.json` 與 `model-base-map.json` 後重新執行腳本。

## 安全注意事項

- **請勿用 `set -x` 或 `curl -v` 執行腳本。** 腳本會把上游 API 金鑰放在 `Authorization` 標頭中；開啟除錯追蹤或 curl 詳細輸出時，金鑰會顯示在終端機（也可能進入終端機紀錄或 CI 日誌），請避免。
- **本 repo 不含任何祕密、`.env`、真實的路由器網址。** 這些都要你自己準備。`.env.example` 裡只有明顯的假值（`change-me`、`https://your-router.example.com`），請勿把真實值寫進任何會被 commit 的檔案。

## 授權

見 [LICENSE](LICENSE)。
