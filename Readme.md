# 紀錄一下Hugo的啟動和日常操作

## 開發環境啟動

### 使用 DevContainer 🚀
使用 VS Code 的 DevContainer 功能，自動設置完整的 Hugo 開發環境：

1. 確保已安裝 VS Code 和 DevContainer 擴展
2. 用 VS Code 開啟此專案
3. 按 `Ctrl+Shift+P` 並執行 "Dev Containers: Reopen in Container"
4. 等待容器建構完成（第一次會比較久）

DevContainer 會自動：
- 安裝最新版本的 Hugo Extended (v0.151.0)
- 設定正確的檔案權限
- 配置開發環境

```bash
# 在 DevContainer 中的常用指令
hugo new posts/my-new-article.md    # 建立新文章
hugo server --bind 0.0.0.0          # 啟動開發服務器
hugo                                 # 建構靜態網站
```

## 常用的hugo指令
```bash
# 啟動開發用blog server
hugo server --bind 0.0.0.0 --buildDrafts
# 新增文章
hugo new posts/gen-import-file.md
# 建構靜態網站
hugo
```


# Q&A
## 移到WSL2後遇到的問題解決
### 1. 發現在容器中，即使執行`hugo server`啟動的網站[http://localhost:1313/] 顯示的內容竟然是空的，
```bash
# 顯示了類似這樣的錯誤訊息
WARN 2022/10/30 03:00:30 found no layout file for "HTML" for kind "page": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2022/10/30 03:00:30 found no layout file for "HTML" for kind "home": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
...
```
找不到layout的原因是部落格有套用Hugo主題，但主題相關的資料夾不存在儲存庫，以下是我安裝Hugo主題的方法(參考[PaperMod教學](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation#method-1))
> 注意! 記得先離開Docker容器並移動到部落格的資料夾內
```bash
# Step1 先將缺少的主題paperMod從github下載到本地資料夾`themes/`裡面
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
# Step2 移動到clone的主題資料夾並完整拉到本地資料夾
cd themes/PaperMod && git pull
```
接著重新進到Docker容器執行指令`hugo server`就會正確的在`/public`目錄下產生對應的靜態頁面
