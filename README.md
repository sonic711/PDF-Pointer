# 使用說明

此專案提供離線 PDF 座標預覽工具，讓使用者上傳 PDF 後，在預覽畫面中以滑鼠左鍵點擊任一位置即可立即得知該點在 PDF 座標系中的 X、Y 數值。

## 主要功能

- 輕量 UI：上傳即預覽，支援明暗模式。
- 十字準線：滑鼠移入 PDF 預覽區即可顯示，便於精準對位。
- 座標查詢：左鍵點擊任一頁面位置即顯示 PDF 座標並彈出提醒。
- 雙點量測：右鍵兩次量測兩點間 ΔX、ΔY，並以暫時紅線標示，按 `Esc` 即可重置。
- Alt 軸鎖：按住 `Alt/Option` 鍵移動滑鼠時，十字準線與座標僅沿水平或垂直方向移動，便於沿著單一軸線對齊。

## 目錄結構

```
./index.html             # 主頁面，含樣式與互動邏輯
./vendor/pdfjs/pdf.min.js # 第三方套件，讓本地可以離線運行
./vendor/pdfjs/pdf.worker.min.js # 第三方套件，讓本地可以離線運行
```

## 執行需求

- 支援本地檔案讀取的現代瀏覽器 (Chrome、Edge、Firefox、Safari 等)。
- 若瀏覽器預設封鎖本地 Web Worker，請於開啟檔案前允許「讀取本機檔案」權限或改用支援 `file://` 的瀏覽器，例如 Safari、Firefox 或啟動 Chrome 的 `--allow-file-access-from-files` 模式。

## 啟動步驟

1. 在 Finder / 檔案總管中找到 `index.html`。
2. 直接雙擊或使用瀏覽器的「開啟檔案」功能載入 `index.html`。
3. 若瀏覽器提示需要允許本地檔案權限，請依指示授權後重新載入即可。

## 使用方式

1. 點擊「選擇 PDF 檔案」上傳本地 PDF。
2. 頁面載入完成後，畫面會顯示每一頁的預覽。
3. 當滑鼠移入預覽區時，會出現十字準線協助對準。
4. 在任一頁面上以滑鼠左鍵點擊想取得座標的位置：
   - 上方座標框會顯示 `頁碼 N | X: xxx, Y: yyy`。
   - 瀏覽器會彈出 alert 顯示相同資訊，可方便記錄。
5. 若要量測兩點間的位移，請使用滑鼠右鍵：
   - 第一次右鍵：記錄起點並在座標框顯示 `起點...`。
   - 第二次右鍵：在終點畫出臨時紅線並顯示 `ΔX/ΔY` 差值。
   - 按下 `Esc` 可取消紅線並重新開始量測。

## 座標計算原理

1. 透過 `pointerdown` 事件取得滑鼠位置 `event.clientX/Y`，再配合該頁 canvas 的 `getBoundingClientRect()`，換算為相對於 canvas 的 CSS 座標。
2. 因 PDF 會依容器寬度縮放，canvas 實際像素 (`canvas.width/height`) 與 CSS 尺寸不同，因此需用縮放比 `canvas.width / rect.width`、`canvas.height / rect.height` 還原成渲染時的像素座標。
3. pdf.js 在渲染時會回傳 `viewport`，包含從 canvas 像素轉換至 PDF 內部座標系的函式 `viewport.convertToPdfPoint`。將上述像素座標帶入後，即可得到 PDF 原生的 X/Y（以左下角為 (0,0)）。
4. 右鍵量測亦沿用同一轉換流程，分別記錄起點與終點的 PDF 座標，再相減計算 `ΔX/ΔY`，並以兩點在 viewer 容器內的座標繪製暫時紅線。
