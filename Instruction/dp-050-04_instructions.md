# DP 050 – SQL Workloads から Azure への移行

## ラボ 4 - SQL Workloads を Azure SQL Database に移行する

**推定時間:** 60 分

**前提条件:** このラボでは、実行する前提条件の手順はありません。

**ラボ ファイル:** このラボにはラボ ファイルがありません

## ラボの概要

このラボでは、Azure SQL Database への移行を実行します。まず、スキーマ移行を実行し、前提条件として宛先データベースを作成し、次にオンプレミスの SQL インスタンスで実行されているデータベースから Azure 上の SQL Database に移行します。Database Migration Service (DMS) を使用してオンライン移行を実行し、新しいデータベースに切り替わるまで、ソース データベースとターゲット データベースの間でデータの同期を維持します。

## ラボの目的

このラボを修了すると、次のことができるようになります。

- Azure Cloud Shell を使用して Azure SQL Database でデータベースを作成する
- Azure Data Migration Service の構成。
- データベース スキーマを Azure SQL データベースに移行する。
- Data Migration Service を使用したオンライン移行の実行。

## シナリオ

あなたは、Adventureworks 社の上級データベース管理リーダーであり、データ モダナイゼイション プロジェクトを実行する準備をしています。一連のデータベースを、Azure 仮想マシン（VM）内の SQL Server に移行するために必要な環境を準備し、Data Migration Assistant を使用してテスト移行を実行することになります。

## 演習 1: Azure Cloud Shell を使用して Azure SQL Database でデータベースを作成する

この演習では、Azure Cloud Shell を使用して次を行います。

- 新しいリソース グループを作成します。
- 新しい Azure SQL Database サーバーを作成する。
- Azure SQL SQL Database Server ファイアウォールを構成する。
- 新しい汎用データベースを作成する。

**推定時間:** 20 分

Azure には、サービスのインストール、管理、および展開を自動化するための多くのオプションがあります。オプションの 1 つは、軽量のクロスプラットフォーム コマンド ライン ツールである Azure CLI コマンド ライン ツールをインストールすることです。もう 1 つのオプションは、Azure Cloud Shell を使用して自動化しスクリプトを作成することです。

Azure Cloud Shell は、Azure でホストされ、ブラウザーを介して管理される対話型シェル環境です。クラウド シェルを使用すると、Bash または PowerShell を使用して Azure サービスを操作できます。ローカル環境に何もインストールしなくても、Cloud Shell にプレインストールされているコマンドを使用して、この記事のコードを実行できます。

### タスク 1: ログオンして Azure Cloud Shell を構成する

> [!注]
> このタスクは、Azure CloudShell で完了することができます。

1. [Azure Shell](https://shell.azure.com)に移動する
1. このトレーニングに使用した資格情報を使用して、Azure サブスクリプションにログオンします。
1. スクリプト環境として **Bash** を選択する。

### タスク 2: 新しいストレージ アカウントを作成し、Azure Cloud Shell と共有する

1. Azure Cloud Shell にストレージ アカウントが作成されていないことを確認するメッセージが表示された場合は、**Show Advanced Settings** をクリックし、次の値を入力し、「**ストレージの作成**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションを選択する |
    | Cloud Shell リージョン | 地理的な場所に最も近い利用可能な Cloud Shell リージョンを選択する |
    | リソース グループ | **dp050lab4rg** という名前で新しいリソース グループを作成します。 |
    | ストレージ アカウント | **dp050sa&lt;youridentifier&gt;** という名前の新しいストレージ アカウントを作成します。**&lt;youridentifier&gt;** は、一意の文字列となります。 |
    | ファイル共有 | **dp050share&lt;youridentifier&gt;** という名前の新しいファイル シェアを作成します。 **&lt;youridentifier&gt;** は、一意の名前となります。 |

1. ストレージ アカウントが作成されると、Azure CloudShell が開きます。

### タスク 3: Azure SQL データベース サーバー インスタンスとデータベースを作成します

1. **{}** アイコンを選択して、コード エディタを開きます。
1. このスクリプトをコード エディタに貼り付けます。

    ```bash
    #新しい sql db サーバー インスタンスと sql db を作成する bash スクリプト  
    
    #Disclaimer：Azure Database を作成する方法に関するサンプル スクリプトです。ラボの目的で最も制限の少ないファイアウォール設定を使用します。このスクリプトを使用しない  
    
    #リソース グループの名前を定義する
    resourcegroup=dp-050-labresourcegroup
    
    #一意のサーバー名を指定するには、次の変数を編集する
    servername=dp-050-servername
    
    #次の変数を編集して場所を提供する。 azure locations は、シェル コマンド インターフェイスで az account list-locations -o table と入力して一覧表示できる
    location=eastus
    
    adminuser=sqladmin
    password=Pa55w.rdPa55w.rd
    firewallrule=dp-050-access
    
    #一意のサーバー名を指定するにはスクリプトを編集する
    labdatabase=dp-050-Adventureworksxxx
    
    #リソース グループを作成する
    az group create --name $resourcegroup --location $location
    
    #sql db servername を作成する
    az sql server create --name $servername --resource-group $resourcegroup --admin-user $adminuser --admin-password $password --location $location
    
    #現在のファイアウォールリストを表示する
    az sql server firewall-rule list --resource-group $resourcegroup --server $servername
    
    #サーバー ベースのファイアウォールを作成する。注意：環境に基づいて開始/終了 IP 範囲を制限する必要があります。
    az sql server firewall-rule create --resource-group $resourcegroup --server $servername --name $firewallrule --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
    
    #汎用 SQL Database を作成する
    az sql db create --name $labdatabase --resource-group $resourcegroup --server $servername -e GeneralPurpose
    ```

1. スクリプトで、値 `dp-050-servername` を `dp-050-serverxxx` などの一意のサーバー名に変更します。この名前には大文字を使用しないでください。
1. スクリプトで、値 `eastus` を近くの Azure の場所に変更します。
1. スクリプトで、値 `dp-050-Adventureworksxxx` を一意のデータベース名に変更します。
1. スクリプトを保存するには、<kbd>CTRL + S</kbd> を押し、**CreateSQLDBandServer.sh** と入力して、「**保存**」 を選択します。
1. コード エディタを終了するには、<kbd>CTRL + Q</kbd> を押します。
1. スクリプトを実行するには、**sh CreateSQLDBandServer.sh** と入力し、<kbd>Enter</kbd> を押します。

    スクリプトが正常に完了すると、リソース グループ、サーバー、およびデータベースが Azure アカウント サブスクリプションで作成されたことを検証することができます。

## 演習 2: データベースを Azure SQL Database に移行する

この演習では、オンプレミスで実行されている SQL Server インスタンスのデータベースのデータベース スキーマを移行し、スキーマを以前の演習で作成した SQL Database に読み込みます。データベース スキーマのソース SQL Server インスタンスは、VMLON-DEV-01 で実行されます。

このラボを修了すると、次のことができるようになります。

- Data Migration Assistant を使用した新しい移行プロジェクトの作成。
- データベースのスキーマの移行。

**推定時間:** 20 分

> [!注]
> この演習は、LON-DEV-01VM で実行されている Data Migration Assistant を使用して完了します。

### タスク 1: Data Migration Assistant を使用した移行プロジェクトの作成

1. 教室環境で実行されている **LON-DEV-01** 仮想マシンにサインインします。ユーザー名は **administrator**、パスワードは **Pa55w.rd** です。
1. 新しいデータ移行プロジェクトを作成するには、**Microsoft Data Migration Assistant** を開き、次に **+** を開きます。
1. 「**新規**」 ページで、次の値を入力し、「**作成**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | プロジェクト タイプ | 移行 |
    | プロジェクト名 | SQLSchemaMigration |
    | ソース サーバー名 | SQL Server |
    | ターゲット サーバー名 | Azure SQL Database |
    | 移行のスコープ | スキーマのみ |

### タスク 2: スキーマ移行を実行する

1. 「**ソースの選択**」 ページの 「**ソース サーバーに接続**」 で、次の値を入力し、「**接続**」を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | サーバー名 | localhost |
    | 認証タイプ | Windows 認証 |
    | 暗号化接続 | いいえ |
    | 「サーバー証明書を信頼する」 | はい |

1. データベースのリストで、**AdventureworksLT2008R2** を選択し、「**移行前にデータベースを評価する**」 チェックボックスの選択を解除します。
1. **「次へ」** をクリックします。
1. 「**ターゲットの選択**」 ページの 「**ターゲット サーバーに接続**」 で、次の値を入力し、「**接続**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | サーバー名 | 前の演習で作成したサーバーの**サーバー名**を入力します。**full qualified name** で、サーバーをリストする (dp-050-servername.database.windows.netなど) |
    | 認証タイプ | SQL Server の認証 |
    | ユーザー名 | sqladmin |
    | パスワード | Pas55w.rdPa55w.rd |
    | 暗号化接続 | いいえ |
    | 「サーバー証明書を信頼する」 | はい |

1. データベースのリストで、前の演習で作成したターゲット データベースを選択し、「**次へ**」 を選択します
1. すべてのスキーマ オブジェクトを確認し、「**SQL スクリプトの生成**」 を選択します

    Data Migration Assistant ウィンドウは次のようになります。

    ![Data Migration Assistant Window](/images/dbmigrate.png)

1. 「**スキーマをデプロイする**」 を選択する。
1. 展開が完了したら、Data Migration Assistant を閉じます。

## 演習 3: オンプレミス データベースを Azure SQL Database に移行する

この演習では、Azure Data Migration を使用してデータベースのオンライン移行を実行します。

この演習を修了すると、次のことができるようになります。

- レプリケーションのソース インスタンスを構成し、移行の前提条件が満たされていることを確認する。
- Azure Data Migration Service を使用してライブ マイグレーションを実行する。

**推定時間:** 20 分

### タスク 1: SQL Server をレプリケーション ディストリビュータとして構成する

このタスクは、SQL 2008 R2 仮想マシンの SQL Management Studio を使用して完了しますが、 **SQL Server 2017 instance.** に接続されます。

> [!注]
> ホストされている SQL 2008 R2 VM と Azure 間の VPN アクセスを構成する必要がないように、SQL Server 2017 インスタンスに接続している間にこれらのタスクを LON-DEV-01 上で実行します。ラボ以外の環境では、通常、VPN または ExpressRoute を設定します。

1. SQL Management Studio Object Explorer で、ラボ 3 で作成した SQL Server 2017 インスタンスに接続します。

    SQL Server 2017 インスタンスが登録されていない場合は、「**接続/データベース エンジン**」 を選択し、次の値を使用します。

    | プロパティ | 値 |
    | --- | --- |
    | サーバー名 | SQL 2017 VM の IP アドレスまたは完全修飾ドメイン名（例：sql2017vmxxxx.centralus.cloudapp.azure.com） |
    | Authentication | SQL Server の認証 |
    | ログイン | sqladmin |
    | パスワード | Pa55w.rdPa55w.rd |

1. **オブジェクト エクスプローラー**で、「**レプリケーション**」 を右クリックし、「**配布を構成する**」 を選択します。
1. **配布の構成ウィザード**で、各配布の構成ページで各既定の設定を受け入れます。ウィザードが **start the SQL Server Agent Automatically** で確認が求める場合、**Yes** を選択します。
1. 「**ウィザードの完了**」 ページの 「**完了**」 を選択します。
1. **SQL Server Agent Fails to start** の場合は、**SQL Server Agent in SQL Management Studio** を右クリックしてエージェントを手動で起動する **| オブジェクト エクスプローラー | SQL Server Agent** とサービスの開始
1. **AdventureworksLT2008R2** に対応するデータベース リカバリ モデルを、新しいクエリ ウィンドウで次のクエリを実行して、FULL に設定します。

    ```sql
    ALTER DATABASE AdventureworksLT2008R2 SET RECOVERY FULL WITH NO_WAIT
    ```

1. Perform a FULL Backup of the database by executing the following query  

    ```sql
    BACKUP DATABASE AdventureworksLT2008R2 TO DISK = 'd:\awlt2008r2backup.bak'
    ```

1. バックアップが完了したら **SQL Management Studio** を閉じます。

### タスク 2: Azure Database Migration Service を使用してオンライン移行を実行する

このタスクでは、Azure Database Migration Service を構成して、VM 内で実行されているデータベースと Azure SQL Database 間のライブマイグレーションを有効にします。

> [!注]
> Azure portal でこれらすべてのタスクを完了します。

1. Azure potal で、**ストレージ アカウント** を選択します。
1. 「**マーケットプレイスの検索**」 テキストボックスに「**Azure Database Migration Service**」と入力し、<kbd>Enter</kbd> を押します。
1. **「作成」** を選択します。
1. 「**移行サービスの作成**」 ウィザードの 「**基本**」 ページで、次の値を入力し、「**次へ**」を選択します。**Networking \>\>**:

    | プロパティ | 値 |
    | --- | --- |
    | サブスクリプション | 使用しているサブスクリプションを選択します。 |
    | リソース グループ | dp050lab4rg |
    | 移行サービス名 | **Dp050dmsxxx** (**xxx**) は一意の値です。 |
    | 場所 | お近くの場所を選択します。 |
    | サービス モード | Azure |
    | 価格レベル | Premium |

1. 「**ネットワーク**」 ページの 「**仮想ネットワーク名**」 テキストボックスに「**OnPremGateway**」と入力し、「**レビュー+作成**」を選択します。
1. 「**レビュー + 作成**」 ページで、「**作成**」 を選択します。

    > [!注]
    > 展開には最大 10 分かかる場合があります。

1. デプロイが完了したら、**「リソースに移動」** を選択します。
1. **「+ 新しい移行プロジェクト」** を選択します。
1. 「**新規移行プロジェクト**」 ページで、これらの数値を入力し、「**アクティビティの作成と実行**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | プロジェクト名 | DP050OnlineMigration |
    | ソース サーバー名 | SQL Server |
    | ターゲット サーバー名 | Azure SQL Database |
    | アクティビティ タイプの選択 | オンライン データ移行 |

1. **SQL Server から Azure SQL データベースへのオンライン移行ウィザード**の 「**ソースの選択**」 ページで、次の値を入力し、「**次へ**」 を選択します。**ターゲットを選択する \>\>**:

    | プロパティ | 値 |
    | --- | --- |
    | ソース SQL Server インスタンス名 | Azure の SQL 7 VM の完全修飾ドメイン名 |
    | 認証タイプ | SQL 認証 |
    | ユーザー名 | sqladmin |
    | パスワード | Pa55w.rdPa55w.rd |
    | 暗号化接続 | 選択解除 |
    | 「サーバー証明書を信頼する」 | 選択済み |

1. 「**ターゲットの選択**」 ページで、これらの値を入力し、「**ターゲットデータベースにマップ\>\>**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | ターゲット サーバー名 | Azure SQL データベース サーバーの完全修飾ドメイン |
    | 認証タイプ | SQL 認証 |
    | ユーザー名 | sqladmin |
    | パスワード | Pa55w.rdPa55w.rd |
    | 暗号化接続 | 選択解除 |
    | 「サーバー証明書を信頼する」 | 選択済み |

1. 「**ターゲット データベースへのマップ**」 ページで、スキーマの移行を行ったデータベースを選択し、ターゲット データベースを選択してから、「**移行設定の構成\>\>**」 を選択します。
1. 「**移行設定の構成**」 ページで、各テーブルのメモを調べて、「**概要\>\>**」 を選択します。
1. 「**アクティビティ名**」 テキストボックスに名前を付け、**dp050lab4activity** と入力して、「**移行を実行する**」 を選択します。

    > [!注]
    運用環境では、SQL データベースのデータベース層を増やして、データの読み込み速度を向上させます。

1. 移行プロセス中に、時々 「**更新**」 を選択し、「**カットオーバーの準備ができました**」 と表示されるまで 「**ステータス**」 列を監視します。

    > [!注]
    > 移行中に、ソース データベースのデータに加えられた変更はすべてターゲット データベースに同期されます。

    時間がある場合は、この SQL コマンドを使用して、ソース データベースの **ProductCategory** テーブルにレコードを複数挿入することができます。ソース VM で次のコマンドを実行していることを確認してください。

    ```sql
    USE [AdventureWorksLT2008R2]
    GO
    
    INSERT INTO [Sales LT].[ProductCategory]
     ([ParentProductCategoryID]
     ,[Name]
     ,[ModifiedDate])
    VALUES
     (1, ' Myproduct', getdate())
    ```

1. データ移行が完了したら、データベースを選択してから、「**カットオーバーの開始**」 を選択します。
1. **「確認」** を確認し、**「完了」** を選択します。

結果: これで、Azure SQL Database へのオンライン移行が正常に完了しました。