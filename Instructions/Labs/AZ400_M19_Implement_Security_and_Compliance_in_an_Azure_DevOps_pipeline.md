---
lab:
    title: 'ラボ: Azure DevOps パイプラインにセキュリティとコンプライアンスを実装する'
    module: 'モジュール 19: DevOps プロジェクトでのセキュリティの実装'
---

# ラボ: Azure DevOps パイプラインにセキュリティとコンプライアンスを実装する
# 学生用ラボ マニュアル

## ラボの概要

このラボでは、**WhiteSource Bolt と Azure DevOps** を使用して、脆弱なオープン ソース コンポーネント、古いライブラリ、コードのライセンス コンプライアンスの問題を自動的に検出します。WebGoat を使用しますが、これは一般的な Web アプリケーションのセキュリティの問題を説明するために設計された OWASP によって維持される、意図的に安全でない Web アプリケーションです。

[WhiteSource](https://www.whitesourcesoftware.com/) は、トップクラスの継続的オープン ソース ソフトウェア セキュリティおよびコンプライアンス管理のリーダーです。WhiteSource は、プログラミング言語、ビルド ツール、または開発環境に関係なく、ビルド プロセスに統合します。自動的かつ継続的にバックグラウンドでサイレントに機能し、WhiteSource の常に更新されているオープン ソース リポジトリの最終データベースに対してオープン ソース コンポーネントのセキュリティ、ライセンス、品質を確認します。

WhiteSource の提供する WhiteSource Bolt は、特に Azure DevOps および Azure DevOps Server との統合向けに開発されたライトウェイト オープン ソース セキュリティおよび管理ソリューションです。WhiteSource Bolt はプロジェクトに基づいて機能し、リアルタイムのアラート機能が備えられていないため、**フル プラットフォーム**が必要です。これは通常、ソフトウェア開発サイクル全体 (リポジトリから開発後段階) を通して、あらゆるプロジェクトと製品でオープン ソース管理の自動化を希望する、より大きな開発チームで推奨されるものです。

Azure DevOps を WhiteSource Bolt に同等すると以下が可能になります:

- 脆弱なオープンソースコンポーネントを検出して修正します。
- プロジェクトまたはビルドごとに包括的なオープン ソース インベントリ レポートを生成します。
- 依存関係のライセンスを含む、オープン ソース ライセンスのコンプライアンスを実施します。
- 更新の推奨事項がある古いオープン ソース ライブラリを特定します。

## 目標

このラボを完了すると、次のことができるようになります。

- WhiteSource Bolt をアクティブにする
- ビルド パイプラインを実行し、WhiteSource のセキュリティおよびコンプライアンス レポートをレビューする

## ラボの所要時間

-   推定時間: **45 分**

## 指示

### 開始する前に

#### ラボの仮想マシンにログインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops) で利用できる手順に従って作成してください。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[Parts Unlimited MRP GitHub リポジトリ](https://www.github.com/microsoft/partsunlimitedmrp)に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### タスク 1: チーム プロジェクトを作成して構成する

このタスクでは、Azure DevOps Demo Generator を使用し、[WhiteSource-Bolt テンプレート](https://azuredevopsdemogenerator.azurewebsites.net/?name=WhiteSource-Bolt&templateid=77362)に基づいて新しいプロジェクトを生成します

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen をご覧ください。

1.  「**サインイン**」 をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」 ページで 「**承諾する**」 をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」 ページで 「**新しいプロジェクト名**」 テキストボックスに「**WhiteSource Bolt**」と入力します。「**組織の選択**」 ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」 をクリックします。
1.  テンプレートのリストのツールバーで 「**DevOps ラボ**」 をクリックし、「**WhiteSource Bolt**」 テンプレートを選択して 「**テンプレートの選択**」 をクリックします。
1.  再び 「**新しいプロジェクトの作成**」 ページで、欠落している拡張機能をインストールするよう指示されたら 「**WhiteSource Bolt**」 の下にあるチェックボックスを選択し、「**プロジェクトの作成**」 をクリックします。。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」 ページで 「**プロジェクトに移動**」 をクリックします。

### 演習 1: WhiteSource Bolt を使用して Azure DevOps パイプラインにセキュリティとコンプライアンスを実装する

この演習では、WhiteSource Bolt を利用し、セキュリティの脆弱性とライセンスのコンプライアンスの問題に関してプロジェクト コードをスキャンし、その結果作成されるレポートを表示します。

#### タスク 1: WhiteSource Bolt をアクティブにする

このタスクでは、新しい生成された Azure Devops プロジェクトで WhiteSource Bolt をアクティブにします。

1.  ラボのコンピューターを使い、Azure DevOps ポータルが表示され、**WhiteSource Bolt** プロジェクトが開いている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーから 「**Pipelines**」 をクリックし、「**Pipelines**」 セクションで 「**WhiteSource Bolt**」 をクリックします。
1.  「**You're almost there（あと一歩）**」 ペインで、「**勤務先の電子メール**」 と 「**会社名**」 を入力し、「**国**」 ドロップダウン リストで居住国を示すエントリを選択します。「*Get Started*」 ボタンをクリックして、WhiteSource Bolt の *Free*バージョンの使用を開始します。新しいブラウザー タブが自動的に開き、「**Bolt の使用開始**」 ページが表示されます。 
1.  Azure DevOps ポータルが表示されている Web ブラウザー タブに戻り、「**You are using a FREE version of WhiteSource Bolt**」と表示されていることを確認します。

#### タスク 2: ビルドをトリガーする

このタスクでは、Java コード ベースの Azure DevOps プロジェクト内でビルドをトリガーします。**WhiteSource Bolt** 拡張機能を使い、このコードに含まれている脆弱なコンポーネントを特定します。

1.  **WhiteSource Bolt** プロジェクトが開いている Web ブラウザー ウィンドウで、 左側にある垂直メニュー バーの 「**Pipelines**」 セクションで 「**Pipelines**」 をクリックします。
1.  「**Pipelines**」 ペインで 「**WhiteSourceBolt**」 ビルドの定義を選択し、「**Run Pipeline**」 をクリックします。「**Run pipeline**」 ペインで 「**Rum**」 をクリックしてビルドをトリガーします。
1.  ビルド ペインの 「**Summary**」 タブの 「**Jobs**」 セクションで 「**Phase 1**」 をクリックし、ビルド プロセスの進捗状況を監視します。

    > **注**: ビルドの定義は以下のタスクで構成されます。

    | タスク | 使用状況 |
    | ---- | ------ |
    | ![npm](images/m19/npm.png) **npm** |  ビルドに必要な npm パッケージをインストールして発行します |
    | ![maven](images/m19/maven.png) **Maven** |  提供された pom xml ファイルを使って Java コードをビルドします |
    | ![whitesourcebolt](images/m19/whitesourcebolt.png) **WhiteSource Bolt** |  提供された作業ディレクトリ / ルート ディレクトリでコードをスキャンし、セキュリティの脆弱性、問題のあるオープン ソース ライセンスを検出します |
    | ![Copy-files](images/m19/copy-files.png) **Copy Files** |  一致パターンを使い、結果的な JAR ファイルをソースからコピー先フォルダーにコピーします |
    | ![Publish-build-artifacts](images/m19/publish-build-artifacts.png) **Publish Artifact** |  ビルドによって生成された成果物を発行します |
    
1.  ビルドが完了したら、「**Summary**」 タブに戻り、「**Tests**」 タブをレビューします。 

#### タスク 3: レポートを分析する

このタスクでは、WhiteSource Bolt Build Reportをレビューします。 

1.  ビルド ペインで 「**WhiteSource Bolt Build Report**」 タブの見出しをクリックし、レポートが完全に表示されるのを待ちます。 
1.  「**WhiteSource Bolt Build Report**」 タブで、WhiteSource Bolt がソフトウェアのオープン ソース コンポーネント (推移的な依存関係とそのライセンスを含む) を自動的に検出したことを確認します。
1.  「**WhiteSource Bolt Build Report**」 タブで、ビルド中に発見された脆弱性が表示されたセキュリティ ダッシュボードをレビューします。

    > **注**: レポートには、あらゆる脆弱なオープン ソース コンポーネントのリストが表示されます。この中には、**脆弱性スコア**、**脆弱なライブラリ**、**重大度配分**が含まれます。あらゆるコンポーネントおよびメタデータとライセンス済み参照へのリンクの詳細なビューを利用して、オープンソース ライセンス配布を特定します。

1.  「**WhiteSource Bolt Build Report**」 タブで 「**Outdated Libraries**」 セクションまでスクロールダウンして、その内容をレビューします。

    > **注**: WhiteSource Bolt はプロジェクトの古いライブラリを追跡し、ライブラリの詳細、より新しいバージョンへのリンク、修復の推奨を提供します。

## レビュー

このラボでは、**WhiteSource Bolt と Azure DevOps** を使用して、脆弱なオープン ソース コンポーネント、古いライブラリ、コードのライセンス コンプライアンスの問題を自動的に検出します。
