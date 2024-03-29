---
lab:
    title: 'ラボ: YAML を使用したコードとしてのパイプラインの構築'
    module: 'モジュール 11: Azure Pipelines を使用した継続的デプロイの実装'
---

# ラボ: YAML を使用したコードとしてのパイプラインの構築
# 学生用ラボ マニュアル

## ラボの概要

多くのチームは YAML を使用してビルドを定義し、パイプラインをリリースすることを好みます。これにより、ビジュアル デザイナーを使用した場合と同じパイプラインの機能にアクセスできます。ただし、マークアップ ファイルは他のソース ファイルと同様に管理できます。YAML ビルドの定義は、該当するファイルをリポジトリのルートに加えるだけでプロジェクトに追加できます。Azure DevOps は、人気の高いプロジェクトの種類の既定テンプレートのほか、YAML デザイナーを提供して、ビルドおよびリリース タスクの定義プロセスを簡素化します。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する 

## ラボの所要時間

-   推定時間: **60 分**

## 指示

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops) で利用できる手順に従って作成してください。

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートと Azure リソースに基づいて事前に構成された Parts Unlimited チーム プロジェクトで構成され、Azure Web アプリと Azure SQL データベースが含まれます。 

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Parts Unlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen をご覧ください。

1.  「**サインイン**」 をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」 ページで 「**承諾する**」 をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」 ページで 「**新しいプロジェクト名**」 テキストボックスに「**YAML を使用したパイプラインのコードとしての構成**」と入力します。「**組織の選択**」 ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」 をクリックします。
1.  テンプレートのリストのツールバーで 「**DevOps ラボ**」 をクリックし、「**PartsUnlimited-YAML**」 テンプレートを選択して 「**テンプレートの選択**」 をクリックします。
1.  再び 「**新しいプロジェクトの作成**」 ページで 「**プロジェクトの作成**」 をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」 ページで 「**プロジェクトに移動**」 をクリックします。

#### タスク 2: Azure リソースを作成する

このタスクでは、Azure portal を使用して Azure Web アプリと Azure SQL データベースを作成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで所有者のロールがあり、このサブスクリプションに関連のある Azure ADテナントでグローバル管理者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal で、ページの左上にある横線 3 本のアイコンをクリックし、ハブ メニューでは 「**リソースの作成**」 をクリックします。
1.  「**新規**」 ブレードの検索テキスト ボックスに「**Web アプリ + SQL**」と入力してから **Enter** キーを押します。
1.  「**Web アプリ + SQL**」 で 「**作成**」 をクリックします。
1.  「**Web アプリ + SQL**」 ブレードで以下の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | アプリの名前 | 有効でグローバルに一意の DNS ホスト名 |
    | 定期売買 | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ「**az400m11l01-RG**」の名前 |

1.  「**App Service プラン/場所**」 をクリックし、「**App Service プラン**」 ブレードで 「**新規作成**」 をクリックします。
1.  「**新しい App Service プラン**」 ブレードで以下の設定を指定して 「**OK**」 をクリックします。

    | 設定 | 値 |
    | --- | --- |
    | App Service プラン | 有効な名前 |
    | 場所| このラボで使用するリソースをデプロイしたい Azure リージョンの名前 |
    | 価格レベル | **D1 Shared** |

1.  再び 「**Web アプリ + SQL**」 ブレードで 「**SQL データベース**」 をクリックします。
1.  「**SQL データベース**」 ブレードで 「**名前**」 テキストボックスに「**partsunlimited**」と入力します。
1.  「**SQL データベース**」 ブレードで 「**ターゲット サーバー**」 をクリックします。
1.  「**新しいサーバー**」 ブレードで以下の設定を指定して 「**選択**」 をクリックします。

    | 設定 | 値 |
    | --- | --- |
    | サーバー名 | 有効でグローバルに一意の DNS ホスト名 |
    | サーバー管理者のログイン | Student |
    | パスワード | Pa55w.rd1234 |
    | 場所 | App Service プラン向けに選択した Azure リージョンの名前 |
    | Azure サービスにサーバーへのアクセスを許可する | enabled |

1.  「**SQL データベース**」 ブレードで 「**選択**」 をクリックします。
1.  再び 「**Web アプリ + SQL**」 ブレードで 「**作成**」 をクリックします。 

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかります。 

### 演習 1: Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する

この演習では、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成します。

#### タスク 1: 既存のパイプラインの実行を無効にする

このタスクでは、既存のパイプラインの実行を無効にします。

1.  ラボのコンピューターで、Azure DevOps ポータルに 「**YAML を使用してパイプラインをコードとして構成する**」 プロジェクトが表示されているブラウザー ウィンドウに切り替え、垂直のナビゲーション ペインで 「**パイプライン**」 を選択します。

    > **注**: YAML パイプラインを構成する前に、既存のビルド パイプラインを無効にします。

1.  「**パイプライン**」 ペインで 「**PartsUnlimited**」 エントリを選択します。 
1.  「**PartsUnlimited**」 ブレードの右上コーナーで、垂直の省略記号をクリックし、ドロップダウン メニューで 「**設定**」 を選択します。
1.  「**パイプライン設定**」 ペインで 「**一時停止**」 オプションを選択します。

### タスク 2: YAML ビルド定義を追加する

このタスクでは、既存のプロジェクトに YAML ビルドの定義を追加します。

1.  「**パイプライン**」 ハブで 「**パイプライン**」 に戻ります。 
1.  「**パイプライン**」 ペインの右上コーナーで 「**新しいパイプライン**」 をクリックします。 

    > **注**: ウィザードを使い、プロジェクトに基づいて YAML の定義を自動的に作成します。

1.  「**コードはどこにありますか?**」 ペインで 「**Azure Repos Git (YAML)**」 オプションをクリックします。
1.  「**リポジトリの選択**」 ペインで 「**PartsUnlimited**」 をクリックします。
1.  「**パイプラインの構成**」 ペインで 「**ASP.NET**」 をクリックし、このテンプレートをパイプラインの起点として利用します。これにより、「**パイプライン YAML のレビュー**」 ペインが開きます。

    > **注**: パイプラインの定義は、リポジトリのルートで「**azure-pipelines.yml**」という名前のファイルとして保存されます。このファイルには、典型的な ASP.NET ソリューションを構築してテストするために必要なステップが含まれます。また、必要に応じてビルドをカスタマイズすることもできます。このシナリオでは、**プール**を更新し、Visual Studio 2017 を実行している VM を使用できるようにします。

1.  「**パイプライン YAML のレビュー**」 ペインの **10** 行目で、`vmImage: 'windows-latest'` を `vmImage: 'vs2017-win2016'` に置き換えます。
1.  「**パイプライン YAML のレビュー**」 ペインで 「**保存して実行**」 をクリックします。
1.  「**保存して実行**」 ペインで既定の設定を承諾し、「**保存して実行**」 をクリックします。
1.  パイプライン実行ペインの 「**ジョブ**」 セクションで 「**ジョブ**」 をクリックし、進捗状況を監視して、完了したことを確認します。 

    > **注**: 警告やエラーなど、YAML ファイルの各タスクをレビューできます。

1.  パイプライン実行ペインに戻り、「**概要**」 タブから 「**テスト**」 タブに切り替えてテスト統計情報をレビューします。

#### タスク 3: YAML 定義に継続的デリバリーを追加する

このタスクでは、以前のタスクで作成したパイプラインの YAML ベースの定義に継続的デリバリーを追加します。

> **注**: ビルドとテストのプロセスに成功すると、YAML 定義にデリバリーを追加できます。 

1.  パイプライン実行ペインで、右上コーナーにある省略記号をクリックし、ドロップダウン メニューで 「**パイプラインの編集**」 をクリックします。
1.  **Azure-pipelines-1.yaml** ファイルのコンテンツが表示されているペインで **8** 行目の `trigger` セクションの後に以下のコンテンツを追加し、YAML パイプラインの 「**ビルド**」 ステージを定義します。 

    > **注**: パイプラインの進捗状況をよりよく整理して追跡するために必要なステージを定義できます。

    ```yaml
    stages:
    - stage: Build
      jobs:
      - job: Build
    ```

1.  YAML ファイルの残りのコンテンツを選択し、**Tab** キーを 2 回押してスペース 4 つ分をインデントします。 

    > **注**: これにより、`pool` セクションで始まるものはすべて `job: Build` の一部になります。 

1.  ファイルの最下部で、以下の項性を追加して 2 番目のステージを定義します。

    ```
    - stage: Deploy
      jobs:
      - job: Deploy
        pool:
          vmImage: 'vs2017-win2016'
        steps:
    ```

1.  YAML 定義の最後で新しいライン上にカーソルを配置します。 

    > **注**: これは、新しいタスクが追加される場所になります。

1.  コード ペインの右側のタスク一覧で、「**Azure App Service のデプロイ**」 タスクを検索して選択します。
1.  「**Azure App Service のデプロイ**」 ペインで以下の設定を指定し、「**追加**」 をクリックします。

- 「**Azure サブスクリプション**」 ドロップダウン リストで、ラボで以前に Azure リソースをデプロイした Azure サブスクリプションを選択し、「**承認**」 をクリックします。指示されたら、Azure リソース デプロイの際に使用したものと同じユーザー アカウントを使用して認証します。
- 「**App Service の名前**」 ドロップダウン リストで、ラボで以前にデプロイした Web アプリの名前を選択します。 
- 「**パッケージまたはフォルダー**」 テキストボックスで `$(System.ArtifactsDirectory)/drop/*.zip` と入力します。 

    > **注**: これにより、デプロイ タスクが YAML パイプラインの定義に自動的に追加されます。

1.  エディターで追加されたタスクを選択したままの状態にして、**Tab** キーを 2 回押し、スペース 4 つ分をインデントします。これにより、**steps** タスクの子としてリストされます。

    > **注**: **packageForLinux** パラメーターは、このラボの状況では不適切ですが、Windows または Linux では有効です。 

    > **注**: 既定により、2 つのステージは独立して実行されます。このため、最初のステージのビルド出力は、さらに変更を加えなければ 2 番目のステージで利用できない可能性があります。このような変更を実装するには、ひとつのタスクを使用してビルド出力をビルド ステージの最後に公開し、別のタスクを使ってデプロイ ステージの最初にこれをダウンロードします。 

1.  ビルド ステージの最後に空白のライン上にカーソルを配置します。
1.  「**タスク**」 ペインで 「**ビルド成果物の公開**」 タスクを検索して選択します。 
1.  「**ビルド成果物の公開**」 ペインで既定の設定を承諾し、「**追加**」 をクリックします。 

    > **注**: これにより、「**drop**」というエイリアスでダウンロードできる場所にビルド成果物が公開されます。

1.  エディターで追加されたタスクを選択したままの状態にして、**Tab** キーを 2 回押し、スペース 4 つ分をインデントします。

    > **注**: また、前後に空白のラインを追加して読みやすくすることをお勧めします。

1.  デプロイ ステージの **steps** ノードで最初のラインにカーソルを配置します。
1.  「**タスク**」 ペインで 「**ビルド成果物のダウンロード**」 タスクを検索して選択します。
1.  「**追加**」 をクリックします。
1.  エディターで追加されたタスクを選択したままの状態にして、**Tab** キーを 2 回押し、スペース 4 つ分をインデントします。 

    > **注**: ここでも、前後に空白のラインを追加して読みやすくすることをお勧めします。

1.  ダウンロード タスクにプロパティを追加し、`drop` の `artifactName` を指定します (必ずスペースを一致させてください):

    ```
    artifactName: 'drop'
    ```

1.  「**保存**」 をクリックし、「**保存**」 ペインで再び 「**保存**」 をクリックして変更を直接、マスター ブランチにコミットします。

    > **注**: これにより、新しいビルドが自動的にトリガーされます。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウの垂直ナビゲーション ペインで、「**パイプライン**」 を選択します。
1.  「**パイプライン**」 ペインで、新しく構成されたパイプラインを示すエントリをクリックします。 
1.  実行が一覧表示されている 「**PartsUnlimited**」 パイプライン ペインの右上コーナーで、垂直省略記号をクリックし、ドロップダウン メニューで 「**設定**」 を選択します。
1.  「**パイプライン設定**」 ペインの 「**YAML ファイル パス**」 ドロップダウン リストで 「**azure-pipelines-1.yml**」 エントリを選択します。そのすぐ上で 「**有効**」 オプションを選択し、「**保存**」 をクリックします。 
1.  再び実行を一覧表示している 「**PartsUnlimited**」 パイプライン ペインの垂直ナビゲーション ペインで 「**パイプライン**」 を選択します。
1.  「**パイプライン**」 ペインで、新しく構成されたパイプラインを示すエントリの上にマウスのポインターを配置すると、右側に省略記号が表示されます。その省略記号をクリックし、ドロップダウン メニューで 「**パイプラインの実行**」 を選択します。
1.  「**概要**」 ペインで、パイプライン実行の進捗状況を監視します。
1.  「*このパイプラインは、この実行のデプロイを継続する前にリソースのアクセス許可が必要です*」というメッセージが表示されたら、「**表示**」 をクリックします。「**レビューの待機中**」 ダイアログ ボックスで 「**許可**」 をクリックします。「**アクセス許可?**」 ペインで再び 「**許可**」 をクリックします。
1.  「**概要**」 ペインの最下部で 「**デプロイ**」 ステージをクリックすると、デプロイの詳細が表示されます。

    > **注**: タスクが完了すると、アプリが Azure Web アプリにデプロイされます。

#### タスク 4: デプロイされたサイトをレビューする

1.  Azure portal を表示している Web ブラウザー ウィンドウに戻り、Azure Web アプリのプロパティが表示されているブレードに移動します。
1.  Azure Web アプリ ブレードの 「**設定**」 セクションで 「**構成**」 を選択します。
1.  Azure Web アプリ構成ブレードの 「**接続文字列**」 セクションで 「**defaultConnection**」 エントリをクリックします。
1.  「**接続文字列の追加/編集**」 ブレードの 「**名前**」 テキストボックスで、現在の値を **DefaultConnectionString** (アプリケーションで期待されているキー) に変更し、「**OK**」 をクリックします。

    > **注**: これにより、アプリ サービス向けに作成されたデータベースにアプリケーションが接続できるようになります。 

1.  再び Azure Web アプリ構成ブレードで 「**保存**」 をクリックし、変更を適用します。指示されたら 「**続行**」 をクリックします。
1.  Azure Web アプリ ブレードで 「**概要**」 をクリックします。概要ブレードで 「**参照**」 をクリックし、新しい Web ブラウザー タブでサイトを開きます。
1.  デプロイされたサイトが新しいブラウザー タブで期待されているように読み込まれていることを確認します。

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l01-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成しました
