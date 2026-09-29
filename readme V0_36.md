# 書法集字去背與數位化工具 v0.38

`v0.38` 是一個專為書法作品、手寫字跡與印章設計的純前端數位化工具。本版本核心重點在於**跨平台下載機制的完美修復**，導入專為 iOS (iPhone/iPad) 設計的原生分享與儲存機制，解決行動裝置下載後找不到檔案的問題，同時保持桌機與 Android 平台的流暢體驗。

---

## 🌟 核心功能與 v0.38 更新重點

- **📱 iOS 跨平台安全下載機制 (v0.38 重點升級)**
  - 導入 `Web Share API` 整合機制，於 iPhone/iPad 裝置下載時，自動呼叫原生分享選單（可直接「儲存影像」至相簿或存入檔案 App）。
  - **100% 向下相容**：Android 與電腦版 (PC/Mac) 完全保留原本的傳統點擊下載流程，不受任何影響。
- **純前端運算 (Zero-Server Architecture)**
  - 所有的影像處理與 WebWorker 運算皆在本地瀏覽器執行，個人資料與圖片絕不上傳伺服器，確保資安與隱私。
- **iOS / Apple HEIC 原生相容**
  - 內建 `heic2any` 轉碼引擎，支援 iPhone/iPad 拍攝之 HEIC 格式照片自動轉碼與導入。
- **高效 WebWorker 圖像去背演算法**
  - 採用獨立 WebWorker 線程進行二值化 (Binarization)、孤立雜訊過濾 (Min Blob Filter) 與筆劃膨脹/收縮 (Dilation/Erosion) 處理，萬級像素運算不卡頓 UI。
- **歷史紀錄 (Undo / Redo) 與狀態還原**
  - 完整紀錄畫布位置、縮放比例、門檻值與顏色設定，支援無限級「上一步」與「下一步」復原。
- **手勢與互動排版**
  - 支援滑鼠拖曳、雙指捏合縮放 (Pinch-to-Zoom)，並提供獨立的字體大小調整與一鍵「置中歸位」按鈕。
- **預覽浮水印保護**
  - 編輯過程中自動繪製「預覽專用 PREVIEW」浮水印，輸出下載時自動剔除，確保正式產出清潔無瑕。
- **多解析度安全匯出引擎**
  - 提供 800px 至 2560px 多種輸出解析度選擇，滿足從網路分享到高解析印刷的多樣需求。

---

## 📐 系統架構與技術細節

本專案採用三層分離架構設計，確保視覺與邏輯分離：

| 層級 | 技術棧 | 負責功能 |
| :--- | :--- | :--- |
| **前端介面 (UI)** | HTML5, CSS3 (Glassmorphism), Vanilla JS | 響應式佈局、控制項監聽、DOM 操作與歷史紀錄佇列管理 |
| **資料傳遞 (Data)** | Base64 / ArrayBuffer / ImageData / Web Share API | 圖像轉碼數據流、WebWorker 雙向訊息傳輸與跨平台分享 Bridge |
| **核心大腦 (Engine)** | HTML5 Canvas API, WebWorker Thread | 影像轉碼、二值化演算法、連通圖雜訊標記、雙線性縮放繪製 |

---

## 🛠️ v0.38 跨平台下載核心邏輯 (safeDownloadCanvas)

```javascript
async function safeDownloadCanvas(sourceCanvas, filename) {
  const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent) && !window.MSStream;

  sourceCanvas.toBlob(async (blob) => {
    if (!blob) return;

    // 1. iOS 裝置：使用 Web Share API 呼叫原生分享選單
    if (isIOS && navigator.canShare) {
      const file = new File([blob], filename, { type: 'image/png' });
      if (navigator.canShare({ files: [file] })) {
        try {
          await navigator.share({
            files: [file],
            title: '下載書法去背圖檔',
            text: '書法集字去背與數位化圖片'
          });
          return;
        } catch (err) {
          if (err.name === 'AbortError') return; // 使用者取消不處理
        }
      }
    }

    // 2. Android 與 Desktop：傳統點擊下載
    const blobUrl = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.download = filename;
    a.href = blobUrl;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    setTimeout(() => URL.revokeObjectURL(blobUrl), 1000);
  }, 'image/png');
}   cd your-repo
