---
lab:
    title: 'ラボ 00: ラボ環境を検証する'
    module: 'モジュール 0: ようこそ'
---

# ラボ 00: ラボ環境を検証する
# 受講生用ラボ マニュアル

## 手順

1. 講師またはその他のソースから、新しい **Azure Pass プロモーションコード** (30 日間有効) を入手します。
2. プライベート ブラウザー セッションを使用して、https://account.microsoft.com で新しい **Microsoft アカウント (MSA)** を取得するか、既存のアカウントを使用します。
3. 同じブラウザー セッションを使用して、https://www.microsoftazurepass.com にアクセスし、Microsoft アカウント (MSA) を使用して Azure Pass を利用します。詳細については、「[Microsoft　Azure　パスの引き換え](https://www.microsoftazurepass.com/Home/HowTo?Length=5)」を参照してください。引き換えの手順に従います。 
4. 同じブラウザー セッションを使用して、https://portal.azure.com にアクセスし、ポータル画面の上部で **Azure DevOps** を検索します。表示されたページで、「**Azure DevOps organization**」をクリックします。 
5. 次に、**My Azure DevOps Organizations** というラベルの付いたリンクをクリックします（または https://aex.dev.azure.com に直接移動します)。
6. 左側のドロップダウン ボックスで、「Microsoft アカウント」 の代わりに「**既定のディレクトリ**」を選択します。
7. *「We need a few more details」* が表示されたら、名前メールアドレス、場所を入力して、「**Continue**」 をクリックします。
8. 「**既定のディレクトリ**」 を選択した状態で https://aex.dev.azure.com に戻り、青いボタン 「**新しい組織の作成**」 をクリックします。
9. 「**Continue**」 をクリックして*利用規約*に同意します。
10. *「Almost done...」* が表示されたら、Azure DevOps　組織の名前を既定のままにし (グローバルに一意の名前である必要があります)、一覧から最寄りのホスティング場所を選択します。
11. 新しく作成した組織が **Azure DevOps** で開いたら、左下隅にある 「**組織の設定**」 をクリックします。
12. 「**Organization settings**」 画面で、「**Billing**」 をクリックします (この画面を開くには数秒かかります)。
13. 「**Set up billing**」 をクリックし、画面の右側で **「Azure Pass - スポンサーシップサブスクリプション」** を選択し、「**Save**」 をクリックしてサブスクリプションを組織にリンクします。
14. 画面の上部にリンクされた Azure サブスクリプション ID が表示されたら、**MS Hosted CI/CD** の**Paid parallel jobs**の数を 0 から **1** に変更します。次に、下部にある 「**Save**」 をクリックします。 
15. 新しい設定がバックエンドに反映されるように、**CI/CD 機能を使用する前に少なくとも 3 時間待ちます**。それ以外の場合は、*「This agent is not running because you have reached the maximum number of requests…」* というメッセージが表示されます。
16. オプション: ビルド パイプラインを作成してトリガーすることで、この新しい設定が正常に適用されたことを検証できます。これを行うには、https://azuredevopsdemogenerator.azurewebsites.net を使用して、講師に相談するか、課金を有効にして新しく作成した組織にデモプロジェクトを作成します。
