---
name: remotion-video-creator
description: 使用 React 與 Remotion 製作、構思、設計及渲染高品質動態影片與字幕動畫。當使用者提及「製作影片」、「Remotion 影片」、「自動化生成影片」、「動態文字/字幕動畫」、「Remotion 渲染」或使用 Remotion 相關技術時啟動。
argument-hint: "[topic] [duration-seconds] [fps]"
metadata:
  author: Antigravity
  version: "1.0.0"
---

# Remotion Video Creator — 程式化影片製作 Skill

使用 React 與 Remotion 框架，將影片製作、字幕動畫、圖表視覺化與音效整合轉化為程式碼驅動的自動化工作流。

## 何時使用 (When to Use)

- 使用 React / Remotion 建立程式化動態影片、短影音 (Reels/Shorts/TikTok)、簡報影片。
- 設計高品質的文字動畫、字幕特效、資料視覺化圖表影片。
- 自動化批次生成個人化影片、產品宣傳片或社群媒體動畫。
- 透過 `npx remotion render` 將 React 組件渲染為 MP4 影片或 GIF。

## 工作流程與核心步驟 (Workflow & Core Steps)

### 1. 專案初始化 (Initialization)
若本機尚未建立 Remotion 專案，協助使用者初始化：
```bash
npx create-video@latest
# 或在現有 React / Node.js 專案中安裝：
# npm install remotion @remotion/cli @remotion/player
```

### 2. 構思與架構設計 (Concept & Architecture)
在寫程式之前，明確定義影片規格：
- **解析度 (Resolution)**：
  - 16:9 橫向 (1920x1080)
  - 9:16 縱向 (1080x1920)
  - 1:1 正方形 (1080x1080)
- **幀率 (FPS)**：通常為 `30` 或 `60`。
- **總幀數 (DurationInFrames)**：`秒數 * FPS`（例如 10 秒 30fps = 300 幀）。

### 3. 組件編寫規範 (Remotion Coding Standards)

#### A. 核心導出點 (Root Configuration)
在 `src/Root.tsx` 中使用 `<Composition>` 宣告影片：
```tsx
import { Composition } from 'remotion';
import { MyVideo } from './MyVideo';

export const RemotionRoot = () => {
  return (
    <Composition
      id="MyVideo"
      component={MyVideo}
      durationInFrames={300}
      fps={30}
      width={1920}
      height={1080}
      defaultProps={{
        titleText: '歡迎使用 Remotion',
        titleColor: '#0070f3',
      }}
    />
  );
};
```

#### B. 核心 Hook 與動畫邏輯 (Hooks & Animations)
- `useCurrentFrame()`：取得當前播放到的幀數（0, 1, 2, ...）。
- `useVideoConfig()`：取得影片參數（`fps`, `durationInFrames`, `width`, `height`）。
- `interpolate()`：數值映射（例如從第 0 到 30 幀，透明度從 0 變為 1）。
- `spring()`：彈簧物理動畫效果。

**範例 - 淡入與彈跳文字 (Fade In & Spring Text)**:
```tsx
import { AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';

export const MyVideo = ({ titleText }: { titleText: string }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // 透明度淡入 (0 到 15 幀)
  const opacity = interpolate(frame, [0, 15], [0, 1], {
    extrapolateRight: 'clamp',
  });

  // 縮放彈簧動畫
  const scale = spring({
    frame,
    fps,
    config: { damping: 12 },
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#0f172a', justifyContent: 'center', alignItems: 'center' }}>
      <h1
        style={{
          opacity,
          transform: `scale(${scale})`,
          color: '#ffffff',
          fontSize: 80,
          fontFamily: 'sans-serif',
        }}
      >
        {titleText}
      </h1>
    </AbsoluteFill>
  );
};
```

#### C. 時間軸與分鏡組合 (Sequence & AbsoluteFill)
- 使用 `<Sequence from={startFrame} durationInFrames={length}>` 來編排不同片段登場的時間點。
- 使用 `<AbsoluteFill>` 作為全螢幕背景或覆蓋層容器。

```tsx
import { AbsoluteFill, Sequence } from 'remotion';

export const MainStory = () => {
  return (
    <AbsoluteFill>
      {/* 0 秒到 3 秒 (0 ~ 90 幀) */}
      <Sequence from={0} durationInFrames={90}>
        <IntroScene />
      </Sequence>

      {/* 2.5 秒開始 (75 幀) 到 10 秒 */}
      <Sequence from={75} durationInFrames={225}>
        <MainContentScene />
      </Sequence>
    </AbsoluteFill>
  );
};
```

### 4. 預覽與渲染輸出 (Preview & Render)

- **本機即時預覽工作室 (Studio UI)**：
  ```bash
  npx remotion studio
  # 或 npm start (視 package.json 設定)
  ```
- **渲染匯出 MP4 影片**：
  ```bash
  npx remotion render src/index.ts MyVideo out/video.mp4
  ```
- **渲染成短動圖 GIF**：
  ```bash
  npx remotion render src/index.ts MyVideo out/animation.gif
  ```

---

## 最佳實踐與防護建議 (Best Practices)

1. **避免在 render 期間呼叫非同步外部 API**：預先準備好 `props` 或在進入序列前載入完畢。
2. **遵守 100% 確定性 (Determinism)**：
   - 禁用 `Math.random()`，改用基於 `frame` 或固定 seed 的亂數演算法。
   - 禁用 `Date.now()`，改用 `frame / fps` 計算相對時間。
3. **音訊與音樂**：使用 `@remotion/media-utils` 或 Remotion 內建 `<Audio src="..." />` / `<Video src="..." />` 控制起止時間與 Fade in/out。
