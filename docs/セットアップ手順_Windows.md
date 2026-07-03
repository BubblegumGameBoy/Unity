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

### B-1. Python（uv）を入れる

Unity MCP は内部で Python を使うので、`uv` というツールを入れます。

1. PowerShell で：

   ```powershell
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   ```

2. PowerShell を開き直して確認：

   ```powershell
   uv --version
   ```

   バージョンが出ればOK。

### B-2. Unity に MCP パッケージを入れる

1. Unity エディタで、上メニュー **Window → Package Manager** を開く。
2. 左上の **「＋」ボタン → 「Add package from git URL...」** を選ぶ。
3. 次のURLを貼って **Add**：

   ```
   https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#v10.0.0
   ```

   （`#v10.0.0` は安定版。最新を試したいなら `#main` でもOK）
4. インストールが終わると、メニューに **Window → MCP for Unity** が増えます。

### B-3. Unity と Claude Code をつなぐ

1. Unity エディタで **Window → MCP for Unity** を開く。
2. **「Configure All Detected Clients」**（検出した全クライアントを設定）をクリック。
   - これで Claude Code 用の接続設定が自動で書き込まれます。
3. うまく検出されない場合は、PowerShell から手動で追加：

   ```powershell
   claude mcp add UnityMCP -- uvx --python ">=3.11" unity-mcp-server@latest
   ```

   （B-2 の Unity 側で表示される正式なコマンドがあれば、そちらを優先してください）

### B-4. 接続を確認する

1. **Unity エディタは開いたまま**にしておく（閉じるとつながりません）。
2. そのプロジェクトフォルダで `claude` を起動。
3. Claude の中で次を打つ：

   ```
   /mcp
   ```

   `UnityMCP`（または coplay 系）が **connected** と出れば成功。
4. 仕上げに、こう頼んでみる：

   ```
   原点に Cube を作って、Rigidbody をつけて
   ```

   Unity のシーンに Cube が現れたら大成功！🎉

---

## つまずいたときのチェックリスト

- **`claude` が見つからない** → PowerShell を開き直す。それでもダメなら再インストール。
- **`/mcp` に何も出てこない** → Unity エディタが開いているか確認。B-3 をやり直す。
- **`uv` が見つからない** → PowerShell を開き直す。B-1 をやり直す。
- **Unity のバージョンが古い** → Unity Hub から 2022 LTS 以上を入れる。
- **英語で困る** → Claude に「日本語で説明して」と頼めばOK。エラー文を貼れば直し方も教えてくれます。

---

## 参考リンク（出典）

- [CoplayDev/unity-mcp（GitHub）](https://github.com/CoplayDev/unity-mcp)
- [Unity 公式ブログ：MCP の始め方](https://unity.com/blog/unity-ai-mcp-how-to-get-started)
- [Coplay ドキュメント：Claude Code との連携](https://docs.coplay.dev/coplay-mcp/claude-code-guide)
- [Claude Code セットアップ（公式）](https://docs.claude.com/en/docs/claude-code/setup)
