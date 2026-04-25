# TradingView MCP (日本語版)

こちらは改良版・日本語フォークです。必要な情報はすべて以下にまとめてあります。

オリジナルは [@tradesdontlie](https://github.com/tradesdontlie) 氏による [tradingview-mcp](https://github.com/tradesdontlie/tradingview-mcp) で、その土台に全面的にクレジットが帰属します。本フォークでは、日本語化及びWindows環境におけるTradingViewデスクトップが起動しない場合、ChromeにてTradingViewを起動する迂回ルートを追加しています。

> [!WARNING]
> **TradingView Inc. および Anthropic とは一切関係ありません。** 本ツールは、ローカルで動作している TradingView Desktop アプリに Chrome DevTools Protocol 経由で接続します。利用前に必ず[免責事項](#免責事項)をご確認ください。

> [!IMPORTANT]
> **有効な TradingView 有料サブスクリプションが必要です。** 本ツールは TradingView の有料機能を回避するものではありません。あくまで、お使いのマシン上で既に動作している TradingView Desktop アプリの内容を読み取り、操作するだけです。

> [!NOTE]
> **すべてのデータ処理はローカルで完結します。** 外部に何も送信されません。TradingView のデータがマシンの外に出ることはありません。

---

## ワンショットセットアップ

### 事前準備
　(Windows / macOS 両対応・Node.js 自動インストール)
 
1. **Windowsの方**: Google Chrome をインストール済みにしておく
2. **Macの方**: 特に追加準備なし(TradingView Desktop版を使うため)
3. TradingView の有料プランにログインできる状態にしておく
4. Claude Code を起動して、下記のワンショットプロンプトを丸ごと貼り付ける

※ Node.js は Claude Code が自動でインストールします(Windows: `winget` / Mac: `brew`)。

### ワンショットプロンプト(こちらをすべて Claude Code にコピペ)

````
あなたはこれからClaude Codeに、TradingView MCP連携をワンショットでセットアップしてください。途中でユーザーに聞かないと進めない箇所(TradingViewへのログイン等)以外は、全部あなたが代行してください。
よろしくお願いします。

【最初にやること】
- 現在のOSを判定する(`uname` または環境変数で判定。Windowsなら %OS% / Macなら `uname` が `Darwin`)
- OSに応じて下記の【Windows手順】または【Mac手順】を選んで実行
- 不明なOS(Linux等)の場合は、ユーザーに「Windows / Mac のみサポートです」と伝えて中断

═══════════════════════════════════════
【Windows手順】
═══════════════════════════════════════

【前提環境】
- OS: Windows 10/11
- Google Chrome がインストール済み(`C:\Program Files\Google\Chrome\Application\chrome.exe` を想定)
- winget が利用可能(Windows 10 1809以降は標準搭載)

■ ステップ0: Node.js の有無を確認、なければ自動インストール
- `node -v` を実行。18以上ならスキップ
- 未導入なら: `winget install -e --id OpenJS.NodeJS.LTS --accept-source-agreements --accept-package-agreements`
- インストール後、新しいシェルで `node -v` を再確認(PATH反映のため)
- それでもダメなら `C:\Program Files\nodejs\node.exe` を直接叩く
- winget自体が無い古いWindowsの場合のみ、https://nodejs.org/ から手動インストールを案内して中断

■ ステップ1: リポジトリのクローンと依存インストール
- ユーザー名を `echo %USERNAME%` で取得
- `C:\Users\<ユーザー名>\tradingview-mcp-jackson` に
  https://github.com/LewisWJackson/tradingview-mcp-jackson.git をクローン
- そのディレクトリで `npm install` を実行

■ ステップ2: MCP設定への追記
- `C:\Users\<ユーザー名>\.claude\.mcp.json` を読み込み(なければ新規作成)
- 既存の `mcpServers` を**上書きしない**。マージする形で以下を追加:

  {
    "mcpServers": {
      "tradingview": {
        "command": "node",
        "args": ["C:\\Users\\<ユーザー名>\\tradingview-mcp-jackson\\src\\server.js"]
      }
    }
  }

- JSON内のパスはバックスラッシュ2つ(`\\`)でエスケープ

■ ステップ3: rules.json の準備
- `~/tradingview-mcp-jackson/rules.example.json` を `rules.json` にコピー
- ユーザーに「後でルールを編集してください」と一言

■ ステップ4: CDP有効ChromeでTradingView Webを起動
- 重要: TradingView Desktop版は使わない。`tv_launch` は呼ばないこと(Windowsでは失敗)
- 既存のChromeが起動中だとCDPポートが開かないので、**別プロファイル**で起動:

  "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="C:\temp\chrome-cdp-tv" "https://www.tradingview.com/chart/"

- 毎回手動で打たなくて済むように、デスクトップに `start-tv-cdp.bat` を作成:

  @echo off
  start "" "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="C:\temp\chrome-cdp-tv" "https://www.tradingview.com/chart/"

- ユーザーに「初回のみTradingViewにログインしてチャートを開いてください。次回以降は `C:\temp\chrome-cdp-tv` にセッションが保存されるので自動ログインになります」と案内し、ログイン完了を待つ

→ ステップ5以降(Claude Code再起動 → 接続確認)は【共通手順】へ

═══════════════════════════════════════
【Mac手順】
═══════════════════════════════════════

【前提環境】
- OS: macOS (Apple Silicon / Intel どちらもOK)
- TradingView Desktop版を使う(Macでは公式アプリが普通に使えるため)

■ ステップ0: Homebrew と Node.js を確認、なければ自動インストール
- `brew -v` を実行。無ければ Homebrew をインストール:

  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

- (インストール時にユーザーパスワード入力が必要な場合あり。その時はユーザーに案内)
- `node -v` で18以上を確認。無ければ: `brew install node`

■ ステップ1: TradingView Desktop のインストール(未導入の場合)
- `/Applications/TradingView.app` の有無を確認
- 無ければ: `brew install --cask tradingview`
- 既にあるならスキップ

■ ステップ2: リポジトリのクローンと依存インストール
- `~/tradingview-mcp-jackson` に
  https://github.com/LewisWJackson/tradingview-mcp-jackson.git をクローン
- そのディレクトリで `npm install` を実行

■ ステップ3: MCP設定への追記
- `~/.claude/.mcp.json` を読み込み(なければ新規作成)
- ユーザーのホームパスを `echo $HOME` で取得
- 既存の `mcpServers` を**絶対に上書きしない**。マージする形で以下を追加:

  {
    "mcpServers": {
      "tradingview": {
        "command": "node",
        "args": ["/Users/<ユーザー名>/tradingview-mcp-jackson/src/server.js"]
      }
    }
  }

- ユーザー名は `$HOME` から動的に取得して埋める

■ ステップ4: rules.json の準備
- `~/tradingview-mcp-jackson/rules.example.json` を `rules.json` にコピー
- ユーザーに「後でルールを編集してください」と一言

■ ステップ5(Mac版): TradingView Desktopを起動
- Macは Chrome CDP方式ではなく、**Desktop版を直接使える**ので話が早い
- `open -a TradingView` で起動
- ユーザーに「初回のみTradingViewにログインしてチャートを開いてください」と案内し、ログイン完了を待つ
- ログイン後、Claude Code側で `tv_launch` を呼べば接続される(Macなら使える)

→ 共通手順のステップ5(Claude Code再起動)へ

═══════════════════════════════════════
【共通手順】(Win/Mac共通)
═══════════════════════════════════════

■ ステップ5: Claude Code を再起動
- `.mcp.json` の変更を反映させるため、Claude Code を一度終了→再起動するよう案内
- 再起動後に「tv_health_check を実行して接続を確認してください」と伝える

■ ステップ6: 接続確認
- 再起動後、`tv_health_check` MCPツールを呼び出して接続を検証
- 失敗時のトラブルシュート:
  - **Windows**: Chromeがポート9222で起動しているか(`netstat -ano | findstr 9222`)、別Chromeが立ち上がっていないか、チャート画面が開かれているか
  - **Mac**: TradingView Desktopが起動しているか、ログインが完了しているか
- 成功したら「セットアップ完了です」と報告し、使い方サンプルとして「`morning_brief` を実行してみてください」と一言添える

═══════════════════════════════════════
【注意事項】
═══════════════════════════════════════
- ユーザーに不要な確認を取らずテキパキ進める(破壊的操作のみ確認)
- 既存の `.mcp.json` がある場合は必ずマージで、上書き厳禁
- 進捗は短く都度報告(「Node.jsインストール完了」「クローン完了」など)
- パッケージマネージャ(winget/brew/npm)の処理は数分かかるので、バックグラウンド実行 or タイムアウト延長を活用

ではOS判定からセットアップを開始してください。
````

### 補足:Win と Mac で接続方式が違う理由

| | Windows | Mac |
|---|---|---|
| TradingView | **Chrome Web版**(CDP経由) | **Desktop版**(`tv_launch`) |
| 理由 | Windows版Desktopアプリが入手困難なため | 公式Desktopアプリが普通に使える |
| 起動方法 | CDP有効Chromeを別プロファイルで起動 | `open -a TradingView` |

### 補足:Linux ユーザー向け

上記のワンショットプロンプトは Linux 非対応です。Linuxの方は、以下の手動セットアップ手順をご参照ください(Chrome CDP方式が使えるはずです)。

---

または、以下の手動セットアップ手順をご参照ください。

---

## 前提環境

- **TradingView Desktop アプリ**(リアルタイムデータには有料サブスクリプションが必要)
- **Node.js 18+**
- **Claude Code**(MCP ツール用)または任意のターミナル(CLI 用)
- **macOS / Windows / Linux**

---









## クイックスタート

### 1. クローンとインストール

```bash
git clone https://github.com/LewisWJackson/tradingview-mcp-jackson.git ~/tradingview-mcp-jackson
cd ~/tradingview-mcp-jackson
npm install
```

### 2. ルールの設定

```bash
cp rules.example.json rules.json
```

`rules.json` を開き、以下を埋めてください:
- **ウォッチリスト**(毎朝スキャンするシンボル)
- **バイアス基準**(自分にとって何が強気/弱気/中立か)
- **リスクルール**(毎セッション前に Claude にチェックさせたいルール)

### 3. CDP 有効化状態で TradingView を起動

TradingView はデバッグポートが有効化された状態で起動している必要があります。

**Mac:**
```bash
./scripts/launch_tv_debug_mac.sh
```

**Windows:**
```bash
scripts\launch_tv_debug.bat
```

**Linux:**
```bash
./scripts/launch_tv_debug_linux.sh
```

または、セットアップ後に MCP ツールを使う場合: *「tv_launch を使って TradingView をデバッグモードで起動して」*

### 4. Claude Code に追加

`~/.claude/.mcp.json` に以下を追加(既存のサーバーがあればマージ):

```json
{
  "mcpServers": {
    "tradingview": {
      "command": "node",
      "args": ["/Users/YOUR_USERNAME/tradingview-mcp-jackson/src/server.js"]
    }
  }
}
```

`YOUR_USERNAME` は実際のユーザー名に置き換えてください。Mac の場合は `echo $USER` で確認できます。

### 5. 接続確認

Claude Code を再起動し、以下のように依頼します: *「tv_health_check を使って TradingView の接続を確認して」*

### 6. はじめてのモーニングブリーフを実行

Claude に依頼: *「morning_brief を実行してセッションバイアスを教えて」*

または、ターミナルから:
```bash
npm link  # tv CLI をグローバルにインストール(初回のみ)
tv brief
```

---

## モーニングブリーフのワークフロー

これがツールキットを「日々の習慣」に変える機能です。

**毎セッションの前に:**

1. TradingView を開く(デバッグポート有効化状態で起動)
2. ターミナルで実行: `tv brief`(または Claude に *「morning_brief を実行して」*)
3. Claude がウォッチリスト内のすべてのシンボルをスキャンし、インジケーター値を読み取り、`rules.json` の基準を適用して、以下のように出力:

```
BTCUSD  | BIAS: Bearish  | KEY LEVEL: 94,200  | WATCH: RSI crossing 50 on 4H
ETHUSD  | BIAS: Neutral  | KEY LEVEL: 3,180   | WATCH: Ribbon direction on daily
SOLUSD  | BIAS: Bullish  | KEY LEVEL: 178.50  | WATCH: Hold above 20 EMA

Overall: Cautious session. BTC leading bearish, SOL the exception — watch for divergence.
```

4. 保存: *「このブリーフを保存して」*(`session_save` を使用)
5. 翌朝、比較: *「昨日のセッションを取得して」*(`session_get` を使用)

---

## このツールでできること

- **モーニングブリーフ** — ウォッチリストをスキャンし、インジケーターを読み取り、ルールを適用してセッションバイアスを出力
- **Pine Script 開発** — AI と協働でスクリプトの作成・注入・コンパイル・デバッグ
- **チャート操作** — シンボル変更、時間軸変更、日付ジャンプ、インジケーター追加/削除
- **ビジュアル分析** — インジケーター値、価格レベル、カスタムインジケーターの描画レベルを読み取り
- **チャートへの描画** — トレンドライン、水平レベル、矩形、テキスト
- **アラート管理** — 価格アラートの作成・一覧・削除
- **リプレイ練習** — 過去のバーを1本ずつ進め、エントリー/イグジットを P&L トラッキング付きで練習
- **スクリーンショット** — チャートの状態をキャプチャ
- **マルチペインレイアウト** — 2x2、3x1 などのグリッドにペインごとに異なるシンボルを表示
- **データストリーム** — ライブチャートから JSONL 形式で監視スクリプト用に出力
- **CLI アクセス** — すべてのツールが `tv` コマンドとしても利用可能、パイプフレンドリーな JSON 出力

---

## Claude はどのツールを使うかをどう判断するか

Claude はこのプロジェクトで作業するとき、`CLAUDE.md` を自動的に読み込みます。そこに完全な意思決定ツリーが書かれています。

| あなたが言うこと | Claude が使うツール |
|------------|---------------|
| 「モーニングブリーフを実行して」 | `morning_brief` → ルール適用 → `session_save` |
| 「昨日のバイアスはどうだった?」 | `session_get` |
| 「今チャートに何が出てる?」 | `chart_get_state` → `data_get_study_values` → `quote_get` |
| 「フル分析をして」 | `quote_get` → `data_get_study_values` → `data_get_pine_lines` → `data_get_pine_labels` → `capture_screenshot` |
| 「BTCUSDの日足に切り替えて」 | `chart_set_symbol` → `chart_set_timeframe` |
| 「〜のPine Scriptを書いて」 | `pine_set_source` → `pine_smart_compile` → `pine_get_errors` |
| 「3月1日からリプレイを始めて」 | `replay_start` → `replay_step` → `replay_trade` |
| 「4分割チャートにして」 | `pane_set_layout` → `pane_set_symbol` |
| 「94200にラインを引いて」 | `draw_shape`(horizontal_line) |

---

## ツールリファレンス(MCP ツール 81 種)

### モーニングブリーフ(本フォークの新機能)

| ツール | 内容 |
|------|------|
| `morning_brief` | ウォッチリストをスキャンし、インジケーターを読み取り、セッションバイアス用の構造化データを返す。`rules.json` を自動で読み込む |
| `session_save` | 生成したブリーフを `~/.tradingview-mcp/sessions/YYYY-MM-DD.json` に保存 |
| `session_get` | 今日のブリーフを取得(未保存なら昨日のもの) |

### チャート読み取り

| ツール | 用途 | 出力サイズ |
|------|------|-------------|
| `chart_get_state` | 最初に呼ぶ — シンボル、時間軸、全インジケーター名+IDを取得 | 約500B |
| `data_get_study_values` | 全インジケーターの現在値(RSI, MACD, BB, EMA など)を取得 | 約500B |
| `quote_get` | 最新価格、OHLC、出来高を取得 | 約200B |
| `data_get_ohlcv` | 価格バーを取得。**`summary: true` 推奨**(コンパクトな統計値) | 500B(summary)/ 8KB(100バー) |

### カスタムインジケーターのデータ(Pine 描画)

任意の表示中の Pine インジケーターから `line.new()`、`label.new()`、`table.new()`、`box.new()` の出力を読み取ります。

| ツール | 用途 |
|------|------|
| `data_get_pine_lines` | 水平価格レベル(サポート/レジスタンス、セッションレベル) |
| `data_get_pine_labels` | テキスト注釈+価格(例: "PDH 24550"、"Bias Long") |
| `data_get_pine_tables` | データテーブル(セッション統計、分析ダッシュボード) |
| `data_get_pine_boxes` | 価格ゾーン({high, low} ペア) |

**特定のインジケーターを対象にするには必ず `study_filter` を使用**: 例 `study_filter: "MyIndicator"`。

### チャート制御

| ツール | 内容 |
|------|------|
| `chart_set_symbol` | ティッカーを変更(BTCUSD, AAPL, ES1!, NYMEX:CL1!) |
| `chart_set_timeframe` | 時間軸を変更(1, 5, 15, 60, D, W, M) |
| `chart_set_type` | チャート種別を変更(Candles, HeikinAshi, Line, Area, Renko) |
| `chart_manage_indicator` | インジケーター追加/削除。**フルネームを使うこと**: "RSI" ではなく "Relative Strength Index" |
| `chart_scroll_to_date` | 日付ジャンプ(ISO形式: "2025-01-15") |
| `indicator_set_inputs` / `indicator_toggle_visibility` | インジケーター設定変更、表示/非表示 |

### Pine Script 開発

| ツール | ステップ |
|------|------|
| `pine_set_source` | 1. エディタにコードを注入 |
| `pine_smart_compile` | 2. 自動検出+エラーチェック付きでコンパイル |
| `pine_get_errors` | 3. コンパイルエラーがあれば読み取り |
| `pine_get_console` | 4. log.info() の出力を読み取り |
| `pine_save` | 5. TradingView クラウドに保存 |
| `pine_analyze` | オフライン静的解析(チャート不要) |
| `pine_check` | サーバーサイドのコンパイルチェック(チャート不要) |

### リプレイモード

| ツール | ステップ |
|------|------|
| `replay_start` | 指定日付でリプレイ開始 |
| `replay_step` | 1バー進める |
| `replay_autoplay` | 自動再生(速度をミリ秒で指定) |
| `replay_trade` | ポジションの売買・決済 |
| `replay_status` | ポジション、P&L、日付を確認 |
| `replay_stop` | リアルタイムに戻る |

### マルチペイン、アラート、描画、UI

| ツール | 内容 |
|------|------|
| `pane_set_layout` | グリッド変更: `s`, `2h`, `2v`, `2x2`, `4`, `6`, `8` |
| `pane_set_symbol` | 任意ペインのシンボルを設定 |
| `draw_shape` | 描画: horizontal_line, trend_line, rectangle, text |
| `alert_create` / `alert_list` / `alert_delete` | 価格アラート管理 |
| `batch_run` | 複数シンボル/時間軸でアクションを横断実行 |
| `watchlist_get` / `watchlist_add` | ウォッチリストの読み取り/変更 |
| `capture_screenshot` | スクリーンショット(対象: full, chart, strategy_tester) |
| `tv_launch` / `tv_health_check` | TradingView 起動と接続確認 |

---

## CLI コマンド

```bash
tv brief                           # モーニングブリーフを実行
tv session get                     # 今日の保存済みブリーフを取得
tv session save --brief "..."      # ブリーフを保存

tv status                          # 接続確認
tv quote                           # 現在価格
tv symbol BTCUSD                   # シンボル変更
tv ohlcv --summary                 # 価格サマリー
tv screenshot -r chart             # チャートをキャプチャ
tv pine compile                    # Pine Script コンパイル
tv pane layout 2x2                 # 4チャートグリッド
tv stream quote | jq '.close'      # 価格ティックを監視
```

全コマンド一覧: `tv --help`

---

## トラブルシューティング

| 問題 | 解決方法 |
|------|----------|
| `cdp_connected: false` | TradingView が `--remote-debugging-port=9222` 付きで起動していない。起動スクリプトを使用してください |
| `ECONNREFUSED` | TradingView が起動していないか、ポート 9222 がブロックされている |
| Claude Code に MCP サーバーが表示されない | `~/.claude/.mcp.json` の構文を確認し、Claude Code を再起動 |
| `tv` コマンドが見つからない | プロジェクトディレクトリで `npm link` を実行 |
| `morning_brief` — "No rules.json found" | `cp rules.example.json rules.json` を実行して中身を埋める |
| `morning_brief` — ウォッチリストが空 | `rules.json` の `watchlist` 配列にシンボルを追加 |
| ツールが古いデータを返す | TradingView がまだロード中 — 数秒待つ |
| Pine Editor 系ツールが失敗する | Pine Editor パネルを先に開く: `ui_open_panel pine-editor open` |

---

## アーキテクチャ

```
Claude Code  ←→  MCP Server (stdio)  ←→  CDP (port 9222)  ←→  TradingView Desktop (Electron)
```

- **オリジナル 78 ツール** + **モーニングブリーフ 3 ツール** = 計 **81 個の MCP ツール**
- **トランスポート**: stdio 経由の MCP + CLI(`tv` コマンド)
- **接続**: localhost:9222 上の Chrome DevTools Protocol
- **外部ネットワーク通信なし** — すべてローカルで完結
- **追加の依存関係ゼロ** — オリジナル以上の依存追加なし

---

## クレジット

本フォークは [@tradesdontlie](https://github.com/tradesdontlie) 氏による [tradingview-mcp](https://github.com/tradesdontlie/tradingview-mcp) を基盤としています。オリジナルツールこそが土台です — 元リポジトリにスターを付けてあげてください。

また、本フォークのベースとなった改良版 [tradingview-mcp-jackson](https://github.com/LewisWJackson/tradingview-mcp-jackson) を公開してくださっている [@LewisWJackson](https://github.com/LewisWJackson) 氏にも感謝します。

---

## 免責事項

本プロジェクトは、**個人・教育・研究目的のみで提供されます**。

本ツールは Chrome DevTools Protocol(CDP)を使用しています。これはすべての Chromium 系アプリケーションに標準搭載されているデバッグインターフェースです。TradingView の独自プロトコルをリバースエンジニアリングしたり、TradingView のサーバーに接続したり、アクセス制御を回避することはありません。デバッグポートは、Chromium 標準のコマンドラインフラグを介して、ユーザーが明示的に有効化する必要があります。

本ソフトウェアを使用することにより、以下に同意したものとみなされます:

1. [TradingView 利用規約](https://www.tradingview.com/policies/)および適用されるすべての法律に準拠する責任は、すべてユーザーにあります。
2. 本ツールは、いつでも変更される可能性のある TradingView の非公開内部 API にアクセスします。
3. 本ツールは、TradingView の市場データの再配布、再販、商業的利用には使用してはなりません。
4. 著作者は、アカウントの BAN、停止、その他の結果について一切の責任を負いません。

**自己責任でご利用ください。**

## ライセンス

MIT — [LICENSE](LICENSE) を参照してください。本ライセンスはソースコードのみに適用され、TradingView のソフトウェア、データ、商標には適用されません。
