# cpp-core-doc-trans

C++ Core Guidelines (https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines 。元はリポジトリ [isocpp/CppCoreGuidelines](https://github.com/isocpp/CppCoreGuidelines) の Markdown) を**開いたときに** 取得して、日本語に翻訳して表示するアプリです (事前に翻訳したものはありません)。メニュー (☰) の「原文」で、英語に戻せます。PC (Linux, Windows, macOS) と Android で動きます。

- **Tauri 2** (Rust) + **React** (TypeScript)。
- 翻訳のサービスは設定で選びます。**Google Cloud Translation** (Basic, v2)、**Gemini API**、**ML Kit** (Android のみ。端末の中で翻訳。オフライン)、**Google 翻訳 (無料)** (キー不要。非公式の入口)。API キーは使う人が自分のものをアプリの設定に入れます (キーはアプリに含めません)。
- 翻訳の結果は端末にキャッシュします (同じ文章をもう一度翻訳しないため。翻訳の費用を減らします)。

## 仕組み

```
章を開く ─> Rust: ガイドラインのページ (全体で 1 つ) を取得 (isocpp.github.io だけ。1 回だけ取得して、覚えておく)
         ─> React: 本文を取り出し、章 (h1) ごとに分ける。開いた章を、段落ごとに分ける
         ─> Rust: 段落 (HTML の断片) を翻訳 (コードには「翻訳しない」印を付ける) ─> キャッシュ
            (Google / Gemini: HTML のまま。ML Kit と無料の Google 翻訳: タグを目印 `[[0]]` に置き換えて翻訳し、戻す)
         ─> React: 翻訳を差し込んで表示 (リンクは、アプリの中で開く)
```

- ガイドラインは、全体が 1 つの大きなページ (約 1.3 MB) です。そのまま翻訳すると、表示が重くなるので、章 (「F: Functions」「C: Classes and class hierarchies」など) ごとに分けて、別のページとして開きます (共通のコードの `sections`)。分けたページの URL は、`…/CppCoreGuidelines?section=<章の id>` です。
- 規則へのリンク (`#Rf-…` など) は、その規則がある章を開いて、その規則へ移ります。章の最後に、次の章へのリンクがあります。
- コードのブロックと、文中のコード (`code` など) は翻訳しません。
- 翻訳の結果で、コードの印やリンクの数が変わった段落は英語のまま残します (画面に件数を出します)。
- 表示する HTML は取得したものも、翻訳されたものも、必ず `DOMPurify` を通します (スクリプトなどを除きます)。
- アプリの中で開くのは、ガイドラインのページ (`https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines`) だけです。ほかのリンク (GitHub、論文、動画など) はブラウザで開きます。

## 使い方

1. アプリを起動します。既定は無料の Google 翻訳で、キーなしで、すぐに使えます。ほかのサービスを使うときは、左上の ☰ のメニューの ⚙ から、翻訳のサービスを選んで、API キーを入れます (ML Kit はキーが要りません)。
2. 最初の章 (表紙と目次) が開きます。リンクを押すと、その章を開いて、翻訳して表示します。
3. 左上の ☰ で**メニュー (ナビゲーションドロワー)** を開きます。メニュー (目次) は、章、節、規則の見出しから作ります (全体が最初から、そろいます)。日本語の表示のときは開いたときに、項目の名前も翻訳します (キャッシュされます)。項目を押すと、その章の、その規則へ移ります。
4. **読んだ場所**: 右端にスクロールバーを出します。つまみを押して動かすと、つまみの横に今の章の、読み進めた場所 / 章全体 (「3 / 12 画面 (25%)」) が出ます。最後に開いた章と、章ごとの読んだ位置を端末に覚えて、次の起動で戻します。
5. **トップとしおり**: ツールバーの ⌂ で、最初の章に戻ります。☆ で今の章にしおりを付けて、メニューの「しおり」から開きます (外すのは ✕)。
6. メニューの「原文」で英語、もう一度押すと日本語にします。

## 共通のコード (サブモジュール)

画面、HTML の取り出し、翻訳、取得、キャッシュ、ML Kit のプラグインは**共通のリポジトリ** [doc-translator-core](https://github.com/sluchin/doc-translator-core) にあります (サブモジュール `shared/`)。このリポジトリには C++ Core Guidelines だけの設定 (`src/site.ts` と `src-tauri/src/lib.rs` の `SITE`) と、アイコンと、アプリの設定だけがあります。姉妹アプリは [zephyr-doc-trans](https://github.com/sluchin/zephyr-doc-trans) (Zephyr のドキュメント)、[kotlin-doc-trans](https://github.com/sluchin/kotlin-doc-trans) (Kotlin のドキュメント) などです。

```bash
git clone --recurse-submodules git@github.com:sluchin/cpp-core-doc-trans.git   # 取り込み済みなら: git submodule update --init
```

## 開発 (Docker)

ビルドと検査は全て Docker の中で行います (ホストに Node.js や Rust は要りません)。`UID` と `GID` をシェルでエクスポートしておきます。

```bash
export UID GID=$(id -g)
docker compose run --rm install          # 依存関係 (npm ci)
docker compose run --rm test             # サイトの設定のテストと cargo test (共通のコードのテストは shared で)
docker compose run --rm format           # 整形の確認 (format-fix で整形)
docker compose run --rm analyze          # tsc, ESLint, Clippy
docker compose run --rm build-desktop    # PC (Linux) のデバッグ版
docker compose run --rm build-android    # Android のデバッグ版 APK
```

- Android の APK: `src-tauri/gen/android/app/build/outputs/apk/universal/debug/app-universal-debug.apk` (arm64)。
- Windows と macOS はそれぞれの OS で `npm ci && npx tauri build` を実行します (Rust, Node.js, 各 OS の Tauri の前提が要ります。https://tauri.app/start/prerequisites/)。
- 画面の確認 (`npm run tauri:dev`) はホストで行います。

## リリース

`v0.1.0` のようなタグを push すると、GitHub Actions の `release.yml` が PC (Windows、macOS、Linux のインストーラ。Windows は exe 単体の zip も) と Android の APK を作って、GitHub Releases の**ドラフト**に載せます (中身を確かめて、手で公開します)。APK はデバッグ鍵で署名したデバッグ版です。タグを打たずに試すときは Actions の Release を手動で実行します (成果物は Artifacts に保存されます)。

## ライセンス

GPL-3.0-or-later。C++ Core Guidelines は Standard C++ Foundation と貢献者のものです (MIT 形式のライセンス。https://github.com/isocpp/CppCoreGuidelines/blob/master/LICENSE)。このリポジトリにはドキュメントの文章を含めません (実行のときに、取得します)。
