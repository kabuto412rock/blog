# 紀錄一下Hugo的啟動和日常操作

## 如何啟動
> 記得在分支 `main`分支啟動容器做調整
> public目錄屬於輸出 靜態html的部分，由github action在我git push之後自動執行

```bash
# 啟動Hugo容器並將當前資料夾與容器內的/src目錄進行同步
docker compose up -d
# 進入容器`hugo-blog`的終端機
docker exec -it hugo-blog bash
```

## 常用的hugo指令
```bash
# 
# 新增文章
hugo  new 
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
