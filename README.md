# CI/CD Lab

這份文件是 Lab 手冊，會帶你完成：

1. 啟動 Fastify 應用
2. 實際 push 到 GitHub 觀察 CI
3. 在本機用 `act` 模擬 CI
4. 理解不同 branch 推送時的效果
5. 完成課堂練習

## 開始前

建議使用 GitHub Codespaces 開啟本 repo（環境已預先準備 Node / Docker / act）

### Fork

1. 到原始 repo 頁面，點右上角 **Fork**
2. 進入你自己的 fork repo
3. 用你的 fork repo 開啟 Codespaces

先確認工具版本：

```bash
node --version
docker --version
docker compose version
act --version
```

## 本地開發

安裝依賴：

```bash
npm ci
```

**說明：** 這裡使用 `npm ci` (alias: clean-install) 而非 `npm install`，以確保開發環境的一致性：
- **嚴格鎖定版本**：完全依照 `package-lock.json` 安裝精確版本，避免套件意外升級。
- **環境最乾淨**：會自動清除舊的 `node_modules` 並重新安裝，避免殘留檔案造成不可預期的錯誤。

<br>

啟動服務：

```bash
npm run build
npm run start
```

驗證服務：

```bash
curl http://localhost:3000/
curl http://localhost:3000/health
```

## Lab-01: Hello GitHub Actions

Push 到 GitHub 觀察 CI

### 第一次使用請先確認 Actions 已啟用

在你的 fork repo：

1. 點選上方 **Actions** 分頁
2. 如果看到啟用提示，點 **I understand my workflows, go ahead and enable them**（或同意按鈕）
3. 回到 Code 頁面繼續操作

### 實際推送一個 feature branch

把 `snippets/01_hello.yaml` 複製到 `.github/workflows/` 底下

```bash
# 1. 建立新的 branch 名為 feature/ci-observe
git checkout -b feature/ci-observe

# 2. 複製新的 workflow
cp snippets/01_hello.yaml .github/workflows/

# 3. 將所有的變更加入暫存區
git add .

# 4. 提交並註明這次的變更
git commit -m "ci: add hello.yaml"

# 5. 推送到遠端
git push -u origin feature/ci-observe
```

### 在 GitHub Actions 頁面觀察 Hello CI 結果

1. 到你的 fork repo 的 **Actions** 頁面
2. 找到最新 `ci` workflow run
3. 查看 01_hello.yaml 的執行步驟與結果

---

## Lab-02: Run test

把 `snippets/02_run-test.yaml` 複製到 `.github/workflows/` 底下

```bash
# 1. 告訴 Git 刪除上一個 Lab 的 workflow
git rm .github/workflows/01_hello.yaml

# 2. 複製新的 workflow
cp snippets/02_run-test.yaml .github/workflows/

# 3. 將所有的變更（包含刪除與新增）加入暫存區
git add .

# 4. 提交並註明這次的變更
git commit -m "ci: add run-test.yaml"

# 5. 推送到遠端
git push -u origin feature/ci-observe
```

### 在 GitHub Actions 頁面觀察 Run test CI 結果

- 觀察 02_run-test.yaml 內容
- 到 GitHub Actions 頁面, 查看 run test 執行結果
- 觀察 **artifact**：請點進該次執行的 Summary 頁面，滑到最底部，你會看到打包好的檔案（例如測試報告）可以點擊下載。

---

## Lab-03: Run GitHub Actions with act locally

### 用 `act` 模擬 push event

`act` 是一個可以在本機執行 GitHub Actions workflow 的工具

你可以用它：

- 在不 push 到 GitHub 的情況下先驗證 workflow
- 快速重跑失敗步驟、縮短除錯時間
- 模擬 `push` / `pull_request` 等事件

官方資源：

- 官網：<https://nektosact.com/>
- GitHub 專案：<https://github.com/nektos/act>

切到要模擬的 branch，執行 `act push`

```bash
act push
```

若只想跑單一 workflow，可加 `-W`：

```bash
act push -W .github/workflows/ci.yaml
```

進階補充：若你需要在「不切 branch」情況下指定事件分支，可以使用：

```bash
act push --env GITHUB_REF=refs/heads/<branch>
```

---

## Lab-04: Conditional workflow and deploying

實際應用中，我們可能會將每一個版本都跑過 CI，並 build 出 image。
但 image tag 要可以區分出是 feature branch 或是 release branch，方便我們理解與追蹤特定版本的 source code。

### 準備工作與觀察 feature branch

1. 建立並切換到新的 feature branch
2. 清理上一個 Lab 的 workflow，避免干擾
3. 放入具備條件判斷的 `ci.yaml` 與 `cd.yaml`

```bash
# 1. 切換到新的 feature branch
git checkout -b feature/a

# 2. 移除 Lab-02 的 workflow
git rm .github/workflows/02_run-test.yaml

# 3. 將新的 workflow 複製進來
cp snippets/ci.yaml .github/workflows/
cp snippets/cd.yaml .github/workflows/

# 4. 提交變更
git add .
git commit -m "ci: add conditional ci and cd workflows"
```

接著，用 act 模擬推送，觀察執行結果：

```bash
act push
```

觀察 build 出來的 image

```bash
docker images
```
**觀察重點**： 請注意看 TAG 這一欄！確認它是不是被打上了 feature-a（或者是對應的分支名稱），而不是預設的 latest。

### 切出 release branch 觀察 ci.yaml 執行結果

切出 release branch

```bash
git checkout -b release/1.0.0
act push
```

再次觀察 build 出來的 image

```bash
docker images
```
**觀察重點**： 確認這次的 TAG 成功變成了 1.0.0！

## 思考

- 如何設計 CI Pipeline 以確保程式碼品質
- 如何設計 CD Pipeline 部署到目標環境
- CI Pipeline 與 CD Pipeline 的相依關係
- 通知或報表機制
