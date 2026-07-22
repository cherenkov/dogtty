# dogtty

[English](README.md) | 日本語

Ghosttyターミナルの右下にAI生成の柴犬画像を表示し、3分ごとに自動で差し替える。

![dogttyのスクリーンショット](screenshot.png)

ADHD対策として、作業中のターミナルにちょっとした癒やしと視覚的な変化を置くためのもの。

## 仕組み

```
launchd (180秒ごと)
  → dogtty
      1. Pollinations.ai で柴犬画像をAI生成（無料・APIキー不要）
      2. 300pxのPNGに変換して ~/.local/share/dogtty/dogtty.png を上書き
      3. Ghostty に SIGUSR2 を送って設定リロード → 右下の画像が替わる
```

ポーズ6種 × 画風6種（水彩・パステル・フラット・色鉛筆・DSLR写真・フィルム写真）× ランダムシードで毎回違う柴犬が出る。

## ファイル構成

ツールの実体は `dogtty` スクリプト1つ: 生成に加えて `install` / `config` / `uninstall` サブコマンドを持つ。自己完結（launchdのplistも自前で生成）なので、このファイル1つだけダウンロードしても使える。

## セットアップ

```bash
./dogtty install
```

犬・テイスト・間隔を対話形式で聞かれる（Enterで表示中の値のまま）。非対話環境では既定値（柴犬・おまかせ・180秒）で無言インストールされる。

犬の選択肢は柴犬（デフォルト）・シーズー・コーギー・トイプードル・ゴールデンレトリバー・自由入力（犬以外でも可）。テイストの選択肢はおまかせ（デフォルト）・イラスト風のみ・写真風のみ・自由入力（例: pixel art）。選択は `~/.config/dogtty/` 配下に保存される。

実行後、Ghostty設定（`~/.config/ghostty/config`）に以下を追記して `Cmd+Shift+,` でリロードする:

```ini
background-image = ~/.local/share/dogtty/dogtty.png
background-image-position = bottom-right
background-image-fit = none
background-image-opacity = 0.3
```

## 使い方

```bash
dogtty                                  # おまかせ柴犬を今すぐ生成
dogtty "corgi puppy, pixel art"         # プロンプト指定で生成
dogtty config                           # 犬・テイスト・間隔を対話形式で変更
dogtty install                          # （再）インストール
dogtty uninstall                        # 全部削除
```

（`install`・`uninstall`・`config` は予約語のため、生成プロンプトとしては使えない。）

## 運用

```bash
# 自動実行を止める
launchctl bootout gui/$(id -u)/local.dogtty

# 自動実行を再開する
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/local.dogtty.plist

# ログを見る（1MBを超えると自動で切り詰める）
cat ~/Library/Logs/dogtty.log
```

インストール後に犬・テイスト・間隔を変えるときは `dogtty config`。各質問には現在の設定が表示され、Enterでそのまま維持できるので、変えたい項目だけ入力すればいい。

Ghosttyが起動していないときは生成せずスキップする（無駄なAPI呼び出しをしない）。

## アンインストール

```bash
dogtty uninstall
```

Ghostty設定に追記した `background-image` の4行だけは手動で削除する。

## 注意

- **`~/.local/bin/dogtty` はこのリポジトリへのシンボリックリンクにしないこと。** `~/Documents` 配下はmacOSのプライバシー保護（TCC）でlaunchdから読めず「Operation not permitted」になる。実体コピーが必要で、`dogtty install` がまさにそれをやる（スクリプトを編集したら再実行して反映）
- **GhosttyにはSIGUSR2以外のシグナルを送らないこと。** 未ハンドルのシグナルはプロセスごと落ちる。SIGUSR2によるconfigリロードはGhostty 1.2+の公式サポート機能
- Pollinations.aiの匿名利用にはレート制限がある。連続実行すると429が返るため、スクリプトは15秒待ち×最大3回の自動リトライをする
- 生成画像はAI生成のため著作権上の懸念なく使える

## 動作環境

- macOS（`sips`・`launchd`・`python3` はOS標準のものを使用）
- Ghostty 1.2以上（`background-image` とSIGUSR2リロードが必要。1.3.1で動作確認）

## License

MIT
