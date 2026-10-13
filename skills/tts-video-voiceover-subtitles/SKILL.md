---
name: tts-video-voiceover-subtitles
description: Edge-TTS 連線設定、語音配音、SRT 字幕產生與影片/PPT 影音字幕合成工具與工作流程。支援批量配音、PPT 轉影片配音、MP4 加旁白、SRT 產生與 ffmpeg 硬字幕燒錄。
argument-hint: "[task-type: ppt-to-video|mp4-voiceover|add-subtitles] [script-or-input-path]"
metadata:
  author: Antigravity
  version: "1.0.0"
---

# TTS 連線設定與影片配音字幕技能 (TTS Video Voiceover & Subtitles)

通用語音合成 (Edge-TTS)、SRT 字幕製作、PPT 簡報轉影片配音，以及使用 FFmpeg 進行影音對齊與字幕燒錄之完整解決方案。

> **雲端硬碟現成工具參考位置**：`04教科書/0自編教材/翰林/PPT動畫/影片製作工具/`
> （內有 `probe_new.py`、`tts_t.py` 等現成腳本）

---

## 核心功能與何時使用 (When to Use)

- 將教學講稿、簡報文字或腳本批次轉換為自然語音（支援繁體中文台灣女聲/男聲）。
- 將 PowerPoint (PPT/PPTX) 簡報匯出之無聲影片搭配逐頁講稿合成有聲影片與同步字幕。
- 為現有無聲 MP4 影片或動畫添加旁白、音軌並同步影音時長（凍結尾幀/音訊補靜音）。
- 自動生成符合標準格式的 SRT 字幕檔，或使用 FFmpeg 燒錄硬字幕、封裝軟字幕。
- 解決 Edge-TTS 403 握手錯誤與連線端點修復。

---

## 第一部分：連上 TTS（核心環境與修復）

### 1. 問題背景
微軟關閉了舊版免費語音端點，直接呼叫舊端點會傳回 `403 Forbidden`。
透過改用新端點，音色與品質完全一致且無須申請 API Key。

### 2. 環境依賴
```bash
pip install edge-tts==7.2.8
```
- 系統必須安裝 `ffmpeg`（包含 `ffprobe`）並加入環境變數 `PATH`。

### 3. 連線 Patch（每次呼叫必加，置於程式最頂端）
```python
import edge_tts.communicate as comm
import edge_tts.voices as voices

KEY = "6A5AA1D4EAFF4E9FB37E23D68491D6F4"
BASE = "api.msedgeservices.com/tts/cognitiveservices"
comm.WSS_URL = f"wss://{BASE}/websocket/v1?Ocp-Apim-Subscription-Key={KEY}"
wh = dict(comm.WSS_HEADERS)
wh["Sec-WebSocket-Protocol"] = "synthesize"
comm.WSS_HEADERS = wh
voices.VOICE_LIST = f"https://{BASE}/voices/list?Ocp-Apim-Subscription-Key={KEY}"
vh = dict(voices.VOICE_HEADERS)
vh["Authority"] = "api.msedgeservices.com"
voices.VOICE_HEADERS = vh
```
> **關鍵注意**：Patch 目標為 `edge_tts.communicate.WSS_URL`，而非 `edge_tts.constants`。
> 若官方上游日後發布更新修復，可升級 `pip install -U edge-tts` 並移除此 Patch。

### 4. 連線探針（合成前必測）
在執行大量合成前，執行探針確認連線正常：
```python
import asyncio
async def main():
    c = comm.Communicate("測試語音連線。", "zh-TW-HsiaoChenNeural", rate="-5%")
    await c.save("probe.mp3")
asyncio.run(main())
```
使用 `ffprobe probe.mp3` 檢查是否有音訊、時長是否合理（24kHz mono mp3）。
偶發握手超時為網路正常現象，重試即可。

### 5. 音色推薦與文字前處理規範
- **推薦音色**：
  - 台灣女聲（教學/旁白首選）：`zh-TW-HsiaoChenNeural`
  - 台灣男聲：`zh-TW-YunJheNeural`
- **語速設定**：`rate="-5%"`（教學影片節奏穩重）；允許範圍 `-50%` 至 `+50%`。
- **講稿文字規範（極重要，避免發音錯誤或合成失敗）**：
  - **嚴禁網址**：刪除或轉為口語（如「請至官方網站查詢」）。
  - **避免連續英文字母**：`CO2` 寫為「二氧化碳」、`pH` 寫為「酸鹼值」、`DNA` 寫為「去氧核糖核酸」或口語字母標記、`NASA` 寫為「美國國家航空暨太空總署」。
  - **數字與符號轉繁體全中文**：
    - `20℃` ➜ 攝氏二十度
    - `3.5%` ➜ 百分之三點五
    - `1995~2004` ➜ 一九九五到二零零四年
    - `第 3 條` ➜ 第三條

### 6. 批量配音穩定模板
具備防限流間隔與斷線重試機制：
```python
import asyncio
import time
import os

async def one(text, out):
    if os.path.exists(out):  # 成品存在則跳過；若要重產需先刪除損壞檔案
        return
    for attempt in range(4):
        try:
            await comm.Communicate(text, "zh-TW-HsiaoChenNeural", rate="-5%").save(out)
            return
        except Exception as e:
            print(f"嘗試 {attempt+1} 失敗: {e}，20秒後重試...")
            time.sleep(20)
    raise RuntimeError(f"合成失敗: {out}")

async def batch_synthesize(pairs):  # pairs = [(文字內容, 輸出mp3路徑), ...]
    for text, out in pairs:
        await one(text, out)
        time.sleep(6)  # 間隔 6 秒以避免微軟端點限流
```

---

## 第二部分：三種常見任務流程

### 任務 A：PPT 簡報 ➜ 有聲影片＋字幕
1. **導出影片**：將 PPT/PPTX 另存為無聲 MP4 影片（或使用放映錄製），記錄每頁停留秒數。
2. **準備講稿**：逐頁撰寫講稿（遵守第一部分文字規範），每頁一個獨立 `.txt` 或字串項目。
3. **批量配音**：執行批量配音生成每頁獨立音訊 `slide_01.mp3`、`slide_02.mp3`...，使用 `ffprobe` 取得各音訊秒數 $d_i$。
4. **目標時長計算**：
   $$T_i = \frac{\lceil(d_i + 0.6) \times 30\rceil}{30}$$
   （預留 0.6 秒換頁呼吸停頓並對齊 30fps 影格整數）。
5. **精確對齊與拼接**：
   - 使用 FFmpeg `trim` 擷取該頁原始影像範圍。
   - 統一規格：`scale=1920:1080`、`fps=30`。
   - 尾幀凍結：使用 `tpad=stop_mode=clone:stop_duration=...` 延長畫面至 $T_i$。
   - 音訊補靜音：使用 `apad` 將音訊填充至 $T_i$。
   - 逐段驗證影像與音訊長度一致後，使用 `concat` 串接為完整影片。
6. **字幕生成**：依累加時長產出 SRT（見第三部分）。

### 任務 B：無聲 MP4 ＋ 分段講稿 ➜ 配音＋字幕
1. **固定段落匹配**：若 MP4 畫面節奏不可動，依每段時間起訖調整講稿字數；若音訊略短，可將尾端以靜音補齊。
2. **講完才切換（強烈推薦）**：將無聲 MP4 依段落拆切，比照任務 A 模式，各段按語音時長 $T_i$ 動態延長最後一幀畫面後再重新串接。

### 任務 C：已有影片，只加字幕
1. **準備 SRT**：產出符合標準格式之字幕檔案。
2. **軟字幕（可切換、不破壞畫質）**：
   ```bash
   ffmpeg -i 影片.mp4 -i 字幕.srt -c copy -c:s mov_text 輸出_軟字幕.mp4
   ```
3. **硬字幕（壓制燒錄）**：參見第三部分第 3 節。

---

## 第三部分：字幕 (SRT) 製作與燒錄

### 1. SRT 格式規範（UTF-8 編碼）
```srt
1
00:00:00,000 --> 00:00:07,400
各位同學好，這一節我們要做探討活動。

2
00:00:07,400 --> 00:00:16,100
本次探討有兩個活動……
```
- 時間戳記：`HH:MM:SS,mmm`（時:分:秒,毫秒）。
- 每句長度建議控制在 15~20 字以內，語意完整不斷詞。

### 2. 依段落時長自動計算 SRT
設各段目標時長為 $T_i$，段落文字為 $W_i$：
- 起始時間：$S_0 = 0$
- 結束時間：$S_{i+1} = S_i + T_i$
- 每一段字幕區間為 $S_i \to S_{i+1}$。
- 檔案建議以 `utf-8-sig`（帶 BOM）儲存，確保 Windows 播放器相容性。

### 3. FFmpeg 燒錄硬字幕命令
#### macOS / Linux：
```bash
ffmpeg -i 輸入.mp4 -vf "subtitles=字幕.srt:force_style='FontName=PingFang TC,FontSize=20,PrimaryColour=&H00FFFFFF,OutlineColour=&H80000000,BorderStyle=1,Outline=1,MarginV=40'" -c:a copy 輸出.mp4
```

#### Windows（注意路徑冒號跳脫）：
```bash
ffmpeg -i 輸入.mp4 -vf "subtitles='C\:/path/字幕.srt':force_style='FontName=Microsoft JhengHei,FontSize=20,PrimaryColour=&H00FFFFFF,OutlineColour=&H80000000,BorderStyle=1,Outline=1,MarginV=40'" -c:a copy 輸出.mp4
```
> **排錯建議**：直接將 `.srt` 與 `.mp4` 置於同目錄執行，避免路徑跳脫錯誤；標點符號請使用全形，避免缺字。

### 4. 替換/覆蓋音軌
保留原畫面，直接置換為旁白音訊：
```bash
ffmpeg -i 無聲.mp4 -i 旁白.mp3 -map 0:v -map 1:a -c:v copy -c:a aac -shortest 輸出.mp4
```

---

## 第四部分：常見錯誤與故障排查速查表

| 症狀 / 報錯 | 原因分析 | 解決方法 |
|---|---|---|
| `WSServerHandshakeError: 403` | 未注入連線 Patch，或端點失效 | 檢查腳本開頭是否加入第一部分 §3 的 Patch；若上游端點再次更新需重新抓包修正 URL/Key |
| 零星配音連線超時中斷 | 網路連線偶發波動 | 正常現象，批次模板內已包含 retry 重試機制 |
| 影音播放越到後面越不同步 | 某分段之影像與音訊長度存在微小誤差 | 逐段使用 `ffprobe` 檢查影音 duration 差，確保每段精確填充至 $T_i$ 再行串接 |
| 字幕出現亂碼 | 編碼不是 UTF-8 | 將 SRT 重新存為 UTF-8 或 `utf-8-sig` |
| 燒錄字幕時報告字型或路徑錯誤 | 路徑跳脫錯誤或字型名稱不符 | 將 SRT 置於當前目錄直接引用檔名；macOS 建議 `PingFang TC`，Windows 建議 `Microsoft JhengHei` |
| PowerShell 報錯 `&&` 或語法錯誤 | PowerShell 不支援 bash 語法串接 | 改分次單行執行，或撰寫 Python 腳本調用 `subprocess` |
| 雲端硬碟掛載磁碟機消失 | Google Drive 服務休眠或當機 | 重新啟動 `GoogleDriveFS`，等待 1~2 分鐘後重新讀取 |
