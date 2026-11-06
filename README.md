# アデランス AI音声スタジオ

開発者用`ai_voice_studio.ipynb`をGoogle Colabで実行する社内配布プロジェクトです。Windowsで音声処理本体を直接実行するものではありません。

- 開発者用: `ai_voice_studio.ipynb`
- 配布用: `アデランス_AI音声スタジオ_v1_0.ipynb`（`v`は小文字）
- GitHub: `https://github.com/ad-developer-robous/ai-voice-studio`

配布ノートブックは固定のブートストラップです。最初のセルがGitHubの`main`にある開発者ノートブックを取得して実行するため、開発者がpushした後の再実行で修正が反映されます。

## 社内ユーザー

初回はGitとPowerShellが使えるWindows端末で、次を実行します。

```powershell
git clone https://github.com/ad-developer-robous/ai-voice-studio.git "$HOME\AI-Voice-Studio"
cd "$HOME\AI-Voice-Studio"
.\Start-AIVoiceStudio.bat
```

以後はcloneしたフォルダーで`.\Start-AIVoiceStudio.bat`を実行してください。スクリプトは安全な更新を試みてからGoogle Colabの配布ノートブックをブラウザーで開きます。ColabでGPUランタイム（推奨T4）を接続し、最初のコードセルを実行してください。直接起動する場合は`.\scripts\Start-AIVoiceStudio.ps1`を使います。

このランチャーが開くURLは次の形式です。ローカルノートブックをColabへアップロードするのではありません。

`https://colab.research.google.com/github/ad-developer-robous/ai-voice-studio/blob/main/アデランス_AI音声スタジオ_v1_0.ipynb`

ローカル変更がある場合、fetch失敗、またはfast-forwardできない場合、更新ヘルパーは警告して現在のcloneを変更せず起動を続けます。stash、reset、強制checkout、上書きは行いません。最新状態を使う場合は別のcloneを作成してください。Colabで取得できない場合はネットワークを確認して配布ノートブックを開き直してください。

## 開発者の公開手順

1. `ai_voice_studio.ipynb`だけを編集し、必要ならColabで確認します。
2. `scripts\Sync-DistributionNotebook.ps1`を実行します。これは開発者ノートブックの実行結果と実行回数を削除します。
3. 両ノートブックとスクリプトの差分を確認します。
4. 必要なファイルをcommitして`main`へpushします。自動commit・pushはしません。

```powershell
.\scripts\Sync-DistributionNotebook.ps1
git diff --check
git status --short --untracked-files=all
```

配布ノートブックは固定ブートストラップなので、開発者ノートブックを配布ファイルへコピーしません。同期スクリプトは配布ファイルがブートストラップでない場合に停止します。

## セキュリティと信頼範囲

最新修正を自動反映するため、配布ノートブックは固定コミットではなくGitHub`main`のRaw URLからコードを取得して実行します。これは意図した信頼モデルです。`main`へのpush内容とGitHubアカウントを配布対象コードとしてレビューしてください。固定コミット運用へ変更する場合は、URLと公開手順を同時に変更します。

アプリは一時的なCloudflare Tunnelを作り、ランダムなURLトークンでアクセスを制限します。トンネルは公開インターネットから到達可能で、URLを知る人がアクセスを試せます。URL・トークンを共有・保存・Gitへ追加せず、組織が承認した音声データだけを扱ってください。現在のアプリは接続元を限定するCORS設定ではないため、この公開範囲を受け入れられない場合は公開前に認証・CORS方針を見直してください。

## Colab依存関係

`fastapi`、`uvicorn`、`jinja2`、`python-multipart`、`https://github.com/Aratako/Irodori-TTS.git`とその`requirements.txt`、Hugging Faceの`Aratako/Irodori-TTS-v4.1-Small`、Linux用Cloudflare Tunnelを使用します。直接インストールするパッケージは`requirements-colab.txt`にも記載しています。これはWindows用ではありません。Google Colab、Linux、外部ネットワーク、GPUが必要です。

## 生成物と検証

トンネルURL・トークン、音声、ログ、モデルキャッシュ、生成された`app.py`・`templates`、APIキー、個人情報はcommitしないでください。公開前に差分を確認してください。

```powershell
python -c "import json, pathlib; [json.loads(p.read_text(encoding='utf-8')) for p in pathlib.Path('.').glob('*.ipynb')]"
[System.Management.Automation.Language.Parser]::ParseFile((Resolve-Path '.\scripts\Update-AIVoiceStudio.ps1'), [ref]$null, [ref]$null)
[System.Management.Automation.Language.Parser]::ParseFile((Resolve-Path '.\scripts\Sync-DistributionNotebook.ps1'), [ref]$null, [ref]$null)
[System.Management.Automation.Language.Parser]::ParseFile((Resolve-Path '.\scripts\Start-AIVoiceStudio.ps1'), [ref]$null, [ref]$null)
git diff --check
```

Google Colab/GPU実行、push後のRaw配信、Windowsのダブルクリック表示は未確認事項です。公開前に実環境で確認してください。
