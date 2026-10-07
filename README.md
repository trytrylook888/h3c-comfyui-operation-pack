# H3.c + ComfyUI Mac 安裝器

在 Finder 雙擊 `H3-ComfyUI-Installer.command`。它會讓你選安裝位置，再選擇只裝 FL2VA，或一起裝 Ref2VA。接著可選擇由安裝器下載 FL2VA，或選取自己已下載的 `MiniMax-H3` 資料夾（下一層必須有 `FL2VA`）。選取既有模型時，安裝器只建立唯讀連結，不會改動權重。安裝器把 h3.c、獨立的 ComfyUI、h3.c bridge 和工作流放在同一個新資料夾；它拒絕使用非空資料夾，因此不會覆蓋既有 ComfyUI 或模型。

只想檢查選項時，可在終端機執行 `bash H3-ComfyUI-Installer.command --preview-only`。它會顯示相同的選擇畫面，最後列出計畫；不會下載、安裝或建立檔案。一般 Finder 雙擊仍會進入正式安裝流程。

模型檔不包在操作包內。安裝過程會從 GitHub 和 Hugging Face 的固定版本下載 h3.c、ComfyUI 和官方 FL2VA 權重；Ref2VA 是額外的選項。它需要至少 230GiB 可用空間，原因是 FL2VA 與選配的 Ref2VA 權重都很大，還要保留下載與執行空間。安裝完成後也不要任意刪除其中任一資料夾。

開始前必須已有 Apple 的 Command Line Tools（提供 `git` 與 `make`）、Python 3.10 以上，以及 `ffmpeg`/`ffprobe`。如果缺少任一工具，安裝器會在下載模型前停止並列出缺少項目，不會改動既有資料夾。安裝完成後雙擊安裝資料夾裡的 `start_h3_comfy.command`；它會避開已被使用的埠號，並印出應開啟的本機網址。

ComfyUI 內附一個 `日本女孩 FL2VA Demo` 工作流範例，已附九宮格圖片，節點順序是載入圖片、H3 Native Video 節點、儲存影片；第一次試跑前可先把提示詞及輸出前綴改成自己的內容。安裝器也會建立橋接節點所需的暫存目錄。

初次安裝會下載大量檔案，請保持網路和電腦不中斷。中斷後，再雙擊一次 `H3-ComfyUI-Installer.command`，並選擇和上次相同的安裝位置與選項：安裝器會偵測到上次未完成的安裝，自動從中斷處繼續（等同於在 Terminal 對核心腳本加上 `--resume`）。若該位置已經安裝完成，它會直接提示你雙擊 `start_h3_comfy.command`，不會重裝。續裝只接受帶有安裝器狀態檔的資料夾；其他非空資料夾仍會拒絕寫入。安裝器會重用已完成的 clone、編譯與 Python 環境，最後從中斷處繼續。狀態檔也會記下第一次選的模式（FL2VA 或 FL2VA + Ref2VA）與模型來源（下載或既有資料夾的路徑）；續裝時如果選了不同的選項，安裝器會在動手前停下，告訴你上次選了什麼，請重新執行並選擇相同的選項。想改用新的選項時，請改選另一個安裝位置。

## 第一次打開被 macOS 擋下

從網路下載的檔案都會被 macOS 加上「來自網路」的標記。雙擊這類檔案時，macOS 會檢查它有沒有經過 Apple 的付費開發者簽署與公證；這個操作包沒有，所以雙擊 `H3-ComfyUI-Installer.command` 會看到「未打開」與「Apple 無法驗證……是否為惡意軟體」，只有「完成」和「丟到垃圾桶」兩個按鈕。這個訊息代表 Apple 沒有審查過這個檔案，不代表檔案有問題；安裝器和 `install_h3c_comfy.sh` 都是純文字腳本，可以用「文字編輯」打開，逐行看它會做什麼。

**建議做法（不需要更改任何系統設定）：用「終端機」啟動**

1. 打開「終端機」（應用程式 → 工具程式 → 終端機）。
2. 輸入 `bash`，後面加一個空白。
3. 把 `H3-ComfyUI-Installer.command` 從 Finder 拖進終端機視窗，路徑會自動填好，按 Return。

接下來的選擇畫面和雙擊時完全一樣。macOS 的這個檢查只在從 Finder 雙擊時進行，由終端機直接執行不會出現上面的訊息。

**另一個方法：在系統設定中允許**

在較新的 macOS 上，按右鍵選「打開」已經不能繞過這個提示，請改用下面的方法：

1. 先雙擊一次，讓 macOS 顯示阻擋訊息，按「完成」（不要按「丟到垃圾桶」）。
2. 打開「系統設定」→「隱私權與安全性」，往下捲到最下方的安全性區塊。
3. 那裡會提到剛才被擋下的檔案，按旁邊的「強制打開」（有些版本顯示為「仍要打開」），再輸入密碼或用 Touch ID 確認。
4. 再雙擊一次，在跳出的確認視窗選擇打開。按鈕文字會因 macOS 版本而略有不同。

第三個方法是在「終端機」執行下面這行（把引號內換成你解壓縮後的資料夾路徑，可以直接把資料夾拖進終端機視窗）：

```
xattr -dr com.apple.quarantine "<解壓縮後的資料夾>"
```

這行會移除 macOS 對「從網路下載的檔案」加上的隔離標記（`-d` 是刪除這個標記，`-r` 是套用到資料夾內所有檔案）。只對你信任來源的檔案這樣做。

## 前置安裝

安裝器開始下載前會一次檢查所有前置條件，並用中文列出每個缺少的項目與修正方法，不會建立任何檔案。請先準備：

- Apple Command Line Tools（提供 `git` 與 `make`）：在「終端機」執行 `xcode-select --install`，跳出視窗後按「安裝」。
- Python 3.10 以上：到 https://www.python.org/downloads/macos/ 下載 macOS 安裝檔，建議 3.12。若 Python 是用 Homebrew 裝的、終端機卻找不到，安裝器同樣會到 `/opt/homebrew/bin` 與 `/usr/local/bin` 找。
- `ffmpeg` 與 `ffprobe`：先到 https://brew.sh 依指示安裝 Homebrew，再執行 `brew install ffmpeg`。如果 Homebrew 已裝好但終端機找不到它，安裝器也會到 `/opt/homebrew/bin` 與 `/usr/local/bin` 找，找到後會讓啟動器使用同一個路徑。

想先檢查、不要安裝時，在「終端機」進入操作包資料夾後執行：

```
bash install_h3c_comfy.sh --install-dir "<你要裝的位置>/MiniMax-H3-h3c-ComfyUI" --preflight-only
```

`--install-dir` 要填你實際要安裝的位置（Finder 精靈會在你選的資料夾下建立 `MiniMax-H3-h3c-ComfyUI`），因為可用空間與磁碟格式是對那個位置檢查的。全部通過時結束碼為 0，有缺少項目時為 1；它不會建立安裝資料夾，也不會下載任何東西。

## 硬體

- 必須是 Apple Silicon（M 系列晶片）的 Mac，Intel Mac 不支援。
- 目前只在 M5 Max、64GB 記憶體的 Mac 上實測過；其他機型沒有實測。
- 記憶體低於 48GB 時，安裝器會自動讓生成使用 SSD streaming（從 SSD 串流模型權重），速度較慢但較省記憶體。若在節點裡開啟 `use_int8_row_fc2`，則不會強制 SSD streaming，因為兩者不能同時使用。
- 記憶體低於 24GB 的機型未經驗證，安裝可以完成，但生成可能失敗或非常慢。
- 裝在外接硬碟時，硬碟必須是 APFS（Mac 格式）；exFAT 或 FAT 格式不支援，因為安裝需要建立符號連結。
