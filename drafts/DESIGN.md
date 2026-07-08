# Design

## Source of truth
- Status: Draft
- Last refreshed: 2026-07-08
- Primary product surfaces: 月度流動性儀表板、月報匯出頁、指標明細頁、資料品質檢查頁
- Evidence reviewed:
  - `美股AI泡沫-流動性槓桿診斷與破裂點推演-分析報告.md`
  - `美股AI泡沫-流動性槓桿診斷與破裂點推演.md`
  - `docs_references/`，目前未含可用設計或資料規格檔

## Brand
- Personality: 冷靜、研究導向、可稽核、避免交易喊單語氣。
- Trust signals: 每個訊號顯示資料截止日、來源、門檻、月變化、判定理由。
- Avoid: 不能用單一大分數取代細節；不能把情境推演包裝成精準預測；不能使用投資建議語氣。

## Product goals
- Goals:
  - 將報告中的三層流動性框架轉成可每月更新的監測介面。
  - 讓使用者在第一屏快速回答：流動性是否更緊、槓桿是否去化、AI capex 是否仍支撐 EPS、加密熱錢是否回流。
  - 將 Bear/Base/Bull 情境與明確的偏空確認、偏多否證門檻綁定。
- Non-goals:
  - 不做即時交易訊號。
  - 不提供買賣建議或部位建議。
  - 不嘗試精準預測 NASDAQ 頂點日期。
- Success signals:
  - 每月可在同一格式下產出狀態判讀。
  - 任一指標的紅黃綠狀態都能回查公式、來源與門檻。
  - 指標更新延遲或缺值會被清楚標示。

## Personas and jobs
- Primary personas: 總經投資研究者、資產配置決策者、風控/研究助理、需要追蹤 AI 泡沫風險的主題研究者。
- User jobs:
  - 每月判斷風險是否從估值壓力轉成流動性或信用壓力。
  - 找出哪些訊號正在支持或否證原始 Bear/Base 論點。
  - 將更新結果整理成可分享的月度摘要。
- Key contexts of use: 月底或月初研究會議、風險委員會前準備、個人研究筆記更新。

## Information architecture
- Primary navigation: 總覽、三層水位計、指標明細、情境推演、資料品質、月報匯出。
- Core routes/screens:
  - `/dashboard/monthly-liquidity`
  - `/dashboard/monthly-liquidity/indicators`
  - `/dashboard/monthly-liquidity/scenarios`
  - `/dashboard/monthly-liquidity/data-quality`
  - `/reports/monthly-liquidity/:month`
- Content hierarchy: 全局風險狀態先於單一指標；三層流動性先於情境；資料品質與來源永遠可追溯。

## Design principles
- Principle 1: 先呈現可否證訊號，再呈現敘事。
- Principle 2: 每個紅色狀態都必須能被資料、門檻與時間序列支撐。
- Tradeoffs: 以月度穩定性優先於即時性；以門檻透明優先於複雜黑箱模型。

## Visual language
- Color: 使用語義色，紅色代表偏空確認，黃色代表中性觀察，綠色代表偏多否證，灰色代表資料缺失或延遲。
- Typography: 研究儀表板風格，標題清楚但不誇張，數字與表格需易掃讀。
- Spacing/layout rhythm: 第一屏緊湊，後續區塊使用一致欄寬與表格節奏。
- Shape/radius/elevation: 控制在 8px 以下，避免行銷頁式大卡片與裝飾。
- Motion: 僅用於狀態切換與資料載入，不使用吸睛動畫。
- Imagery/iconography: 使用簡潔圖示表示政策、市場信用、投機熱錢與 AI 盈餘，不使用抽象裝飾圖。

## Components
- Existing components to reuse: 目前 repo 未提供前端元件。
- New/changed components:
  - RegimeHeader
  - FourQuestionSummary
  - LiquidityLayerGauge
  - IndicatorStatusTable
  - ScenarioMatrix
  - DataFreshnessBadge
  - MonthlyReportExporter
- Variants and states: normal、warning、bear-confirmed、bull-invalidated、stale、missing。
- Token/component ownership: 前端實作時需建立語義色 token、資料狀態 token、表格密度 token。

## Accessibility
- Target standard: WCAG 2.2 AA。
- Keyboard/focus behavior: 所有篩選、月份選擇、匯出與明細展開可鍵盤操作。
- Contrast/readability: 紅黃綠不能作為唯一判讀方式，需搭配文字標籤。
- Screen-reader semantics: 指標表格需有欄位標題與狀態文字。
- Reduced motion and sensory considerations: 遵守 prefers-reduced-motion。

## Responsive behavior
- Supported breakpoints/devices: 桌面優先，支援平板；手機提供閱讀版但不要求完整分析操作。
- Layout adaptations: 桌面使用 12 欄；平板改 2 欄；手機改單欄與可橫向捲動表格。
- Touch/hover differences: hover tooltip 在觸控裝置改為點擊展開。

## Interaction states
- Loading: 顯示上次成功更新月份與目前抓取中的資料源。
- Empty: 顯示尚未建立該月份資料，並提供匯入 CSV 或手動輸入入口。
- Error: 顯示失敗來源、錯誤時間、重試動作與是否沿用上月值。
- Success: 顯示本月更新完成、資料截止日與已變更訊號數。
- Disabled: 未完成資料品質檢查前，月報匯出按鈕不可用。
- Offline/slow network, if applicable: 允許讀取本地快取，但所有快取值標示 stale。

## Content voice
- Tone: 分析式、保留不確定性、避免確定式預言。
- Terminology: 延用報告術語，例如政策流動性、市場信用流動性、投機流動性、偏空確認、偏多否證。
- Microcopy rules: 每個判讀句需包含「因為哪個指標觸發哪個門檻」。

## Implementation constraints
- Framework/styling system: 未指定，規格應可交由任一前端框架實作。
- Design-token constraints: 必須建立狀態色與資料品質狀態，不硬寫散亂顏色。
- Performance constraints: 月度資料量小，優先要求可稽核與可匯出。
- Compatibility constraints: CSV 匯入與本地 JSON 快取需可作為第一版資料管線。
- Test/screenshot expectations:
  - 指標門檻單元測試。
  - 缺值與 stale 狀態測試。
  - 桌面、平板、手機主要畫面截圖檢查。

## Open questions
- [ ] 實作平台是 Web app、Google Sheets/Looker Studio、Notion/Obsidian，還是 Python Streamlit？
- [ ] 指標資料來源要採自動 API、手動 CSV，還是混合模式？
- [ ] 月報匯出格式要 Markdown、PDF、HTML，還是三者都要？
- [ ] 情境機率是否允許手動調整，或完全由門檻規則生成？
