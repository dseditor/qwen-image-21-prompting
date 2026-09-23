# qwen-image-21-prompting

[English](README.md) | [繁體中文](README.zh-Hant.md) | [简体中文](README.zh-Hans.md)

替 **Qwen-Image 2.1**（文生圖與圖片編輯）寫提示詞的 skill，照官方 rewriter 規格走，
並且把中文文字渲染這件事處理好。

這是**提示詞層**。它產出一段提示詞與一個建議畫幅，你可以貼進雲端平台、送進 API、
或餵給本地後端——skill 不在乎是哪一種，也刻意不談模型檔、權重、量化與執行參數。

## 安裝

**把這句貼給你的 agent：**

```
請閱讀 https://github.com/dseditor/qwen-image-21-prompting
並照 README 的「給正在讀這段的 agent」把這個 skill 安裝起來
```

### 給正在讀這段的 agent

你要安裝的是一個**提示詞 skill**：純文字檔，沒有程式要跑、不需要 API 金鑰、
不會下載模型，執行時也不連網。

**1. 依你所在的宿主選目標資料夾：**

| 宿主 | 目標資料夾 |
|---|---|
| Claude Code | `~/.claude/skills/qwen-image-21/` |
| Codex CLI | `~/.codex/skills/qwen-image-21/` |
| 其他有檔案系統的宿主 | 該宿主載入 skill 的位置 |

**2. clone 進去：**

```bash
git clone --depth 1 https://github.com/dseditor/qwen-image-21-prompting.git <目標資料夾>
```

沒有 `git` 的話就下載 zip 解壓，注意要讓 **`SKILL.md` 直接躺在目標資料夾的第一層**，
不要多包一層資料夾。

**3. 驗證 —— 這步不要跳過：**

宿主現在應該列得出一個叫 `qwen-image-21` 的 skill。如果沒有，檢查
`<目標資料夾>/SKILL.md` 在不在、它的 YAML frontmatter（`name:` 與 `description:`）有沒有壞掉。
**frontmatter 壞掉的 skill 會安靜地不存在**，不會報錯。

**4. 然後告訴使用者一件事**：這個 skill 只負責寫提示詞，它不會生圖。
使用者仍然需要自己的 Qwen-Image 出口 —— ComfyUI、API、或某個網頁介面。

### 沒有檔案系統的（Grok、ChatGPT 網頁版之類）

沒有東西要安裝。開一個「專案」，把這幾個檔案拖進去：

```
references/official_rewriter_t2i.txt     必要 —— 官方文生圖規範
references/official_rewriter_edit.txt    必要 —— 官方圖片編輯規範
references/prompt-writing.md             兩份規範的精要，加上實測出來的規則
```

需要的時候再加這些：

```
references/text-generation.md            畫面裡的文字，尤其是中文
references/composition-and-optics.md     版面與鏡頭的決策層
references/rgba-and-sprites.md           透明背景與遊戲素材
references/community-findings.md         實測結果 —— 包含【不成立】的那些
```

然後直接描述你要的畫面就好，模型會照專案檔案裡的規範走。

## 裡面有什麼

```
SKILL.md                          入口——路由，以及唯一值得重複的那條規則
references/
  official_rewriter_t2i.txt         官方 rewriter 系統提示詞（文生圖），原文未改
  official_rewriter_edit.txt        官方 rewriter 系統提示詞（圖片編輯），原文未改
  prompt-writing.md                 兩份規格的精要，加上實測出來的寫法規則
  text-generation.md                畫面文字，尤其是中文
  community-findings.md             宣稱的能力實測——包含【不成立】的那些
  delivery-and-verification.md      送出前的接線核對、拿回圖之後的驗收、A/B 紀律
```

**參考文件只留英文，這是刻意的。** 它們原本有三種語言，而同一條結論一旦被修正，
三份副本就會開始漂移——而且是無聲的，因為沒有人會去 diff 一份翻譯。
改成翻譯 README：README 是人讀一次的東西，參考文件是 agent 每次都要讀的東西。

## 濃縮版

**用一個問題挑規格——有沒有輸入圖？**
沒有輸入圖 → 走文生圖 rewriter；有輸入圖 → 走編輯 rewriter。
把對應的檔案整份讀完，照順序執行：那份規格是一串有序步驟，**後面的步驟不會回頭修改前面的**。

**畫面裡要有文字時，只做兩件事。** 在提示詞裡用「佔畫面的比例」講出最小字級，
並且告訴呼叫者：文字若長壞了就說一聲，我換一個詞重生。
固定 seed ＋ 換掉壞掉的那個詞，是唯一穩定划算的文字技巧。

**沒有任何自動化的東西能驗中文字形。** LLM、OCR、VLM 都用上下文消解歧義——
而那正是「錯字剛好符合預期詞義」得以藏住的同一個機制。
文字非對不可的時候，要嘛讓人去看，要嘛用真字型合成上去。

## 實測結論都有日期

`community-findings.md` 記的是量了什麼、什麼時候量的、怎麼量的——
包含那些被宣稱、但實際上站不住的能力（360° 全景接縫合不起來；
透明背景對一般物件乾淨，對文字會劣化）。
在你自己的組態上重驗過再依賴它，**尤其是否定結論**——後續版本很可能已經修好了。

## 提示詞不是全部的工作

`delivery-and-verification.md` 講提示詞前後那兩段：
模型有某個能力，不代表你正在呼叫的那個端點把它接出來了
（實例：同一台機器，文生圖實際輸出 1152×2048，它自己的編輯路徑卻無聲忽略了送進去的尺寸）；
以及圖回來之後要在**未經修飾的原圖**上驗收，不是看預覽。
**已經準備好的請求，不是已經生成的圖片。**

## 出處

`references/official_rewriter_*.txt` 是 Qwen-Image 2.1 官方 rewriter 系統提示詞，原文收錄未修改。
其餘都是自己動手測出來的筆記，另有一份清楚標註出處的第三方 benchmark 摘要在 `community-findings.md`。

`delivery-and-verification.md` 借鑒了 Craft Skills 的
[`qwen-image-gen`](https://github.com/ZSeven-W/craft-skills/tree/main/skills/qwen-image-gen)
在交付面的整理框架（MIT，© Fini.Yang）。裡面的實測數據是我們自己的。
