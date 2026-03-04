# Fix Chrome Crash on HTC U Ultra (LineageOS 18.1)

這個 Magisk 模組旨在修復 HTC U Ultra (oce) 刷入特定的非官方 LineageOS 18.1 系統後，所有基於 Chromium 核心的瀏覽器與系統 WebView 無法啟動（閃退）的問題。

## 📱 目標環境 (Target Environment)

* **裝置型號**：HTC U Ultra (oce)
* **作業系統**：[LineageOS 18.1 Unofficial (Android 11)](https://xdaforums.com/t/rom-11-0-unofficial-oce-lineageos-18-1-stable.4175765/)

## 🐛 異常描述 (Issue)

在上述 LineageOS 18.1 環境中，安裝並開啟 Google Chrome、Microsoft Edge 或依賴 Android System WebView 的應用程式時，App 會在啟動瞬間發生 `FATAL EXCEPTION` 並完全崩潰，無法正常瀏覽網頁。

抓取 Logcat 日誌可發現以下致命錯誤：
`Caused by: java.lang.UnsatisfiedLinkError: dlopen failed: library "libbinder_ndk.so" not found: needed by ... /libwebviewchromium.so in namespace classloader-namespace`

## 🔍 根本原因 (Root Cause)

從 Android 10 開始，Chromium 核心強烈依賴 `libbinder_ndk.so` 來處理底層行程間通訊 (IPC)。
雖然在該 ROM 的 `/system/lib64/` 目錄下確實存在這個函式庫，但 ROM 的編譯者遺漏了將其加入到系統的**公開函式庫白名單** (`/system/etc/public.libraries.txt`) 中。

由於 Android 嚴格的動態連結器 (Dynamic Linker) 命名空間安全機制，當應用程式試圖呼叫不在白名單上的底層庫時，系統會拒絕掛載並直接拋出 `not found` 錯誤，導致進程被強制終止 (Signal 9)。

## 💡 解決方案 (Solution)

本專案製作了一個極簡的 Magisk 模組，利用 Magisk 的 **Systemless (無系統替換)** 掛載機制（Magic Mount），在系統開機時，自動將補上 `libbinder_ndk.so` 的 `public.libraries.txt` 覆蓋至系統層。

優勢：
* 不需要修改或解包唯讀的系統分區 (System Partition)。
* 完美解決權限阻擋問題，讓所有 Chromium 相關應用滿血復活。
* 隨時可透過 Magisk 停用或解除安裝，安全無副作用。

## ⚙️ 安裝方式 (Installation)

1. 進入本倉庫的 [Releases] 頁面，下載最新的 `.zip` 模組壓縮檔。（或下載本專案源碼自行將內容打包成 ZIP）
2. 開啟手機的 **Magisk Manager** 應用程式。
3. 切換到「模組 (Modules)」分頁。
4. 點擊「從儲存空間安裝 (Install from storage)」。
5. 選擇剛剛下載的 ZIP 檔案進行刷入。
6. 完成後，點擊「重新啟動 (Reboot)」。
7. 開機後，Chrome 與 WebView 即可正常運作！

## 👨‍💻 作者 (Author)

* **lokey0905**
