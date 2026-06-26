# Pathors Color Token · Token Bench

把 Figma 的品牌標準色,轉成 **shadcn/ui (Tailwind v4 · OKLCH)** 的 `globals.css`,並即時預覽元件、一鍵推回 repo 給工程團隊。

**線上工具:** https://lintingyii.github.io/Pathors_ColorToken/figma-to-shadcn.html

---

## 這是什麼

設計端在 Figma 有一套品牌色;工程端用 shadcn/ui。傳統做法是丟一份 hex 色票給工程,但這樣每次改色都要人工去比對、替換。

Token Bench 改變交付的單位:**不是交付 hex 清單,而是把品牌色對映到 shadcn 的語意 token**(`primary` / `secondary` / `muted` / `border`…),以 OKLCH 輸出成一份 `globals.css`。工程只要更新這一個檔,所有元件自動套用——改色從「逐處替換」變成「改一處、全站生效」。

`globals.css` 等於設計與工程之間的一份契約:兩端使用同一套語意命名,中間不需要任何人翻譯「這個藍是要當什麼用」。

---

## Repo 內容

| 檔案 | 說明 |
| --- | --- |
| `figma-to-shadcn.html` | Token Bench 工具本體(單一檔、免建置) |
| `globals.css` | 目前交付給工程的品牌主題(OKLCH semantic tokens) |
| `tokenbench-backup.json` | (選用)設計端的存檔紀錄,供跨電腦同步,與工程無關 |

---

## 給工程團隊:如何接入 `globals.css`

最新版固定在這個 raw 連結:

```
https://raw.githubusercontent.com/lintingyii/Pathors_ColorToken/refs/heads/main/tokenbench-backup.json
```

接入步驟:

1. 取得最新 `globals.css`(raw 連結或 `git pull`)。
2. 把其中的 **`:root` 與 `@theme inline`** 兩段貼進你專案的 `app/globals.css`,取代原本的色彩 token 區塊。保留你既有的 `@import "tailwindcss";` 與 base layer。
3. 完成。shadcn 元件會自動讀取這些變數,**不需要改任何元件程式碼**。

```css
/* app/globals.css */
@import "tailwindcss";

/* ↓↓↓ 貼上 Token Bench 交付的 :root 與 @theme inline ↓↓↓ */
:root {
  --background: oklch(...);
  --primary: oklch(...);
  /* … */
}
@theme inline {
  --color-primary: var(--primary);
  /* … */
}
```

額外的品牌色(非 shadcn 標準語意)以 utility 取用:

```html
<div class="bg-brand-deepblue text-brand-skyblue-light">…</div>
<span class="text-brand-accent-green">success</span>
<!-- 圖表色:bg-chart-1 ~ bg-chart-6 -->
```

> 目前 `globals.css` **僅含淺色模式(`:root`)**,深色模式待後續設計。

---

## 給設計師:如何使用工具

打開[線上工具](https://lintingyii.github.io/Pathors_ColorToken/figma-to-shadcn.html),然後:

1. **上傳** Figma 匯出的 Variables JSON(拖入或點上傳框),或點「載入範例」先看效果。
2. 工具自動把每個顏色轉成 **OKLCH**,並用關鍵字啟發式**自動對映**到 shadcn 語意 token。
3. 左側每個 token 都有下拉選單可**手動改派**;圓角用滑桿調整。
4. 右側「**元件預覽**」分頁用你的顏色即時渲染 buttons / cards / forms / tabs / table / chart 等元件;改任何顏色都會即時反映。
5. 「**globals.css**」分頁可**直接點色塊改色**(OKLCH 自動重算),token 名稱維持唯讀。
6. 完成後用「**複製**」/「**下載 .css**」/「**純文字**」取得結果。

### 從 Figma 匯出 token

在 Figma 把 Variables(色彩)以 **JSON(DTCG 格式)** 匯出,確保每個 token 帶有實際色值(hex 或 components)。工具會遞迴解析所有群組與巢狀 token。

---

## 跨電腦同步 / 推回 repo

工具左側「雲端同步」區,展開「**GitHub repo 聯動**」:

- 填 `owner` / `repo` / 分支 / token,即可:
  - **↻ 載入紀錄** — 從 repo 拉回設計端的存檔紀錄
  - **⤴ 推送紀錄** — 把存檔紀錄寫成 `tokenbench-backup.json`
  - **⤴ 推送 globals.css 到 repo** — 把目前的 CSS 直接 commit 回 repo,工程端即可 pull 最新
- 也可用「**從網址載入**」貼上任何 raw JSON 連結同步紀錄。


**owner:** `lintingyii`

**repo:** `Pathors_ColorToken`

**分支:** `main`

**紀錄路徑保留:** `tokenbench-backup.json`

**CSS 路徑保留:** `globals.css`

**⚠️ Please ask for token**


### Token 注意事項

- 推送(寫入)需要 GitHub **fine-grained personal access token**,權限請限定**只有這個 repo**、`Contents` 設為 **Read and write**:https://github.com/settings/tokens?type=beta
- token **不會寫進這個 HTML 檔**,只存在你本機瀏覽器;共用 / 公用電腦請勿勾「記住 token」。
- token 全程走 `Authorization` header,絕不放進網址。

---

## 對映規則(設計決策)

- **語意 token** 是設計與工程的共同語言;品牌色對映到 `primary` / `secondary` / `muted` / `accent` / `border` 等。
- **鮮明品牌色**(SkyBlue、yellow、pink、green)另開 `--brand-*` 擴充 token,不塞進語意上應低調的 `--accent`;同時鋪進 `--chart-1…6`。
- **foreground 類** token 依底色對比自動選黑/白字。
- **`destructive`** 因品牌色票無紅色,套用通用紅。
- 背景採 NeutralColor、卡片留白,讓淺色模式有層次。

---

## 開發 / 部署

- 單一自包含 HTML,無相依套件、免建置。本機開發:`python3 -m http.server`。
- 已透過 GitHub Pages 部署(Settings → Pages → `main` / root)。

---

## 限制 / 待辦

- [ ] 深色模式 token(`.dark`)
- [ ] Figma **別名(alias)**引用的解析(目前只讀帶實際色值的 token)
- [ ] 紀錄同步為**單向拉取**;雙向更新需手動推回 repo
- [ ] code 面板的色票回傳 hex,OKLCH 由 hex 推算(尚無直接編輯 L/C/H 的模式)
