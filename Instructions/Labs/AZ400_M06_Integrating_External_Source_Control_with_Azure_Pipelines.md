﻿---
lab:
    title: 'ラボ: 外部ソース管理を Azure Pipelines と統合する'
    module: 'モジュール 6: Azure Pipelines を使用した継続的インテグレーションの実装'
---

# ラボ: 外部ソース管理を Azure Pipelines と統合する
# 学生用ラボ マニュアル

## ラボの概要

Azure DevOps の導入により、Microsoft は開発者に Azure Pipelines と呼ばれる新しい継続的インテグレーション/継続的デリバリー (CI/CD) サービスを提供します。これにより、任意のプラットフォームまたはクラウドに継続的にビルド、テスト、デプロイできます。Linux、macOS、Windows 向けのクラウドホスト型エージェントがあり、ネイティブ コンテナーをサポートし、Kubernetes、VM、サーバーレス環境に柔軟にデプロイできます。

Azure Pipelines は、無制限の CI/CD 分と 10 の並列ジョブをすべての Git Hub オープン ソース プロジェクトに無料で提供します。すべてのオープン ソース プロジェクトは、有料の顧客が使用するのと同じインフラストラクチャで実行されます。つまり、同じ高速パフォーマンスと高品質のサービスが得られます。Atom、CPython、Pipenv、Tox、Visual Studio Code、TypeScript など、トップのオープン ソース プロジェクトの多くはすでに CI/CD 用の Azure Pipelines を使用しており、そのリストは日々増え続けています。

このラボでは、GitHub プロジェクトを使用して Azure Pipelines を簡単にセットアップできることと、メリットをすぐに確認できることを確認します。

## 目標

このラボを完了すると、次のことができるようになります。

-   GitHub Marketplace から Azure Pipelines をインストールします。
-   GitHub プロジェクトを Azure DevOps パイプラインと統合します。
-   パイプラインを介してプル要求を追跡します。

## ラボの所要時間

-   推定時間: **60 分**

## 指示

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボに必要なアプリケーションを確認する

このラボで使用するアプリケーションを特定する:
    
-   Microsoft Edge

#### Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)の手順に従って作成してください。

#### GitHub アカウントを設定する

このラボで使用できる GitHub アカウントをまだお持ちでない場合は、[新しい GitHub アカウントのサインアップ](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account)にある手順に従ってください。

### 演習 1: Azure Pipelines の使用を開始する

この演習では、Marketplace の新しいAzure Pipelines 統合を使用して、GitHub プロジェクトをAzure DevOps と統合します。

#### タスク 1: GitHub リポジトリをフォークして Azure Pipelines をインストールする

このタスクでは、GitHub リポジトリをフォークし、GitHub アカウントに AzurePipelines をインストールします。

1.  GitHub にまだサインインしていない場合は、ラボ コンピューターで、Web ブラウザーを起動し、[GitHub アクションデモ/電卓サイト](https://github.com/actionsdemos/calculator)サイトに移動し、今すぐサインインします。

    > **注**: これは、このラボでフォークして使用するベースライン プロジェクトです。

1.  **アクションデモ/電卓サイト** ページで、「**フォーク**」 をクリックして、リポジトリを自分の GitHub アカウントにフォークします。プロンプトが表示されたら、リポジトリをフォークするアカウントを選択します。

    > **注**: **GitHub Marketplace** には、プロジェクト ワークフローの拡張に役立つ、Microsoft およびサードパーティのさまざまなツールが用意されています。 

1.  フォークされたリポジトリを表示しているページのトップ メニューで、「**Marketplace**」 をクリックします。
1.  「**アプリとアクションの検索**」 に「**Azure Pipelines**」と入力し、**Enter** キーを押して、結果のリストで 「**Azure Pipelines**」 をクリックします。 
1.  「**Azure Pipelines**」 ページで、「**続きを読む**」 をクリックし、Azure Pipelines の利点を確認します。

    > **注**: Azure Pipelines オファリングは、パブリック リポジトリに誰でも無料で使用でき、プライベート リポジトリを使用している場合は単一のビルド キューに無料で使用できます。 

1.  「**Azure Pipelines**」 ページで、「**無料でインストール**」 をクリックします。複数の **GitHub** アカウントがある場合は、「**請求先アカウントの切り替え**」 ドロップダウンから計算機をフォークしたアカウントを選択します。
1.  「**注文の確認**」 ページで、「**注文の完了とインストールの開始**」 をクリックします。
1.  「**Azure Pipelines のインストール**」 ページで、デフォルト オプション 「**すべてのリポジトリ**」 を使用して、「**インストール**」 をクリックします。

    > **注**: 含めるリポジトリを指定するオプションがありますが、このラボでは、すべてのリポジトリを含めるだけです。Azure DevOps は、そのサービスを実行するために、リストされた一連のアクセス許可を必要とすることに注意してください。 

1.  プロンプトが表示されたら、GitHub パスワードで認証して続行します。
1.  プロンプトが表示されたら、「**Azure Pipelines プロジェクトのセットアップ**」 ページの 「**Azure DevOps 組織の選択**」 ドロップダウン リストで、Azure DevOps アカウントを選択し、「**新しいプロジェクトの作成**」 をクリックします。
1.  プロンプトが表示されたら、「**Azure Pipelines プロジェクトのセットアップ**」 ページの 「**プロジェクト名**」 テキストボックスに「**外部ソース管理と Azure Pipelines の統合**」と入力し、**プロジェクトの可視性**を**プライベート**に設定したままにして、「**続行**」 をクリックします。
1.  「**Microsoft による Azure Pipelines のページへのアクセス許可**」 ページで、「**Azure Pipelines の承認**」 をクリックします。

### タスク 2: Azure Pipelines プロジェクトを構成する

このタスクでは、前のタスクで作成した GitHub リポジトリのフォークに基づいて Azure Pipelines プロジェクトを構成します。

> **注**: これで Azure DevOps サイトにアクセスし、Azure Pipelines プロジェクトをセットアップする必要があります。 

1.  Azure DevOps ポータルの 「**パイプライン**」 ビューの 「**リポジトリの選択**」 ペインで、前のタスクで作成した GitHub 電卓リポジトリのフォークを選択します。

    > **注**: Azure Pipelines は、既存のテンプレートが適切かどうかを判断するためにプロジェクトを分析します。この場合、推奨されるテンプレートは **Node.js** 用であり、これは私たちのニーズに最適です。いくつかの代替テンプレートも提案されていますが、このラボには推奨されるテンプレートが最適です。 
1.  「**パイプラインの構成**」 で、「**Node.js**」 を選択します。

    > **注**: ビルド パイプラインは **YAML** として定義されます。これは、リポジトリ内の他のファイルと同じようにパイプラインの構成を管理できるため、このようなプロセスの定義に適したマークアップ構文です。これは、ビルド用に VM をプルするプール、ビルド用に Node.js をインストールするプロセス、および実際のビルド自体を識別する非常に単純なテンプレートです。 

1.  「**パイプライン YAML の確認**」で、「**保存して実行**」 をクリックしてパイプラインを保存し、新しいビルドをキューに入れます。
1.  「**保存して実行**」 ペインで、デフォルト設定を受け入れ、「**保存して実行**」 をクリックします。

    > **注**: このラボでは、この新しいファイルをマスター ブランチに直接コミットできます。 

    > **注**: パイプラインが完了するまで少し時間がかかります。この間、ビルド エージェントを構成し、GitHub からソースをプルして、パイプライン定義に従ってビルドします。

1.  ビルド ジョブのペインの 「**概要**」 タブで、ビルドが正常に完了したことを確認します。

### タスク 3: YAML ビルド パイプライン定義を変更する

このタスクでは、フォークされた GitHub リポジトリの YAML ビルド定義を変更し、変更によってトリガーされたビルド プロセスを追跡します。

> **注**: デフォルトのパイプラインは素晴らしいスタートですが、自動化したいすべてのことを実行できるわけではありません。たとえば、テストを実行して、変更によってバグが発生しないことを確認できれば素晴らしいと思います。YAML を手動で編集できる GitHub に戻りましょう。 

1.  ビルド ジョブのペインの 「**概要**」 タブで、「**リポジトリとバージョン**」 ラベルの横にある、このラボで前に作成したフォークをホストしている GitHub プロジェクト リポジトリを表すエントリを右クリックし、「**新しいタブでリンクを開く**」 を選択します。これにより、新しいブラウザー タブが開き、フォークのコンテンツを含む GitHub ページが表示されます。

    > **注**: このラボでは、GitHub と Azure DevOps の間を行ったり来たりする必要があるため、それぞれに対してブラウザー タブを開いたままにしておく方が簡単です。

1.  フォークのコンテンツを表示している GitHub ページで、ファイル **azure-pipelines.yml** を表すエントリを見つけてクリックします。これにより、ファイルが自動的に開き、その内容が表示されます。 
1.  **master/calculator/azure-pipelines.yml** ページのファイル コンテンツを表示しているペインの右上隅にある、鉛筆の形をした 「**このファイルを編集**」 アイコンをクリックします。 

    > **注**: 私たちのプロジェクトにはすでに Mocha を使用して作成されたテストが含まれているため、パイプライン外で実行する必要があります。 

1.  テスト実行を追加するには、同じインデントを使用して、`npm run build` コマンドのすぐ下に `npm test` コマンドを追加します。さらに、`npm test` エントリを `'npm install, build, and test'` に更新して、ビルドの各タスクが実行していることを明確に示します。 

    ```
        npm test
      displayName: 'npm install, build, and test'
    ```

1.  ページの一番下までスクロールし、デフォルトのコミット メッセージを **Adding npm test** に置き換えて、「**変更のコミット**」 をクリックします。 

    > **注**: これもラボ環境であることを考慮して、この変更をマスター ブランチに直接コミットすることは許容されます。

1.  **Azure DevOps** ポータルを表示しているブラウザー タブに戻り、ブレッドクラム ナビゲーションを使用して、「**パイプライン**」 ビューの 「**パイプライン**」 ペインに移動します。
1.  更新によってトリガーされた新しいビルドが、「**最近実行したパイプライン**」 リストの 「**最近**」 タブに既に表示されていることを確認します。パイプラインに対応するエントリをクリックし、「**実行**」 タブで最新の実行を選択し、「**ジョブ**」 セクションで 「**ジョブ**」 エントリをクリックします。
1.  ジョブの詳細を表示しているペインで、ジョブの個々のタスクをクリックして、完了まで実行します。

### タスク 4: GitHub プル要求を介して変更を提案する

このタスクでは、無効な変更を提案し、プル要求によってトリガーされたビルドの結果を確認します。

> **注**: このパイプライン設定の大きな利点の 1 つは、誰かが変更をコミットするたびに自動的に実行される品質ゲートがあることです。これにより、プロジェクトの管理がはるかに簡単になり、さまざまなレベルの品質が向上する可能性があります。 

1.  **azure-pipelines.yml** ファイルのコンテンツを表示する GitHub ページを表示しているブラウザー タブに戻り、フォークされたリポジトリのコンテンツを一覧表示しているページに戻り、「**ファイルに移動**」 をクリックします。
1.  **電卓/** プロンプトで、**arithticController.js** と入力し、結果のリストで **api/controllers/arithmeticController.js** をクリックします。これにより、ブラウザー セッションが自動的に **master/calculator/api/controllers/arithmeticController.js** ページにリダイレクトされ、そのファイルのコンテンツが表示されます。

    > **注**: このコントローラーには、アプリのコア機能が含まれています。ただし、**追加**操作のコードは完全には明確ではありません。やる気はあるが、Java Scriptの経験が不足している人の立場に立って考えてください。彼らはこれを、コードをクリーンアップして改善することで支援する機会としてとらえるかもしれません。

1.  **master/calculator/api/controllers/arithmeticController.js** ページで、ファイルの内容を表示しているペインの右上隅にある、鉛筆の形をした 「**このファイルを編集**」 アイコンをクリックします。 
1.  行 `    'add':      function(a,b) { return +a + +b },` を `    'add':      function(a,b) { return a + b },` に変更します。

    > **注**: これは誤った変更であり、無効な結果になります。

1.  ページの一番下までスクロールし、デフォルトのコミット メッセージを「**追加機能の変更**」に置き換え、「**新しいブランチの作成**」 を選択し、その名前を「**addition-cleanup**」に設定して、「**ファイルの変更を提案**」 をクリックします。
1.  「**プル要求を開く**」 ページで、「**プル要求の作成**」 をクリックして、テストされていない変更を本番コードに取り込むプロセスを開始します。 

    > **注**: Azure DevOpsは 変更を検出し、ビルド パイプラインを開始します。これにより、チェックが失敗し、GitHub UI の更新がトリガーされます。 

    > **注**: 「プロジェクト所有者」の元の考え方に戻ります。

1.  「**追加機能 #1 の変更**」 プル要求ページの 「**すべてのチェックに失敗しました**」 セクションで、「**詳細**」 をクリックして詳細を確認します。
1.  「**注釈**」 セクションを確認し、そのすぐ下にある 「**Azure Pipelines の詳細を表示する**」 リンクをクリックします。これにより、新しいブラウザー タブが開き、Azure DevOps ポータルで失敗したジョブの実行が表示されます。
1.  Azure portal の失敗したジョブ ペインで、「**ジョブ**」 エントリをクリックして詳細を表示します。 
1.  ジョブ タスクのリストで、**npm install、build、test** タスクをクリックして、その出力を表示します。
1.  失敗したテストをリストするセクションを見つけます。 

    > **注**: テストが失敗した理由はすぐにはわからない場合がありますが、パイプラインで蓄積されたすべての履歴により、この新しいプル要求からの何かが原因であることが簡単に特定できます。次のステップは、「21 +21」が予想される「42」ではなく「2121」を生成した理由を理解することです。

1.  Azure DevOp sポータルで失敗したジョブの実行を表示しているタブを閉じます。

### タスク 5: 壊れたプル要求を使用してプロジェクトを改善する

このタスクでは、前のタスクで作成されたプル要求で導入された無効な変更を修正します。

> **注**: 「プロジェクト所有者」の元の考え方に戻ります。

1.  **追加機能 #1 の変更** GitHub ページを表示しているブラウザー タブに戻り、フォークされたリポジトリのコンテンツを一覧表示するメイン ページに戻ります。 
1.  リポジトリ ファイルのリストを含むペインの上部で、**リクエストをプルし**、最新のプル要求を表すエントリをクリックします。
1.  **追加機能 #1 の変更** GitHub ページで、「**変更されたファイル**」 タブをクリックし、その内容を確認します。

    > **注**: 変更は、各変数の前のプラス記号がそれらの変数をそれらの数値表現に強制変換するために必要であることに気付いていない誰かによって行われたようです。それらを削除することにより、Java Script は中央のプラス記号を文字列連結演算子として解釈しました。これは、失敗したテストで 21 + 21 = 2121 である理由を説明しています。 

1.  「**追加機能 #1 の変更**」 GitHub ページで、「**変更の確認**」 ボタンのすぐ下にある省略記号をクリックし、ドロップダウン メニューで 「**ファイルの編集**」 をクリックします。 
1.  **a** 変数と **b** 変数の前にプラス記号を追加して元の変更を元に戻し、結果として 'add':      function(a,b) { return +a + +b },` になります。さらに、前の行に`// +演算子を使用して、文字列の連結を防ぐためにキャスト変数を整数として入力する」というコメントを含めます。
1.  ページの一番下までスクロールし、デフォルトのコミット メッセージを「**追加機能の修正**」に置き換え、「**追加クリーンアップ ブランチに直接コミットする**」 オプションが選択されていることを確認して、「**変更のコミット**」 をクリックします。
1.  **追加関数 #1 の変更** GitHub ページで、「**会話**」 タブを選択します。

    > **注**: Azure DevOps は変更を再度検出し、ビルド パイプラインを開始します。すべてのチェックに合格するのを待ちます。 

1.  すべてのチェックに合格したら、「**プル要求のマージ**」 をクリックし、「**マージの確認**」 をクリックします。

### タスク 6: ビルド ステータス バッジを追加する

このタスクでは、ビルド ステータス バッジを GitHub リポジトリに追加します。

> **注**: 高品質プロジェクトの重要な兆候は、ビルド ステータス バッジです。プロジェクトが現在正常なビルド状態にあることを示すバッジが付いているプロジェクトを誰かが見つけた場合、それはプロジェクトが効果的に維持されていることを示しています。 

1.  **Azure DevOps** ポータルを表示しているブラウザー タブに戻り、ブレッドクラム ナビゲーションを使用して、**パイプライン** ビューの 「**パイプライン**」 ペインの 「**最近**」 タブに移動します。
1.  「**最近実行したパイプライン**」 リストの 「**最近**」 タブで、このラボで使用したパイプラインに対応するエントリをクリックします。 
1.  パイプライン ペインで、右上隅の省略記号をクリックし、ドロップダウンリスト で 「**ステータス バッジ**」 を選択します。 

    > **注**: **ステータス バッジ** UIを使用すると、ビルド ステータスをすばやく簡単に参照できます。多くの場合、提供された URL を独自のダッシュボードで使用するか、Markdown スニペットを使用して Wiki ページなどの場所にステータス バッジを追加できます。 

1.  「**ステータス バッジ**」 ペインで、「**サンプル Markdown**」 の 「**クリップボードにコピー**」 ボタンをクリックします。
1.  フォークされたリポジトリのコンテンツを表示する GitHub ページを表示しているブラウザー タブに戻り、必要に応じて 「**<> コード**」 タブをクリックします。
1.  リポジトリ ファイルのリストで、「**README.md**」 をクリックし、**master/calculator/README.md** ページで、ファイルの内容を表示しているペインの右上隅にある 「**このファイルを編集**」 アイコンをクリックします。 
1.  6 行目の上に行を追加し、クリップボードの内容を貼り付けます。
1.  ページの一番下までスクロールし、デフォルトのコミット メッセージを「**Azure Pipelines ステータス バッジの追加**」 に置き換えて、「**変更のコミット**」 をクリックします。 

    > **注**: で、プロジェクトのフロント ページに動的なビルド ステータス バッジが表示され、プロジェクトを効果的に管理していることを全員に知らせることができます。

#### レビュー

このラボでは、Marketplace の新しいAzure Pipelines 統合を使用して、GitHub プロジェクトをAzure DevOps と統合しました。