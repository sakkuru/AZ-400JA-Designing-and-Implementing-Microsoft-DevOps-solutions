---
lab:
  title: "ラボ: Azure Key Vault と Azure DevOps の統合"
  module: "モジュール 7: アプリケーション構成とシークレットの管理"
---

# ラボ: Azure Key Vault と Azure DevOps の統合

# 学生用ラボ マニュアル

## ラボの概要

Azure Key Vault は、キー、パスワード、証明書などの機密データの安全な保管と管理を行います。Azure Key Vault には、ハードウェア セキュリティ モジュールのサポートに加えて、さまざまな暗号化アルゴリズムとキーの長さが含まれています。Azure Key Vault を使用することで、開発者がよく犯す間違いであるソース コードを介して機密データを開示する可能性を最小限に抑えることができます。Azure Key Vault にアクセスするには、コンテンツに対するきめ細かいアクセス許可をサポートする適切な認証と承認が必要です。

このラボでは、次の手順を使用して、Azure KeyVault を Azure DevOps パイプラインと統合する方法を説明します。

- MySQL サーバーのパスワードをシークレットとして保存する Azure Key Vault を作成します。
- Azure Key Vault 内のシークレットへのアクセスを提供する Azure サービス プリンシパルを作成します。
- サービス プリンシパルがシークレットを読み取れるようにする権限を構成します。
- Azure Key Vault からパスワードを取得し、それを後続のタスクに渡すようにパイプラインを構成します。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure Active Directory (Azure AD) サービス プリンシパルを作成します。
- Azure Key Vault を作成します。
- Azure DevOps パイプラインを介して Pull request を追跡します。

## ラボの所要時間

- 推定時間: **40 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。

- ユーザー名: **Student**
- パスワード: **Pa55w.rd**

#### このラボに必要なアプリケーションを確認する

このラボで使用するアプリケーションを特定する:

- Microsoft Edge

#### Azure サブスクリプションを準備する

- 既存の Azure サブスクリプションを特定するか、新しいサブスクリプションを作成します。
- Azure サブスクリプションの所有者のロールと Azure サブスクリプションに関連付けられた Azure AD テナントのグローバル管理者ロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。詳細については、[Azure ポータルを使用した Azure のロール no 割り当ての一覧表示](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory での管理者のロールの表示と割り当て](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

#### Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)の手順に従って作成してください。

### 演習 0: ラボの前提条件を構成する

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートに基づいて事前構成された Parts Unlimited チームプロジェクトで構成されます。

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**Azure Key Vault** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボ コンピューターで、Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボに必要なコンテンツ (作業項目、リポジトリなど) が事前に入力されたアカウント内に新しい Azure DevOps プロジェクトを作成するプロセスを自動化します。

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」 をクリックし、Azure DevOps サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1.  必要な場合は、**Azure DevOps Demo Generator** ページで、「**承諾する**」 をクリックして、Azure DevOps サブスクリプションにアクセスするためのアクセス許可の要求を受け入れます。
1.  「**Create a new project**」 ページの 「**New Project Name**」 テキストボックスに、「**Integrating Azure Key Vault with Azure DevOps**」と入力し、「**Select organization**」 ドロップダウンリストで、Azure DevOps 組織を選択して、「**Choose template**」 をクリックします。
1.  「**Choose a template**」 ページのヘッダー メニューで、「**DevOps Labs**」 クリックし、テンプレートのリストで 「**Azure Key Vault**」 テンプレート をクリックしてから、「**Select template**」 をクリックします。
1.  「**Create a new project**」 ページに戻り、「**ARM Outputs**」 ラベルの下のチェックボックスを選択して、「**Create Project**」 をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**Create New Project**」 ページで、「**Navigate to project**」 をクリックします

### 演習 1: Azure Key Vault を Azure DevOps と統合する

- Azure Key Vault 内のシークレットへのアクセスを提供する Azure サービス プリンシパルを作成します。
- MySQL サーバーのパスワードをシークレットとして保存する AzureKey Vault を作成します。
- サービス プリンシパルがシークレットを読み取れるようにする権限を構成します。
- Azure Key Vault からパスワードを取得し、それを後続のタスクに渡すようにパイプラインを構成します。

#### タスク 1: サービス プリンシパルの作成

このタスクでは、Azure CLI を使用してサービス プリンシパルを作成します。

> **注**: サービス プリンシパルが既にある場合は、次のタスクに直接進むことができます。

Azure Pipelines から Azure リソースにアプリをデプロイするには、サービス プリンシパルが必要です。パイプラインでシークレットを取得するため、Azure Key Vault を作成するときにサービスにアクセス許可を付与する必要があります。

サービス プリンシパルは、パイプライン定義内から Azure サブスクリプションに接続するとき、またはプロジェクト設定ページから新しいサービス接続を作成するときに、Azure Pipeline によって自動的に作成されます。ポータルから、または Azure CLI を使用してサービス プリンシパルを手動で作成し、プロジェクト間で再利用することもできます。事前定義された権限のセットが必要な場合は、既存のサービス プリンシパルを使用することをお勧めします。

1.  ラボ コンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動して、このラボで使用する Azure サブスクリプションで所有者のロールを持ち、このサブスクリプションに関連付けられている Azure AD テナントでグローバル管理者のロールを持つユーザー アカウントでサインインします。
1.  Azure portal で、ページ上部の検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1.  **Bash** または **PowerShell** のいずれかを選択するためのプロンプトが表示されたら、「**Bash**」 を選択します。

> **注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」を選択します。

1.  Bash プロンプトの Cloud Shell ペインで、次のコマンドを実行してサービス プリンシパルを作成します (`<service-principal-name>` を文字と数字で構成される一意の文字列に置き換えます)。

    ```
    az ad sp create-for-rbac --name <service-principal-name>
    ```

    > **注**: このコマンドは JSON 出力を生成します。出力をテキスト ファイルにコピーします。このラボの後半で必要になります。

1.  **Bash** プロンプトの 「**Cloud Shell**」 ペインで、次のコマンドを実行して、Azure サブスクリプション ID とサブスクリプション名の属性の値を取得します。

    ```
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **注**: 両方の値をテキスト ファイルにコピーします。これらは、このラボの後半で必要になります。

#### タスク 2: Azure Key Vault を作成する

このタスクでは、Azure portal を使用して Azure Key Vault を作成します。

このラボ シナリオでは、MySQL データベースに接続するアプリがあります。MySQL データベースのパスワードをシークレットとして Key Vault に保存します。

1.  Azure portal の 「**リソース、サービス、ドキュメントの検索**」 テキスト ボックスに「**Key Vault**」と入力し、**Enter** キーを押します。
1.  **Key Vault** ブレードで、「**+ 追加**」 をクリックします。
1.  「**Key Vault の作成**」 ブレードの 「**基本**」 タブで、次の設定を指定し、「**次へ:**」 をクリックします。**アクセス ポリシー**:

    | 設定                             | 値                                                |
    | -------------------------------- | ------------------------------------------------- |
    | サブスクリプション               | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ                | 新しいリソース グループの名前 **az400m07l01-RG**  |
    | Key Vault 名                     | 任意の一意の有効な名前                            |
    | リージョン                       | ラボ環境の場所に近い Azure リージョン             |
    | 価格レベル                       | **Standard**                                      |
    | 削除されたボールトを保持する日数 | **7**                                             |
    | 消去保護                         | **消去保護の無効化**                              |

1.  「**キーボールトの作成**」 ブレードの 「**アクセス ポリシー**」 タブで、「**+ アクセスポリシーの追加**」 をクリックして、新しいポリシーを設定します。

    > **注**: 許可されたアプリケーションとユーザーのみを許可することにより、Key Vault のアクセスを保護する必要があります。ボールトからデータにアクセスするには、パイプラインでの認証に使用するサービス プリンシパルに読み取り (取得) アクセス許可を提供する必要があります。

1.  「**アクセスポリシーの追加**」 ブレードで、「**プリンシパルの選択**」 ラベルのすぐ下にある 「**選択されていません**」 リンクをクリックします。
1.  **プリンシパル** ブレードで、前の演習で作成したセキュリティ プリンシパルを検索して選択し、「**選択**」 をクリックします。

    > **注**: プリンシパルの名前または ID で検索できます。

1.  「**アクセス ポリシーの追加**」 ブレードに戻り、「**シークレット アクセス許可**」 ドロップダウン リストで、「**取得**」 および 「**一覧**」 アクセス許可の横にあるチェックボックスを選択し、「**追加**」 をクリックします。
1.  「**Key Vault の作成**」 ブレードの 「**アクセス ポリシー**」 タブに戻り、「**確認および作成**」 をクリックし、「**確認および作成**」 ブレードで 「**作成**」 をクリックします。

    > **注**: Azure Key Vault がプロビジョニングされるのを待ちます。これは 1 分もかかりません。

1.  「**デプロイが完了しました**」 ブレードで、「**リソースに移動**」 をクリックします。
1.  Azure Key Vault ブレードのブレードの左側にある垂直メニューの 「**設定**」 セクションで、「**シークレット**」 をクリックします。
1.  「**シークレット**」 ブレードで、「**生成/インポート**」 をクリックします。
1.  「**シークレットの作成**」 で、次の設定を指定し、「**作成**」 をクリックします (他の設定はデフォルト値のままにします)。

    | 設定           | 値                                       |
    | -------------- | ---------------------------------------- |
    | Upload options | **手動**                                 |
    | 名前           | **sqldbpassword**                        |
    | 値             | 任意のパスワード値(8 文字以上の複雑な値) |

#### タスク 3: Azure Pipelines を確認する

このタスクでは、Azure Key Vault からシークレットを取得するように Azure Pipelines を構成します。

1.  ラボ コンピューターで、Web ブラウザーを起動し、前の演習で作成した **Integrating Azure Key Vault with Azure DevOps**プロジェクトに移動します。
1.  Azure DevOps ポータルの垂直ナビゲーション ペインで、「**Pipelines**」 を選択し、「**Pipelines**」 ペインが表示されていることを確認します。
1.  「**Pipelines**」 ペインで、**SmartHotel-CouponManagement-CI** パイプラインを表すエントリをクリックし、「**SmartHotel-CouponManagement-CI**」 ペインで 「**Run pipeline**」 をクリックします。
1.  「**Run pipeline**」 ペインで、デフォルト設定を受け入れ、「**Runs**」 をクリックしてビルドをトリガーします。
1.  Azure DevOps ポータルの垂直ナビゲーション ペインの 「**Pipelines**」 セクションで、「**Releases**」 を選択します。
1.  **SmartHotel-CouponManagement-CD** ペインで、右上隅にある 「**Edit**」 をクリックします。
1.  「**All pipelines」 > 「SmartHotel-CouponManagement-CD**」 ペインで、「**Tasks**」 タブを選択し、ドロップダウン メニューで 「**Dev**」 を選択します。

    > **注**: **Dev**ステージのリリース定義には、**Azure Key Vault** タスクがあります。このタスクは、Azure KeyVault から*シークレット*をダウンロードします。ラボで以前に作成したサブスクリプションと Azure Key Vault リソースを指定する必要があります。

    > **注**: Azure にデプロイするには、パイプラインを承認する必要があります。Azure Pipelines は、新しいサービス プリンシパルとのサービス接続を自動的に作成できますが、以前に作成したものを使用したいと思います。

1.  **Azure Key Vault** タスクを選択し、右側の **Azure Key Vault** タスクのプロパティで、**Azure subscription** ラベルの横にある 「**Manage**」 をクリックします。
    これにより、Azure DevOps ポータルの 「**Service connections**」 ペインを表示する別のブラウザー タブが開きます。
1.  「**Service connections**」 ペインで、「**New Service connection**」 をクリックします。
1.  「**New Service connection**」 ウィンドウで、「**Azure Resource Manager**」 オプションを選択し、「**Next**」 をクリックして、「**Service Principal (manual)**」 を選択し、もう一度 「**Next**」 をクリックします。
1.  「**New Service connection**」 ペインで、Azure CLI を使用してサービス プリンシパルを作成した後、この演習の最初のタスクでテキスト ファイルにコピーした情報を使用して、次の設定を指定します。

    - サブスクリプション ID: `az account show --query id --output tsv` を実行することにより取得される値
    - サブスクリプション名: `az account show --query name --output tsv` を実行することにより取得される値
    - サービス プリンシパル ID: `az ad sp create-for-rbac --name <service-principal-name>` を実行することにより生成される出力の **appId** のラベルが付いた値
    - サービス プリンシパル Key: `az ad sp create-for-rbac --name <service-principal-name>` を実行することにより生成される出力の **password** のラベルが付いた値
    - Tenant Id: `az ad sp create-for-rbac --name <service-principal-name>` を実行することにより生成される出力の **tenant** のラベルが付いた値

1.  「**New service connection**」 ペインで、「**Verify**」をクリックして、指定した情報が有効かどうかを確認します。
1.  「**Verification Succeeded**」という応答を受け取ったら、「**Service connection name**」 テキストボックスに「**kv-service-connection**」と入力し、「**Verify and Save**」 をクリックします。
1.  **Azure Key Vault** タスクを表示している Web ブラウザー タブに戻ります。
1.  **Azure Key Vault** タスクを選択した状態で、「**Azure Key Vault**」 ペインの 「**Refresh**」 ボタンをクリックし、「**Azure subscription**」 ドロップダウン リストで **kv-service-connection** エントリを選択し、「**Key Vault**」 ドロップダウン リストで、最初のタスクで作成した Azure Key Vault を表すエントリを選択し、「**Secrets filter**」 テキストボックスに「**sqldbpassword**」と入力します。最後に、「**Output Variables**」 セクションを展開し、「**Reference name**」 テキストボックスに「**sqldbpassword**」と入力します。

    > **注**: 実行時に、Azure Pipelines はシークレットの最新の値をフェッチし、それをタスク変数 **$(sqldbpassword)** として設定します。タスクは、その変数を参照することにより、後続のタスクによって消費される可能性があります。

1.  これを確認するには、次のタスクである **Azure Deployment** を選択します。このタスクは、ARM テンプレートをデプロイし、「**Override template parameters**」 テキスト ボックスの内容を確認します。

    ```
    -webAppName $(webappName) -mySQLAdminLoginName "azureuser" -mySQLAdminLoginPassword $(sqldbpassword)
    ```

    > **注**: **Override template parameters**のコンテンツは、**sqldbpassword** 変数を参照して mySQL 管理者パスワードを設定します。これにより、キーボールトで指定したパスワードを使用して、ARM テンプレートで定義された MySQL データベースがプロビジョニングされます。

    > **注**: タスクのサブスクリプションと場所を指定することで、パイプライン定義を完了することができます。パイプライン **Azure App Service Deploy** の最後のタスクについても同じことを繰り返します。最後に、新しいリリースを保存して作成し、デプロイを開始します。

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングされた Azure リソースを削除して、予期しない料金を排除します。

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m07l01-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m07l01-RG')].「name」" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

#### レビュー

このラボでは、次の手順を使用して、Azure KeyVault を Azure DevOps パイプラインと統合しました。

- MySQL サーバーのパスワードをシークレットとして格納する Azure Key Vault を作成しました。
- Azure Key Vault 内のシークレットへのアクセスを提供する Azure サービス プリンシパルを作成しました。
- サービス プリンシパルがシークレットを読み取れるようにアクセス許可を構成しました。
- Azure Key Vault からパスワードを取得し、それを後続のタスクに渡すようにパイプラインを構成しました。
