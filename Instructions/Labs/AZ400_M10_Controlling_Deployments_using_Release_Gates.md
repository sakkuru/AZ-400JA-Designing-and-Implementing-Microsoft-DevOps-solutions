---
lab:
  title: "ラボ: Controlling Deployments using Release Gates"
  module: "モジュール 10: リリース戦略の設計"
---

# ラボ: リリース ゲートを使用したデプロイの制御

# 学生用ラボ マニュアル

## ラボの概要

このラボでは、デプロイ ゲートの構成と、それらを使用して Azure Pipelines の実行を制御する方法について詳しく説明します。それらの実装を説明するために、Azure Web アプリの 2 つの環境でリリース定義を構成します。アプリのブロック バグがない場合にのみ、Canary Environments にデプロイし、Azure Monitor の Application Insights にアクティブなアラートがない場合にのみ、Canary Environments を完了としてマークします。

リリース パイプラインは、さまざまな環境にデプロイされるアプリケーションのエンドツーエンドのリリース プロセスを指定します。各環境へのデプロイは、ジョブとタスクを使用して完全に自動化されます。アプリケーションに対する新しい更新プログラムをすべてのユーザーに同時に公開することは理想的です。更新を段階的に公開する、つまり、ユーザーのサブセットに公開し、使用状況を監視し、最初のユーザー セットの経験に基づいて他のユーザーに公開するのがベストプラクティスです。

承認とゲートにより、リリースのデプロイの開始と完了を制御できます。承認により、ユーザーがデプロイを手動で承認または拒否するのを待つことができます。リリース ゲートを使用すると、リリースが次の環境に昇格する前に満たす必要があるアプリケーション正常性基準を指定できます。環境デプロイの前後に、指定されたすべてのゲートは、すべてのゲートが合格するか、定義されたタイムアウト期間に達して失敗するまで自動的に評価されます。

ゲートは、Pre-deployment conditions または Post-deployment conditions パネルから、リリース定義の環境に追加できます。環境条件に複数のゲートを追加して、リリースに対するすべての入力が正常に行われるようにすることができます。

例

- デプロイ前のゲートは、ビルドを環境にデプロイする前に、作業項目または問題管理システムにアクティブな問題がないことを確認します。
- デプロイ後のゲートは、アプリがデプロイされた後、次の環境にリリースをプロモートする前に、アプリの監視システムまたはインシデント管理システムからのインシデントがないことを確認します。

すべてのアカウントには、既定で 4 種類のゲートが含まれています。

- Azure 関数の呼び出し: Azure 関数の実行をトリガーし、正常に完了するようにします。
- Azure Monitor アラートのクエリ: アクティブなアラートの構成済みの Azure Monitor 警告ルールを確認します。
- REST API の呼び出し: REST API を呼び出して、成功した応答が返されたら続行します。
- Query Work Items: クエリから返された一致する作業項目の数がしきい値内であることを確認してください。

## 目標

このラボを完了すると、次のことができるようになります。

- リリース パイプラインを構成する
- リリース ゲートを構成する
- リリース ゲートをテストする

## ラボの所要時間

- 推定時間: **75 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。

- ユーザー名: **Student**
- パスワード: **Pa55w.rd**

#### このラボに必要なアプリケーションを確認する

このラボで使用するアプリケーションを特定する:

- Microsoft Edge

#### Azure DevOps 組織をセットアップします。

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)の手順に従って作成してください。

#### Azure サブスクリプションを準備する

- 既存の Azure サブスクリプションを特定するか、新しいサブスクリプションを作成します。
- Azure サブスクリプションの所有者のロールと Azure サブスクリプションに関連付けられた Azure AD テナントのグローバル管理者ロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。詳細については、[Azure ポータルを使用した Azure のロール no 割り当ての一覧表示](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory での管理者のロールの表示と割り当て](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

### 演習 0: ラボの前提条件を構成する

この演習でを設定します。これには、Azure DevOps DemoGenerator テンプレートに基づく事前構成済みの PartsUnlimited チームプロジェクトと、**Canary**環境と**Production**を表す 2 つの Azure Web アプリが含まれます。これは、Azure Pipelines を介してアプリケーションをデプロイします。

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**Parts Unlimited** テンプレート基づいて新しいプロジェクトを生成します。

1.  ラボ コンピューターで、Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボに必要なコンテンツ (作業項目、リポジトリなど) が事前に入力されたアカウント内に新しい Azure DevOps プロジェクトを作成するプロセスを自動化します。

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」 をクリックし、Azure DevOps サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1.  必要な場合は、**Azure DevOps Demo Generator** ページで、「**承諾する**」 をクリックして、Azure DevOps サブスクリプションにアクセスするためのアクセス許可の要求を受け入れます。
1.  「**Create a new project**」 ページの 「**New Project Name**」 テキストボックスに、「**Controlling Deployments using Release Gates**」と入力し、「**Select organization**」 ドロップダウン リストで、Azure DevOps 組織を選択して、「**Choose template**」 をクリックします。
1.  テンプレートのリストのツールバーで、「**DevOps Labs**」 をクリックし、**ReleaseGates** テンプレートを選択して、「**Select template**」 をクリックします。
1.  「**Create New Project**」 ページで、「**Create Project**」 をクリックします

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**Create New Project**」 ページで、「**Navigate to project**」 をクリックします

#### タスク 2: 2 つの Azure Web アプリを作成する

このタスクでは、**Canary**環境と**Production**環境を表す 2 つの Azure Web アプリを作成し、Azure Pipelines を介してアプリケーションをデプロイします。

1.  ラボ コンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動して、このラボで使用する Azure サブスクリプションで所有者のロールを持ち、このサブスクリプションに関連付けられている Azure AD テナントでグローバル管理者のロールを持つユーザー アカウントでサインインします。
1.  Azure portal で、ページ上部の検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1.  **Bash** または **PowerShell** のいずれかを選択するためのプロンプトが表示されたら、**「Bash」** を選択します。

> **注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」を選択します。

1.  **Bash** プロンプトの 「**Cloud Shell**」 ペインで、次のコマンドを実行してリソースグループを作成します (`<region>` プレースホルダーを 2 つの Azure Web アプリをホストする Azure リージョンの名前に置き換えます)。

    ```bash
    RESOURCEGROUPNAME='az400m10l01-RG'
    az group create -n $RESOURCEGROUPNAME -l '<region>'
    ```

1.  アプリ サービス プランを作成する

    ```bash
    SERVICEPLANNAME='az400m01l01-SP1'
    az appservice plan create -g $RESOURCEGROUPNAME -n 'az400m01l01-sp1' --sku S1
    ```

1.  ユニークなアプリ名を持つ 2 つの Web アプリを作成します。

    ```bash
    SUFFIX=$RANDOM$RANDOM
    az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n PU$SUFFIX-Canary
    az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n PU$SUFFIX-Prod
    ```

1.  **az400m10l01-RG**リソースグループ に移動し、作成したリソースを確認します。
1.  リソースのリストで、**Canary** Web アプリをクリックします。
1.  **Canary** Web アプリ ページの左側の垂直メニューの 「**設定**」 セクションで、「**Application insights**」 をクリックします。
1.  「**Application insights**」 ブレードで、「**Application insights を有効にする**」 をクリックし、デフォルト設定を受け入れ、作成したリソースと同じリージョンに Application insights のリソースを作成し、 「**適用**」をクリックします。

- 確認を求められたら、「**はい**」 をクリックします。

1.  **az400m10l01-RG** リソース グループに戻り、ページ ビューを更新して、作成した Application Insights リソースをクリックします。

    > **注**: ここでモニター アラートを作成します。これは、このラボの後半で使用します。

1.  Application Insights リソースで、「**アラート**」 をクリックし、「**新しいアラート ルール**」 をクリックします。
1.  「**アラートルール の作成**」 ブレードの 「**条件**」 セクションで、「**条件の追加**」 リンクをクリックします。
1.  「**シグナルロジックの構成**」 ブレードの 「**シグナル名で検索**」 テキストボックスに「**Failed Requests**」と入力して選択します。
1.  「**シグナルロジックの構成**」 ブレードの 「**アラート ロジック**」 セクションで、「**しきい値**」 を 「**静的**」 に設定したままにし、「**しきい値**」 テキストボックスに **0**と 入力して、「**完了**」 をクリックします。

    > **注**: ルールは、過去 5 分間に失敗したリクエストの数が 0 を超えると、アラートを生成します。

1.  「**アラート ルールの作成**」 ペインに戻り、「**アラート ルールの詳細**」 で次の設定を指定し、「**アラート ルールの作成**」 をクリックします (他のユーザーはデフォルト値のままにします)。

    | 設定         | 値                                |
    | ------------ | --------------------------------- |
    | 警告ルール名 | **PartsUnlimited_FailedRequests** |
    | 重大度       | **重要度 3**                      |

    > **注**: メトリック アラートをルールがアクティブになるまでに最大 10 分かかる場合があります。

    > **注**: 可用性 < 99%、サーバー応答時間 > 5 秒、サーバー例外 > 0 など、さまざまなメトリックで複数のアラート ルールを作成できます。

### 演習 1: リリース パイプラインを構成する

この演習では、リリース パイプラインを構成します。

#### タスク 1: リリース タスクを更新する

このタスクでは、リリース タスクを更新します。

1.  ラボ コンピューターで、Azure DevOps ポータルの 「**Controlling Deployments using Release Gates**」 プロジェクトを表示するブラウザー ウィンドウに切り替え、垂直ナビゲーション ペインで 「**Pipelines**」 を選択し、「**Pipelines**」 セクションで 「**Releases**」 をクリックします。
1.  「**Releases**」 ビューの 「**PartsUnlimited-CD**」 ペインで、「**Edit**」 をクリックします。

    > **注**: パイプラインには、**Canary Environment** と **Production** という名前の 2 つのステージが含まれています。

1.  「**Artifacts**」 の **PartsUnlimited-CI**ビルド アーティファクトの右上隅にある 「**Continuous deployment trigger**」 ボタン(雷マーク)をクリックします。
1.  **PartsUnlimited-CI** ビルドの継続的デプロイ トリガーが無効になっている場合は、スイッチを切り替えて有効にします。他のすべての設定をデフォルトのままにし、右上隅の **x** マークをクリックして、「**Continuous deployment trigger**」 ペインを閉じます。
1.  **Canary Environments**ステージ内で、**1 job, 2 tasks**のラベルをクリックします。

    > **注**: Canary Environments には 2 つのタスクがあり、それぞれパッケージを Azure Web App に公開し、デプロイ後のアプリケーションの継続的な監視を可能にします。

1.  **「All pipelines」 > 「PartsUnlimited-CD」** ペインで、「**Canary Environments**」 ステージが選択されていることを確認します。「**Azure subscription**」 ドロップダウン リストで、Azure サブスクリプションを選択し、「**Authorize**」 をクリックします。プロンプトが表示されたら、Azure サブスクリプションの所有者ロールを持つユーザー アカウントを使用して認証します。
1.  「**App Service name**」 ドロップダウンリストで、**Canary** Web アプリの名前を選択します。
1.  「**Resource Group and Application Insights**」 ドロップダウン リストで、**az400m10l01-RG** エントリを選択します。
1.  「**Application Insights resource name**」 ドロップダウン リストで、**Canary** Application Insights リソースの名前を選択します。
1.  2021/08/19追記: 「Enable Coninuous Monitoring」を右クリックメニューからDisableにします。
    > **注**: これは一時的なワークアラウンドです。
1.  「**Tasks**」 タブをクリックし、ドロップダウン リストで 「**Production**」 を選択します。
1.  **Production**ステージを選択した状態で、「**Azure subscription**」 ドロップダウ ンリストで、**Canary Environments**ステージに使用した Azure サブスクリプションを選択し、「**Authorize**」 をクリックします。
1.  「**App Service name**」 ドロップダウン リストで、**Prod** Web アプリの名前を選択します。
1.  「**Save**」 をクリックし、「**Save**」 ダイアログ ボックスで 「**OK**」 をクリックします。
1.  「**Pipelines**」 ペインで、**PartsUnlimited-CI** ビルド パイプラインを表すエントリをクリックしてから、「**PartsUnlimited-CI**」 ペインで 「**Run pipeline**」 をクリックします。
1.  「**Run pipeline**」 ペインで、デフォルト設定を受け入れ、「**Run**」 をクリックしてパイプラインをトリガーします。

    > **注**: ビルドが成功すると、リリースが自動的にトリガーされ、アプリケーションが両方の環境にデプロイされます。

1.  垂直ナビゲーション ペインの 「**Pipelines**」 セクションで 「**Releases**」 をクリックし、「**PartsUnlimited-CD**」 ペインで最新のリリースを表すエントリをクリックします。

1.  **PartsUnlimited-CD > Release-1** ブレードで、リリースの進行状況を追跡し、両方の Web アプリへのデプロイが正常に完了したことを確認します。
1.  Azure portal インターフェイスに切り替え、リソース グループ **az400m10l01-RG** に移動し、リソースのリストで**Canary** Web アプリをクリックし、Web アプリ ブレードで 「**参照**」 をクリックして、Web ページが新しい Web ブラウザー タブに正常に読み込まれることを確認します。
1.  **Parts Unlimited** Web サイトを表示している Web ブラウザー タブを閉じます。
1.  Azure portal インターフェイスに切り替え、リソース グループ **az400m10l01-RG** に移動し、リソースのリストで**Production** Web アプリをクリックし、Web アプリ ブレードで 「**参照**」 をクリックして、Web ページが新しい Web ブラウザー タブに正常に読み込まれることを確認します。
1.  **Parts Unlimited** Web サイトを表示している Web ブラウザー タブを閉じます。

    > **注**: これで、CI/CD で構成されたアプリケーションができました。次の演習では、リリース パイプラインにゲートを設定します。

### 演習 2: リリース ゲートを構成します。

この演習では、リリース パイプラインにゲートを設定します。

#### タスク 1: デプロイ前ゲートを構成する

このタスクでは、デプロイ前ゲートを構成します。

1.  Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替え、垂直ナビゲーション ペインの 「**Pipelines**」 セクションで 「**Releases**」 をクリックし、「**PartsUnlimited-CD**」 ペインで 「**Edit**」 をクリックします。
1.  **「All pipelines」 > 「PartsUnlimited-CD」** ペインで、**Canary Environments**ステージを表す長方形の左端で、**Pre-deployment conditions**を表す楕円形をクリックします。
1.  「**Pre-deployment conditions**」 ウィンドウで、「**Pre-deployment approvals**」 スライダーを 「**Enabled**」 に設定し、「**Approvers**」 テキスト ボックスに 自分の Azure DevOps アカウント名を入力して選択します。
1.  「**Pre-deployment conditions**」 ペインで、「**Gates**」 スライダーを 「**Enabled**」 に設定し、「**add**」 をクリックして、ポップアップ メニューで 「**Query Work Items**」 をクリックします。
1.  「**Pre-deployment conditions**」 ペインの 「**Query Work Items**」 セクションの 「**Query**」 ドロップダウン リストで、「**Shared Queries**」 の下の 「**Bugs**」 を選択し、**Upper threshold**の値を **0** のままにします。

    > **注**: **Upper threshold**設定の値に基づいて、この場合クエリはアクティブなバグ作業アイテムを返し、リリース ゲートは失敗します。

1.  「**Pre-deployment conditions**」 ペインで、「**Delay before evaluation**」 設定の値を **5 分**のままにします。

    > **注**: **Delay before evaluation**は、追加されたゲートが最初に評価されるまでの時間を表します。ゲートが追加されていない場合、デプロイは続行する前にその期間待機します。ゲート機能を初期化して安定させるために (正確な結果が返されるまでに時間がかかる場合があります)、結果が評価され、デプロイが承認または拒否されるかどうかを判断するために使用されるまでの遅延を構成します。

1.  「**Pre-deployment conditions**」 ペインで、「**Evaluation options**」 セクションをデプロイし、次のオプションを構成します

    - **Time between re-evaluation of gates**の値を **5 分**に設定します。

    > **注**: **Time between re-evaluation of gates**は、すべてのゲートの各評価間の時間間隔を表します。各サンプリング間隔で、新しい要求が各ゲートに同時に送信され、新しい結果が得られます。サンプリング間隔は、すべての応答を受信できるように、構成されたゲートの通常の最長応答時間よりも長くする必要があります。

    - **Timeout after which gates fail**の値を **8 分**に設定します。

    > **注**: **Timeout after which gates fail**は、すべてのゲートの最大評価期間を表します。同じサンプリング間隔ですべてのゲートが成功する前にタイムアウトに達すると、デプロイは拒否されます。タイムアウトに指定できる最小値は、サンプリング間隔として 6 分と 5 分です。

    > **注**: この場合、リリースがトリガーされると、ゲートは *0 分*と *5 分*にサンプルを検証します。結果が**合格**の場合、承認のために通知が送信されます。結果が**失敗**の場合、リリースは _8_ 分後にタイムアウトになります。

    > **注**: 実際には、これらの値は複数時間にわたる場合があります。

1.  「**Pre-deployment conditions**」 ペインで、「**On successful gates, ask for approvals**」 ラジオ ボタンを選択します。
1.  右上隅の **x** マークをクリックして、「**Pre-deployment conditions**」 ペインを閉じます。

1.  **Query Work Items** ゲートが機能するには、**Project Build Service**に Azure Board クエリの読み取りアクセス許可が必要です。
1.  Azure DevOps ポータルの垂直ナビゲーション ペインで、マウス ポインターを 「**Boards**」 に合わせ、**Ctrl** キーを押しながら 「**Query**」 をクリックして、「**Query**」 ペインで別のブラウザー タブを開きます。
1.  **Boards** ビューの 「**Query**」 ペインで、「**All**」 をクリックして、すべてのクエリのリストを取得します。
1.  「**Shared Queries**」 フォルダーを右クリックし、「**Security...**」 を選択して、「**Permissions for Shared Queries**」 ペインを開きます。
1.  「**Permissions for Shared Queries**」 ペインの 「**Search for users or groups**」 フィールドに、「**Project 名 Build Service (Org 名)**」を入力し、見つかった ID をクリックします。

    > プロジェクト名が「**Controlling Deployments using Release Gates**」の場合は「**Controlling Deployments using Release Gates Build Service (Org 名)**」です。

    > ユーザーの **Project Collection Build Service** と **Project Build Service** を混同しないでください。

1.  右側のサイトで 「**Read**」 アクセス許可を 「**Allow**」 に設定します。
1.  右上隅の **x** マークをクリックして、「**Permissions for Shared Queries**」 ペインを閉じます。
1.  リリース パイプラインがまだ開いているブラウザー タブに戻ります。

#### タスク 2: デプロイ後のゲートを構成する

このタスクでは、Canary Environments のデプロイ後のゲートを有効にします。

1.  **「All pipelines」 > 「PartsUnlimited-CD」** ペインに戻り、**Canary Environments**ステージを表す長方形の右端で、**Post-deployment conditions**を表す楕円形をクリックします。
1.  「**Post-deployment conditions**」 ペインで、「**Gates**」 スライダーを 「**Enabled**」 に設定し、「**add**」 をクリックして、ポップアップ メニューで 「**Query Azure Monitor Alerts**」 をクリックします。
1.  「**Post-deployment conditions**」 ペインの 「**Query Azure Monitor Alerts**」 セクションの 「**Azure subscription**」 ドロップダウン リストで、Azure サブスクリプションを表すエントリを選択し、「**Resource group**」 ドロップダウン リストで、**az400m10l01-RG** エントリを選択します。
1.  「**Post-deployment conditions**」 ペインで、「**Evaluation options**」を開き、次のオプションを構成します。

- **Time between re-evaluation of gates**の値を **5 分**に設定します。
- **Timeout after which gates fail**の値を **8 分**に設定します。
- 「**On successful gates, ask for approvals**」 オプションを選択します。

  > **注**: サンプリング間隔とタイムアウトは連携して機能するため、ゲートは適切な間隔で関数を呼び出し、タイムアウト期間内の同じサンプリング間隔で成功しなかった場合はデプロイを拒否します。

1.  右上隅の **x** マークをクリックして、「**Post-deployment conditions**」 ペインを閉じます。
1.  **PartsUnlimited-CD** ペインに戻り、「**Save**」 をクリックし、「**Save**」 ダイアログボックスでもう一度 「**Save**」 をクリックします。

## 演習 3: リリース ゲートをテストする

この演習では、アプリケーションを更新してリリース ゲートをテストします。これにより、デプロイがトリガーされます。

#### タスク 1: リリース ゲートを追加した後、アプリケーションを更新しデプロイする

このタスクでは、アプリケーション コードに小さな変更を加え、リポジトリに更新をコミットし、ビルドとリリースのプロセスを追跡します。

1. パイプライン作成画面の上方にある、「**Create relase**」から、パイプラインを起動します。
1. 「**Release-2**」 エントリをクリックして、**Canary Environments**へのデプロイの進行状況を確認します。
1. **Canary Environments**ステージを表す長方形の左端にある**Pre-deployment conditions**の状態を表す楕円形をクリックします。この時点で、**ゲートの評価**または**デプロイ前のゲートが失敗した**というラベルが付いている可能性があります。
1. 「**Canary Environments**」 ペインで、**Query Work Items** ゲートが失敗したことに注意してください。

   > **注**: これは、アクティブな作業項目があることを示しています。先に進むには、これらの作業項目を閉じる必要があります。次のサンプリング時間は 5 分後です。

1. 新しいブラウザー タブを開き、Azure DevOps ポータルに移動し、垂直ナビゲーション ペインで 「**Boards**」 を選択し、「**Boards**」 セクションで 「**Query**」 を選択します。
1. **Boards** ビューの 「**Query**」 ペインで、「**All**」 タブをクリックします。
1. 「**Query**」 ペインの 「**All**」 タブの 「**Shared Queries**」 セクションで、「**Bugs**」 エントリをクリックし、**「Queries」 > 「Shared Queries」 > 「Bugs」** ペインで、「**Run query**」 をクリックします。
1. クエリが、**New**状態の**Disk out of space in Canary Environment**というタイトルの作業項目を返すことを確認します。

   > **注**: インフラストラクチャ チームがディスク容量の問題を修正したと仮定しましょう。

1. 「**Disk out of space in Canary Environment**」 エントリをクリックします。
1. 「**Disk out of space in Canary Environment**」 ペインで、左上隅の 「**State**」 ラベルの横にある 「**New**」 をクリックし、ドロップダウン リストで 「**Close**」 をクリックして、「**Save**」 をクリックします。
1. 「**Canary Environments**」 ペインに戻り、2 番目の評価が合格するのを待ちます。

   > **注**: 2 番目の評価がすでに失敗した場合は、**Canary Environments**ステージを表す長方形の上にマウスポインターを置いて 「**Redeploy**」 オプションを表示し、「**Redeploy**」 をクリックし、「**Canary Environments**」 ブレードで、「**Deploy**」 をクリックし、デプロイ前のゲートの処理ステータスを監視します。

   > **注**: 評価が成功すると、Pre-deployment approvals のリクエストが表示されます。

1. Canary Environments にデプロイするには、「**Approve**」 をクリックします。

   > **注**: 成功すると、Application Insights を使用して、新しくデプロイされたアプリケーションを対象とする失敗したリクエストの存在を検出するデプロイ後のゲートが動作します。

1. Post deployment プロセスを失敗させるために、Azure portal を表示している Web ブラウザー ウィンドウに切り替え、**Canary** Azure Web アプリ ブレードに移動して、「**参照**」 をクリックします。これにより、PartsUnlimited Web サイトを表示する新しいブラウザー タブが開きます。
1. PartsUnlimited Web サイトで、「**More**」 をクリックします。

   > **注**: Web サイトのこの部分は意図的に誤って構成されているため、失敗した要求がトリガーされます。

1. PartsUnlimited Web サイトのホームページに戻り、もう一度 「**More**」 をクリックして、この手順をさらに数回繰り返します。
1. **Canary** Web アプリ ページの Application Insights ブレードに移動して、失敗した要求が Application Insights によって検出されたことを検証し、Application Insights ブレードで 「**アラート**」 をクリックして、ページに 1 つ以上の **Sev3** アラートがリストされていることを確認します。

   > **注**: 例外によってトリガーされるアラートがあるため、**Query AzureMonitor** ゲートは失敗します。これにより、**Production**環境への展開が妨げられます。

### 演習 4: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングされた Azure リソースを削除して、予期しない料金を排除します。

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m10l01-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m10l01-RG')].「name」" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、リリース パイプラインを構成してから、リリース ゲートを構成してテストしました。
