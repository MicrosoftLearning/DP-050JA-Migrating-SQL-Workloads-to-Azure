---
lab:
    title: 'ラボ 4 - SQL-Workloads から Azure への移行'
    module: 'モジュール 4: SQL ワークロードを SQL Database に移行する'
---

# DP-050 – SQL-Workloads から Azure への移行

## ラボ 4 - SQL-Workloads から Azure への移行  

**予想時間：** 60 分

**前提条件：** このラボでは、実行する前提条件の手順はありません。

**ラボ ファイル：** このラボにはラボ ファイルがありません

## ラボの概要

このラボでは、Azure SQL Database への移行を実行します。まず、スキーマ移行を実行し、前提条件として宛先データベースを作成し、次に SQL インスタンスで実行されているデータベースから Azure 上の SQL Database に移行します。Database Migration Service (DMS) を使用してオンライン移行を実行し、データ移行が切り替わるまで、ソース データベースとターゲット データベースの間でデータの同期を維持します。

## ラボの目的

このラボの最後には、次ができるようになります。

1. Azure Cloud Shell を使用した SQL Database の作成
1. Azure Data Migration Service の構成
1. データベース スキーマを Azure SQL データベースに移行する
1. Data Migration Service を使用したオンライン移行の実行

## シナリオ

あなたは、Adventureworks 社の上級データベース管理リーダーであり、データ モダナイゼイション プロジェクトを実行する準備をしています。一連のデータベースを、Azure 仮想マシン内の SQL Server に移行するために必要な環境を準備し、Data Migration Assistant を使用してテスト移行を実行することになります。

## 演習 1：Azure Cloud Shellを使用した SQL Database の作成

この演習では、Azure Cloud Shell を使用して次を行います。

- 新しいリソース グループの作成

- 新しい Azure Cache for Redis インスタンスの作成

- Azure SQL SQL Database Server ファイアウォールの構成

- Azure SQL Database のコピーの作成

**予想時間：** 20 分間

Azure には、サービスのインストール、管理、および展開を自動化するための多くのオプションがあります。オプションの 1 つは、軽量のクロスプラットフォーム コマンド ライン ツールである Azure CLI コマンド ライン ツールをインストールすることです。もう 1 つのオプションは、Azure Cloud Shell を使用して自動化しスクリプトを作成することです。

Azure Cloud Shell は、Azure でホストされ、ブラウザーを介して管理される対話型シェル環境です。クラウド シェルを使用すると、bash または PowerShell を使用して Azure サービスを操作できます。Cloud Shell がプレインストールされたコマンドを使用すると、ローカル環境上に何もインストールしなくても、ここに記載されているコードを実行できます。

### タスク 1：ログオンして Azure Cloud Shell を構成する

注意：このタスクは、Azure Portal で完全に完了できます

1. [Azure Shell](https://shell.azure.com)に移動する
1. このトレーニングで使用した資格情報を使用して Azure サブスクリプションにログオンする

 ![Welcome to Azure Shell Sign in Window](/images/azureshell.png)

3. スクリプト環境として Bash を選択する

![Azure Shell Screen to select Bach shell](/images/bash.png)

### タスク 2：新しいストレージ アカウントを作成し、Azure Cloud Shell と共有する

1. Azure Cloud Shell にストレージ アカウントが作成されていないことを確認するメッセージが表示された場合は、 **Show Advanced Settings** をクリックします。
1. 次の詳細を完成させてください。
    1. サブスクリプション：**このラボで使用する Azure サブスクリプションを選択する**
    1. Cloud Shell リージョン：**地理的な場所に最も近い利用可能な Cloud Shell リージョンを選択する**
    1. リソース グループ：  
        1. **Create new** をクリックする
        1. **Dp050lab4rg** と入力する
    1. ストレージ アカウント：
        1. **Create new** をクリックする
        1. **youridentifier** が一意味の名前である場合は、 **dp050sa&lt;youridentifier&gt;** を入力する
    1. ファイル共有:
        1. **Create new** をクリックする
        1. **youridentifier** が一意味の名前である場合は、 **dp050share&lt;youridentifier&gt;** を入力する
    1. Create ストレージを選択する

完了すると、Azure Cloud Shell コマンドラインが開く 

### タスク 3：次のスクリプトを変更して実行することにより、SQL Database サーバー インスタンスと SQL Database を作成する

1. **{}** アイコンをクリックして、エディタを開く
1. 以下のスクリプトを貼り付け、ファイルとして保存します。 **CreateSQLDBandServer**

&lt;スクリプトファイルは、ホストされたラボ環境 Labfiles フォルダ内、および github のラボ起動フォルダ内でも利用できます&gt;

```shell
#新しい sql db サーバー インスタンスと sql db を作成する bash スクリプト  

#Disclaimer：Azure Database を作成する方法に関するサンプル スクリプトです。ラボの目的で最も制限の少ないファイアウォール設定を使用します。このスクリプトを使用しない  

#Azure CloudShell Bash に渡される変数定義

resourcegroup=dp-050-labresourcegroup

#一意のサーバー名を指定するには、次の変数を編集する

servername=dp-050-servername

#次の変数を編集して場所を提供する。 azure locations は、シェル コマンド インターフェイスで az account list-locations -o table と入力して一覧表示できる

location=eastus

adminuser=sqladmin

password=Pa55w.rd.123456789

firewallrule=dp-050-access

#一意のサーバー名を指定するにはスクリプトを編集する

labdatabase=dp-050-AdventureworksLT2008R2

#リソース グループを作成する

az group create --name $resourcegroup --location $location

#sql db servername を作成する

az sql server create --name $servername --resource-group $resourcegroup --admin-user $adminuser --admin-password $password --location $location

#現在のファイアウォールリストを表示する

az sql server firewall-rule list --resource-group $resourcegroup --server $servername

#サーバー ベースのファイアウォールを作成する。注意：環境に基づいて開始/終了 IP 範囲を制限する必要があります。

az sql server firewall-rule create --resource-group $resourcegroup --server $servername --name $firewallrule --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255

#汎用 SQL Database を作成する

az sql db create --name $labdatabase --resource-group $resourcegroup --server $servername -e GeneralPurpose -f Gen4 -c 1 

```

3. **sh CreateSQLDBandServer** と入力して、スクリプトを実行します。

スクリプトが完了して正常に実行された場合には、リソース グループ、サーバー、およびデータベースが Azure アカウント サブスクリプションに作成されたことが検証できます。

bash スクリプト出力は、次の強調表示テキストに似たサーバー名を示す必要があります。

```shell
{ "administratorLogin": "sqladmin",

  "administratorLoginPassword": null,

  "fullyQualifiedDomainName": "dp-050-servername.database.windows.net",

  "id": "/subscriptions/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/resourceGroups/dp-050-labresourcegroup/providers/Microsoft.Sql/servers/dp-050-servername",

  "identity": null,

  "kind": "v12.0",

  "location": "eastus",

  "name": "dp-050-servername",

  "resourceGroup": "dp-050-labresourcegroup",

  "state": "Ready",

  "tags": null,

  "type": "Microsoft.Sql/servers",

  "version": "12.0"

}
```

## 演習 2：データベースを Azure SQL Database (オフライン) に移行する

この演習では、Azure（以前のラボで作成された SQL 2017 Vm）で実行されている SQL Server インスタンスのデータベースのデータベース スキーマを移行し、スキーマを以前の演習で作成した SQL Database に読み込みます。

次のことができるようになります。

- Data Migration Assistant を使用した新しい移行プロジェクトの作成
- スキーマ移行のみの実行

**予想時間：** 20 分間

**注意：** この演習は、SQL Server 2008 R2 仮想マシンで実行されている Data Migration Assistant を使用して完了する

### タスク 1：Data Migration Assistant を使用した移行プロジェクトの作成

1. **Microsoft Data Migration Assistant** を開く
1. **[+]** をクリックして新しいプロジェクトを開始する
1. プロジェクトの種類として **Migration** を選択する
1. **SQLDBMigration** をプロジェクトの名前として入力する
1. 次の設定を検証します。
1. ソース サーバーの種類：**SQL Server**
1. ターゲット サーバーの種類：**Azure SQL database**
1. 移行のスコープ：**スキーマのみ**
1. **Create** を選択する

### タスク 2：スキーマ移行を実行する

1. ソース サーバーに接続で **LONDON** と入力する
1. **Encrypt Connection** オプションをオフにする
1. **Connect** を選択する
1. データベースのリストが表示されるので、 **AdventureworksLT2008R2** を選択する
1. 前回の評価で移行の問題が確認されなければ、 **Uncheck the Assess database before Migration** チェックボックスをオフにする
1. **Next** をクリックする
1. ターゲット サーバーに接続]ダイアログ ボックスで、前の演習で作成したサーバーの  **servername** を入力します。**full qualified name** で、サーバーをリストする (dp-050-servername.database.windows.netなど) 
1. 認証の種類を **SQL Server Authentication** に変更する
1. ユーザー名の入力： **sqladmin**
1. パスワードの入力： **Pa55w.rd.123456789**
1. **Connect** をクリックする
1. 前の演習で作成したターゲット データベースを SQL Database サーバー上のターゲット データベースとして選択する
1. **Next** をクリックする
1. 問題が見つからなかったため、すべてのスキーマオブジェクトを確認し、SQLスクリプトの生成をクリックします。

Data Migration Assistant ウィンドウは次のようになります。
![Data Migration Assistant Window](/images/dbmigrate.png)

15. Deploy Schema をクリックする
1. Data Migration Assistant を閉じる

**注意：** すべてのデータベース オブジェクトが Azure SQL Database に移行されます。この移行が完了するまでに最大 10 分かかる場合があります 

## 演習 3：オンプレミス データベースを Azure SQL Database  (オンライン) に移行する

この演習では、Azure Data Migration を使用してデータベースのオンライン移行を実行します。

次のことができるようになります。

- レプリケーションのソース インスタンスを構成し、移行の前提条件が満たされていることを確認する。
- Azure Data Migration Service を使用してライブ マイグレーションを実行する

**予想時間：** 20 分間

### タスク 1：SQL Server をレプリケーション ディストリビュータとして構成する

このタスクは、SQL 2008 R2 仮想マシンの SQL Management Studio を使用して完了しますが、 **SQL Server 2017 instance.** に接続されます。

**注意：** ホストされている SQL 2008 R2 VM と Azure 間の VPN アクセスを構成する必要がないように、SQL Server 2017 インスタンスに接続している間にこれらのタスクを実行します。ラボ以外の環境では、通常、VPN または Expressroute を設定します。

1. SQL Management Studio Object Explorer で、Lab3 で作成した SQL Server 2017 インスタンスに接続します。
    SQL Server 2017 インスタンスが登録されていない場合は、次の手順を実行します。
        を選択する **Connect | Database Engine** を使用し、 **Connect to Server** ダイアログ ボックスに次の情報を入力します。

            Servername： **<the full qualified domain name of your SQL 2017 VM>**

            たとえば、sql2017vmxxxx.centralus.cloudapp.azure.com 

2. **Replication** で **Right-click** する
1. **Configure Distribution** を選択する
1. **Configure Distribution Wizard** の構成を実行する
1. 配布ページの構成の各既定の設定を受け入れます。
    1. ウィザードが **start the SQL Server Agent Automatically** で確認が求める場合、 **Yes** をクリックする   
1. **Next** をクリックする
1.  **Snapshot** で **Next** をクリックする 
1. **Distribution** データベース名で **Next** をする
1. **Publisher** ダイアログ ウィンドウで **Next** をする 
1. **Wizard Action** ダイアログウィンドウの **Next** をする
1. **Finish** をクリックする
1. **SQL Server Agent Fails to start** の場合は、を右クリックしてエージェントを手動で起動する **SQL Server Agent in SQL Management Studio | Object Explorer | SQL Server Agent** とサービスの開始
1. **AdventureworksLT2008R2** に対応するデータベース リカバリ モデルを、新しいクエリ ウィンドウで次のクエリを実行して、FULL に設定します。 

```sql 
ALTER DATABASE AdventureworksLT2008R2 SET RECOVERY FULL WITH NO_WAIT
```

14. Perform a FULL Backup of the database by executing the following query  

```sql
BACKUP DATABASE AdventureworksLT2008R2 TO DISK = ‘d:\awlt2008r2backup.bak’
```

15. バックアップが完了したら **SQL Management Studio** を閉じます。 

### タスク 2：Azure Database Migration Service を使用してオンライン移行を実行する

このタスクでは、Azure Database Migration Service を構成して、VM 内で実行されているデータベースと SQL Database 間のライブマイグレーションを有効にします。

**注:** これらのタスクはすべて Azure ポータルで完了します

1. Azure portal で、 **Create a resource** をクリックする
1. **Azure Database Migration Service** と入力する
1. **[作成]** をクリックする
1. サービス名で、サービス名 **dp050dmsxxxxxx** を入力する。 xxxxx は、一意の値
1. **Azure subscription** を選択する
1. 以前に作成した **resource group** を選択する
1. **location** を選択する
1. **new Virtual network** を作成し、 **OnPremGateway** と命名する 
1. 価格レベルを **Premium** に変更する
1. **OK** をクリックする
1. **Create** をクリックする

**注意：** 展開には最大 10 分かかる場合があります

12.展開が完了したら、Go to Resource をクリックする

    オプションで、Azure ポータルの左側のブレードに  Azure Data Migration Service を追加できます

    a) Azure Portal 上のすべてのサービスで Azure Data Migration Services を検索する 
    b) スターアイコンをオンして、Azure ポータルの左側のブレードに Azure Data Migration Services を追加する
    c) 左側のブレードを Azure Data Migration Services に展開する
    d) Data Migration Service を選択する 

13. New Migration Project をクリックする
1. New Migration Project ダイアログボックスを次のように完了します。

    プロジェクト名：**DP050OnlineMigration**

    ソース サーバーの種類：**SQL Server**

    ターゲット サーバーの種類：**Azure SQL Database**

    アクティビティの種類を選択：**オンライン データ移行への変更**

15. Save を選択する
1. Create and Run Activity をクリックする
1. 移行ソースの詳細では、次の情報を入力します。

    ソース SQL Server インスタンス名：&lt;sql2017 vm の 完全修飾ドメイン名&gt; 

    認証タイプ：**SQL 認証**

    ユーザー名： **sqladmin**

    パスワード：**Pa55w.rd.123456789**

18. 接続の暗号化のチェックを外す
1. Save を選択する
1. SQL 移行のターゲット先を選択し、移行ターゲットの詳細に次の詳細を入力します。

    ターゲット サーバー名：&lt;Azure SQLデータベースサーバーの完全修飾ドメイン&gt;

    認証タイプ：**SQL 認証**

    ユーザー名：**sqladmin**

    パスワード：**Pa55w.rd.123456789**

21. **Save** を選択する
1. **Map to target Database** ウィンドウで、スキーマの移行を行ったデータベースを選択し、ターゲット データベースも選択します。 
1. **Save** を選択する
1. **Configure migration** 設定ウィンドウで、ドロップダウン リストをクリックすると、dbo テーブルに対して Change Data Capture の変更が有効になっていないため、1 つのテーブルがレプリケートされていないことがわかります。 
1. **Save** を選択する
1. **Migration project** アクティビティに名前を付けて、 **dp050lab4activity** と入力する

**注:** 運用環境では、SQL データベースのデータベース層を増やして、データの読み込み速度を向上させます。

27. **Run Migration** をクリックする
1. 移行プロセス中に、 **Activity** ウインドウを **リフレッシュ** します。
1. ステータスが **Ready to cutover** と表示されるまで、ステータスを確認します。

    **注:** データベースの変更が発生した場合は、増分データの負荷を確認できます。

    または、次のコマンド (ソース データベースとして設定された SQL インスタンス) を使用して、ソース データベースの **Productcategory** テーブルに一部のレコードを挿入することもできます。

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

    次に、増分データ同期タブを確認し、挿入を確認します。

30. データベースを選択し、**カットオーバーの開始** をクリックする
1. **Confirm** を確認する
1. **Validate my database(s)** をオンにする
1. すべての検証オプションを確認する
1. **Apply** をクリックする

おめでとう！これで、Azure SQL Database へのオンライン移行が正常に完了しました。
