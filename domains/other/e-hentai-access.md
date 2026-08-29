---
title: E-Hentai/ExHentai 進入機制與使用者帳號狀態（2026）
scope: domains/other
status: active
created: 2026-08-29
updated: 2026-08-29
tags:
  - ehentai
  - exhentai
  - igneous
  - bronze-star
  - donation
  - bitopro
---

# 使用者帳號狀態（2026-08-29 實測驗證）

- 帳號 `ipb_member_id=4626058`、`ipb_pass_hash=b07367571d01256b76f78698d23b1193`（亞洲 IP 註冊老號；捐款前無里站資質：249 國掃描全 mystery）
- 2026-08-29 捐款約 $24 BCH（NT$750 經 BitoPro 購買）→ **銅星**（cookie `star=1-…`、`hath_perks=a.t1.m1`）
- 捐款後驗收：表站 e-hentai.org **搜尋包含全部里站內容**（loli 搜尋 2,282,361 筆；里站專屬 `rape` 標籤也有結果）；里站書（如 `4155412`）可在表站直接開啟；exhentai.org 直連（台灣 IP）正常
- 每日使用：**只用表站＝零維護**（瀏覽器自動登入）；ex.org 直連需每月刷新 igneous
- 目前 igneous：`up69bp2u2lukmt1uv`（2026-08-29 經 CF Pages 自美國 IP 取得，**2026-09-28 過期**）
- 手機：Android 上裝 EhViewer NekoInverter fork v1.8.13，cookie 登入（三值），画廊站點設 e-hentai

# 里站進入機制（2026 實測）

- 無資質＝**純白屏**（JS 殼、HTTP 200、body 0 bytes）；sad panda 圖 2024–25 cookie 改版後已取消
- 資質＝註冊 IP 判定＋捐款解鎖；亞洲 IP 老號可捐款解鎖
- `igneous` 為 **17/19 位動態值**、有效約 30 天；舊 9 位值全死；過期會被覆寫 `mystery`
- 免費代取工具（POST `{"ipb_member_id":"...","ipb_pass_hash":"..."}`）：
  - `https://exhentai-igneous-generator.pages.dev/api`（穩定，回 set-cookie 含 igneous/sk/star/hath_perks）
  - `https://exhentai-cookie.vercel.app/api/proxy`（工具自身 IP 可能擋掉，回 mystery，不代表帳號有問題）
- 刷新流程：工具取新值 → 寫入瀏覽器 cookie（Brave CDP `Network.setCookie`，domain `.exhentai.org`，含 `sk`/`star`/`hath_perks`）或 EhViewer cookie 登入重貼
- 官方**無書面保證**「銅星＝里站准入」（ex.org 白屏無文字、官方 wiki 銅星福利表未列 ex access）；依據＝社群多來源交叉驗證＋實測

# 銅星（捐款）流程

- e-hentai.org → My Home → `bitcoin.php` → 複製**個人專屬 cashaddr `q...` 收款地址**（只能存幣不能取，貼錯找不回）→ 交易所匯 BTC/BCH（BCH 手續費低、官方推薦）→ 回頁面填 USD 金額（銅星 $20）→ Apply，自動核帳自動上星，不用 PM
- 匯率：max(7 日、24 時) 均價 × 1.10 加碼，匯 $19–20 可上 $20
- 銅星福利（sinner.ehentai.info）：表站去廣告、預覽 10 行、圖片配額 ×2、隱私圖庫、**表站觀看里站內容**

# 台幣→BCH（2026 台灣環境）

- **BitoPro（幣託）一條龍**：註冊→KYC（台胞身分證+人臉+綁銀行，1~3 天）→遠東商銀虛擬帳號/超商入金→一鍵買幣（免手續費）→提幣（network=BCH）
- 使用者 BitoPro 帳戶已 KYC（2026-08；銀行＝國泰；推薦碼 2720985376 → 180 天手續費 8 折）
- 匯款鐵律：本人帳戶、網銀/ATM、附言空白；❌臨櫃/第三方代匯/企網銀
- 交易所分類：BitoPro/MAX＝本土（收台幣）；Binance/OKX/Bybit＝國際（需 USDT 轉匯）
- 銀行風控：小額（<10 萬）幾乎不卡；嚴格者＝凱基（審核 3 個月）、郵局（限額）、公股；**Bankee（遠東數位帳戶）最穩**（與交易所信託帳戶同行免跨行）
- 法規：《虛擬資產服務法》2026/6/30 三讀通過，金管會主管，牌照制最快 2027 Q1

# 手機 App 注意事項

- E 站論壇有 Cloudflare：App 內帳密登入可能 403 → 用 **cookie 登入或網頁登入**
- 下載只走 GitHub Releases（EhViewer-NekoInverter/EhViewer）；「EhViewer 官網/台灣版」等搜尋結果多為 SEO 洗版站改裝包
- 画廊站點設 **e-hentai**：銅星後內容完整且免 30 天 cookie 維護；要純里站才切 exhentai