# 🛠️ Contribution Guide

歡迎貢獻 HRC Tech Docs！  
以下會說明協作流程與注意事項，請在開始修改前先閱讀了解。

---

## 專案架構簡介

本筆記專案架設於 GitHub 上，並透過 GitHub Pages 自動部署至網頁。主要架構說明如下：

- `docs/`：放置所有 Markdown 筆記文件。
- `.github/workflows/gh-pages.yml`：
    CI/CD 自動化流程。  
    已定義在每次 `git push` 到 `main` branch 後，會自動建置 mkdocs 並部署到 [GitHub Pages](https://hrc-pme.github.io/hrc-tech-docs/)。
- `mkdocs.yml`：mkdocs 網站的設定檔。包含主題、導航列、語言、功能等設定。

---

## 更新步驟

1. 下載專案：
    ```
    git clone https://github.com/hrc-pme/hrc-tech-docs.git
    ```
2. 瀏覽到專案目錄，在 Linux OS 或 WSL 上做本地部署：
    ```
    ./run.sh
    ```

3. 開啟瀏覽器 `http://localhost:8000/`，能看到在本地中渲染好的網頁。
    網頁會隨文件內容動態更新。

4. 由於 Github Pages 只吃 `main` branch 上的版本。因此，
    * 僅做測試/請求校正：  
        1. 在本地新建 branch (e.g `dev-pomelo925`)。並且在本地部署及更新測試。
        2. 更新完成後 `git push` 到 Github 上。  
            由於 Github 只吃 `main` branch 的內容，推送新 branch 是不會影響到 Github Pages 的。  
        3. 丟 PR 給 `main` branch，並請他人審核後再合併到 `main` branch 上。 
    
    * 直接部署：
        1. 本地測試時就在 `main` branch 上，並且更新完成後做 `git push`。
        2. 在 Github 上等待 Workflows 完成，即可刷新 Github Pages 檢查結果。