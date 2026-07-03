# Unity と Claude Code をくっつける手順（Windows・初心者版）

このドキュメントは「Unityエンジンを入れただけ」の状態から、
Claude Code で開発できるようにするまでを、**上から順番に**なぞるためのものです。

やることは大きく分けて2段階です。

- **STEP A**：Claude Code に C# スクリプトを書いてもらう（かんたん・まずここまで）
- **STEP B**：Claude が Unity エディタ自体を操作する（Unity MCP・ちょっと本格的）

まずは STEP A まで進めば「連携できた」と言えます。STEP B はそのあとで大丈夫です。

> 💡 大事なこと：これは全部 **あなたの Windows PC** で行う作業です。
> クラウド上の Claude Code ではなく、自分のパソコンで進めます。

---

## 事前に用意するもの

- Windows 10 または 11 のパソコン
- Unity（インストール済み）… ただし「Unity Hub」も必要です（後述）
- Anthropic のアカウント（Claude Code のログインに使う）

---

## STEP 0. Unity Hub と Unity プロジェクトを用意する

「エンジンを入れただけ」だと、まだ**プロジェクト（作品の入れ物）**がありません。まず作ります。

1. **Unity Hub** を開く（無ければ https://unity.com/download から入れる）。
   - Unity Hub は「プロジェクトやバージョンを管理するアプリ」です。
2. 左メニューの **「Installs（インストール）」** で、Unity 本体が入っているか確認。
   - 入っていなければ「Install Editor」から **Unity 6（または 2022 LTS）** を入れる。
   - ※このガイドは Unity 2021.3 〜 6.x に対応しています。
3. 左メニューの **「Projects（プロジェクト）」** → **「New project」** をクリック。
4. テンプレートは **「3D (Built-In Render Pipeline)」** を選ぶ（迷ったらこれ）。
5. プロジェクト名（例：`MyFirstGame`）と保存場所を決めて **「Create project」**。
6. しばらく待つと Unity エディタが開きます。ここまでで入れ物が完成。

📌 **保存場所のフォルダのパスを覚えておいてください。** 例：
`C:\Users\あなたの名前\MyFirstGame`
このフォルダが、あとで Claude Code を起動する場所になります。

---

## STEP A. Claude Code を入れて、コードを書いてもらう

### A-1. Claude Code をインストールする

Windows では PowerShell で1行流すのが一番かんたんです。

1. スタートメニューで「PowerShell」と検索して開く。
2. 次を貼り付けて Enter：

   ```powershell
   irm https://claude.ai/install.ps1 | iex
   ```

3. 終わったら、**PowerShell を一度閉じて開き直す**。
4. 動作確認：

   ```powershell
   claude --version
   ```

   バージョン番号が出ればOK。

> うまくいかない時は、公式の案内も参考に：https://docs.claude.com/en/docs/claude-code/setup

### A-2. Unity プロジェクトのフォルダで Claude Code を起動する

1. PowerShell で、STEP 0 で覚えたフォルダに移動：

   ```powershell
   cd "C:\Users\あなたの名前\MyFirstGame"
   ```

2. Claude Code を起動：

   ```powershell
   claude
   ```

3. 初回はブラウザが開いてログインを求められます。Anthropic アカウントでログイン。
4. 起動したら、試しにこう打ってみてください（日本語でOK）：

   ```
   プレイヤーを矢印キーで動かす C# スクリプトを作って、
   Assets フォルダに置いて
   ```

   Claude が `.cs` ファイルを作ってくれます。
   Unity エディタに戻ると、そのスクリプトが Assets に現れます。

🎉 **ここまでで「①コードを書く連携」は完了です。** これだけでもかなり使えます。

---

## STEP B. Unity エディタ自体を Claude に操作させる（Unity MCP）

ここからは、Claude が Unity の中で
「Cube を置く」「シーンを編集する」「コンパイルエラーを読む」などを
**直接できる**ようにする連携です。少し準備が増えます。

使うのは定番の **CoplayDev / unity-mcp** です。

### B-0. 【重要】先に Git を入れておく（つまずきポイント①）

Unity が git URL からパッケージを取ってくるには、パソコンに **Git** が別で入っている必要があります。無いと B-2 が静かに失敗します。

1. PowerShell で確認：

   ```powershell
   git --version
   ```

   バージョンが出れば入っています → B-1 へ。
2. 「認識されない」等のエラーなら、https://git-scm.com/download/win から入れる。
   インストール中の選択肢は**全部そのまま Next でOK**。
3. **入れたら、Unity を一度完全に閉じて開き直す**（超重要。開きっぱなしだと Git を認識しません）。

### B-1. Python（3.10+）と uv を入れる（つまずきポイント②）

Unity MCP は内部で Python を使います。**Python 本体と uv の両方**が必要です。

1. **uv を入れる**（PowerShell）：

   ```powershell
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   ```

2. **Python 本体を入れる**：Microsoft Store で「Python 3.12」を検索してインストールが一番かんたん。
   （python.org から入れる場合は、最初の画面で **「Add python.exe to PATH」に必ずチェック**）
3. PowerShell を開き直して両方確認：

   ```powershell
   uv --version
   python --version
   ```

   両方バージョンが出ればOK。

### B-2. Unity に MCP パッケージを入れる

1. Unity エディタで、上メニュー **Window → Package Manager** を開く。
2. 左上の **「＋」ボタン → 「Install package from git URL...」** を選ぶ
   （Unity のバージョンにより「Add」表記の場合あり）。
3. 次のURLを貼って **Install**：

   ```
   https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#v10.0.0
   ```

   （`#v10.0.0` は安定版。最新を試したいなら `#main` でもOK）
4. インストールが終わると、メニューに **Window → MCP for Unity** が増えます。
   - 失敗する場合はほぼ B-0 の Git 未インストールが原因。Console（Window → General → Console）の赤いエラーを確認。

### B-3. 依存関係チェック（Local Setup Window）

1. Unity で **Window → MCP for Unity → Local Setup Window** を開く。
2. **Python** と **UV** が**両方とも緑の●**になっているか確認。
   - 赤があれば B-1 をやり直し、`Refresh` を押して緑にする。
3. 「All requirements met!」が出たら次へ。**Claude Code にチェックを入れて `Configure Selected`** を押す。

### B-4. 【最重要】Unity 側のサーバーを起動する（つまずきポイント③）

ここが一番の落とし穴。**「MCP for Unity」のメニューには3つの項目があり、サーバー起動ボタンがあるのは "Toggle MCP Window" の中だけ**です。
（Local Setup Window / Edit EditorPrefs には起動ボタンはありません）

1. Unity で **Window → MCP for Unity → `Toggle MCP Window`** を開く
   （ショートカット：**Ctrl + Shift + M**）。
2. **`Server`** セクションを探す。中に：
   - `Transport:`（通信方式。既定は HTTP）
   - `HTTP URL:`（既定 `http://localhost:8080`）
   - **`Local Server:` の横の `Start Server` ボタン** ← これを押す。
3. `Start Server` を押すと**黒いターミナル窓**が開いてサーバーが起動します。**その窓は閉じない**。
4. その下の `Start`（connection-toggle）も押し、状態表示が **Connected** になればOK。
   - ポート 8080 が他ソフト（Tailscale 等）とぶつかる場合は、URL を `http://localhost:8090` などに変えて Start し直す。

**確認**：PowerShell で `curl http://localhost:8080/health` を打って応答が返ればサーバー稼働中。

### B-5. 接続を確認する

1. **Unity エディタ・プロジェクトは開いたまま**、**サーバーも起動したまま**にしておく。
2. **必ずプロジェクトフォルダの中で** `claude` を起動する
   （UnityMCP の設定はそのプロジェクト専用。別フォルダで起動すると出てきません）：

   ```powershell
   cd "C:\Users\ユーザー名\...\プロジェクト名"
   claude
   ```

3. Claude の中で `/mcp` を打つ。
   `UnityMCP · ✓ connected · (数十) tools` と出れば**成功！**
4. 仕上げに、こう頼んでみる：

   ```
   原点に赤いキューブを作って、Rigidbody をつけて
   ```

   Unity のシーンに Cube が現れたら大成功！🎉

---

## つまずいたときのチェックリスト

- **`claude` が見つからない** → PowerShell を開き直す。それでもダメなら再インストール。
- **B-2 のパッケージ導入が失敗する** → Git が入っていない（B-0）。Git を入れて Unity を再起動。
- **`/mcp` に UnityMCP が出てこない** → Claude を**プロジェクトフォルダの中で**起動しているか確認（B-5）。別フォルダだと出ません。
- **`UnityMCP · ✗ failed`／`Failed to connect`／port 8080 に何もいない** → Unity 側のサーバー未起動。`Toggle MCP Window`（Ctrl+Shift+M）→ `Start Server`（B-4）。
- **`curl http://localhost:8080/health` が繋がらない** → 同上。サーバーを起動する。
- **`uv` や `python` が見つからない** → PowerShell を開き直す。B-1 をやり直す。
- **Unity のバージョンが古い** → Unity Hub から 2022 LTS 以上を入れる。
- **英語で困る** → Claude に「日本語で説明して」と頼めばOK。エラー文を貼れば直し方も教えてくれます。

> 💡 一度つながっても、**PCを再起動したり Unity を開き直すとサーバーは止まります**。
> その時は毎回 B-4（`Toggle MCP Window` → `Start Server`）でサーバーを起動してください。
> Advanced Settings の **「Auto-Start Server on Editor Load」** をオンにしておくと、Unity 起動時に自動で立ち上がって楽です。

---

## 参考リンク（出典）

- [CoplayDev/unity-mcp（GitHub）](https://github.com/CoplayDev/unity-mcp)
- [Unity 公式ブログ：MCP の始め方](https://unity.com/blog/unity-ai-mcp-how-to-get-started)
- [Coplay ドキュメント：Claude Code との連携](https://docs.coplay.dev/coplay-mcp/claude-code-guide)
- [Claude Code セットアップ（公式）](https://docs.claude.com/en/docs/claude-code/setup)
