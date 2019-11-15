---
lab:
    title: 'ラボ 3 – SQL Workloads を Azure Virtual Machine 内の SQL Server に移行する'
    module: 'モジュール 3: SQL ワークロードを Azure Virtual Machines に移行する'
---

# DP-050 – SQL Workloads から Azure への移行

## ラボ 3 – SQL Workloads を Azure Virtual Machine 内の SQL Server に移行する

**予想時間：** 60 分
**前提条件：** このラボでは、実行する前提条件の手順はありません。
**ラボ ファイル：** このラボにはラボ ファイルがありません

## ラボの概要

受講者は、最初に、オンプレミスの SQL Server 2008 R2 インスタンスから仮想マシンで実行されている SQL Server 2017 への移行に使用する移行プロセスを評価します。次に、Data Migration Assistant を使用して移行を実行し、Data Migration Assistant を使用してデータベースを移動します。  最後に、移行の成功を評価します。

## ラボの目的

このラボの最後には、次ができるようになります。

1. Azure で新しい SQL Server 2017 仮想マシンを作成する
2. Azure ストレージアカウント リソースと Fileshare の作成
3. SQL Server 2008 R2 データベースを Azure VM の SQL Server に移行する

## シナリオ

あなたは、Adventureworks 社の上級データベース管理リーダーであり、データ モダナイゼイション プロジェクトを実行する準備をしています。一連のデータベースを、Azure 仮想マシン内の SQL Server に移行するために必要な環境を準備し、Data Migration Assistant を使用してテスト移行を実行することになります。

## 演習 1：Azure で新しい SQL Server 2017 仮想マシンを作成する

この演習では、Azure Portal を使用して、Azure 上に新しい仮想マシンを作成します。

**予想時間：** 20 分間

この演習の主なタスクは、次のとおりです。

1.  Azure Portal 内に、新しい仮想マシンを作成する

### タスク 1：Provision a SQL Server 2017 仮想マシンのプロビジョニング

**注:** ホスト型ラボ環境でこのラボを実行している場合は、プロビジョニングされたラボ環境内で次の手順を実行します。

1. [Azure ポータル](https://portal.azure.com) で、 **Create a resource** を選択し、Marketplace 検索ボックスに **SQL Server 2017 on Windows 2016** と入力します。
1. **Free SQL Server License: SQL Server 2017 Developer on Windows Server 2016**を選択します。**select a software plan** ドロップダウンで。
1. 事前設定された構成で開始を選択します。
1. **Select a workload environment** で **Dev/Test** を選択します。
1. **Select a workload type** では、デフォルトの **General purpose (D-series)** のままにします。   
1. **Continue to Create a VM** を選択します。 
1. プロジェクトの詳細ウィンドウで、 **Create new** を選択して新しいリソース グループを作成します。 
1. リソースグループ名として **DP-050-Training** を指定します。 
1. 次の詳細を指定して、インスタンスの詳細を指定します。

    1. 仮想マシン名： **sql2017vm**
    1. リージョン: **select a region closest to your physical location**
    1. 画像: **無料 SQL サーバー ライセンス：SQL Server 2017 Developer on Windows Server 2016**.
    1. ユーザー名：**sqladmin**
    1. パスワード：**Pa55w.rd.123456789**
    1. パスワードを確定：**Pa55w.rd.123456789**

    **ファイアウォールの設定：**

      パブリック受信ポート： 選択したポートを許可するを選択する

      b. 着信ポートの選択ドロップダウンから RDP を選択します。

10. **Next:** を選択する **ディスク**

    ディスクの設定を確認し、変更なしで受け入れます。

1. **Next:** を選択する **ネットワーキング**
1. **Next:** を選択します **管理**

    管理設定を確認し、ブート診断を変更します。

    a. ブート診断を **Off** に設定する

13. **Next:** を選択する **詳細**
1. 詳細タブをスキップし、**SQL Server settings** を選択する

    SQL Server の設定タブで、次の情報が表示されます。

    a. SQL 接続：**select Public (Internet)**
    b. SQL 認証：**編集を有効にするを選択する**
    C. パスワードボックスに **Pa55w.rd.123456789** と入力します。 

15. **Review + create** を選択する

    Review + create タブの設定を確認する

    a. **Create** を選択して、仮想マシン (VM) の作成を開始します。 

**注意：** このプロセスの完了には 10分くらいかる場合があります

16.	VM の作成が完了したら、仮想マシン ブレードを開きます。
17.	 作成した **sql2017vm** を選択します。 
18.	パブリック IP アドレスを見つけて、将来の接続のために書き留める
19.	 **Configure DNS name** を選択する
20.	一意に識別可能な DNS 名を入力し、指定した完全な DNS 名を書き留めます。

    たとえば: SQL2017VMxxxx.centralus.cloudapp.azure.com

21.	[保存] をクリックします。

``` shell
結果: この演習を完了すると、Azure Virtual Machine で SQL Server 2017 インスタンスが実行されます。
```

## 演習 2: Azure ストレージ アカウントと Fileshare の作成

この演習では、Azure Portal を使用して、Azure 上に新しい仮想マシンを作成します。

**予想時間：** 15 分

この演習の主なタスクは、次のとおりです。

1. Azure Storage アカウントを作成する
2. Azure Storage アカウントでファイル共有を作成する

### タスク 1：Azure Storage アカウントを作成する

1. [Azure portal](https://portal.azure.com) で、**Storage Accounts** ブレードを選択します。
2. **Add**を選択します。
3. ストレージアカウントの作成ウィンドウで、次の情報を入力します。
    a.リソース グループ:  **Existing** を選択する
    b. 前の演習で作成した **DP-050-Training** リソース グループを選択する
    c. ストレージ アカウント名： **dp050storagexxxx** (xxxx) は乱数の文字
    d. 場所：前の演習で仮想マシンを作成した場所に最も近い場所を選択します。
    e. 他のデフォルト設定はそのままにしておきます。
4. **Review + create** をクリックして、詳細セクションとタグセクションをスキップする
5. **Create** を選択する

    **注意：**  この展開には数分かかる場合があります。

6. 完了時に、 **go To Resource** をクリックする

### タスク 2：Azure FileShare を作成する

1. ストレージ アカウントページで、 **Files** を選択する
2. ファイルページ で、 **+ File share** を選択する
3. 次の情報を入力してください。
    a. 名前： **backupshare**
    クォータ： **200 Gib**
4. **Create** を選択する
5. ファイル共有の作成後、作成されたファイル共有の右側で…を選択する
6. ドロップダウン リストから **Connect** を選択する

![ドロップダウン リスト](https://github.com/MicrosoftLearning/DP-050-Migrating-SQL-Workloads-to-Azure/blob/master/images/dropdown.png)

7. 接続ブレードで、ドライブ文字 **U:** を選択する
8. Alternatively の下にリストされているテキストから、接続コマンドの構文をコピーします。キーがスラッシュで始まらない場合は、代わりにこのコマンドを実行します。

テキストは、次のコマンド構文のようになります。

```shell
cmdkey /add:dp050storagexxxx.file.core.windows.net /user:Azure\dp050storagexxxx /pass:Hcty8OXn7jcON/ePk3OvswD7eRusfWWAw9lakhX9P9m4MnuqZEt2I8pDYAEiyAZJx4Na39UZEwAyH8PlnVa36Q==

```

9. **Notepad** を開く
10. 上記のコマンドの内容を貼り付て、ファイルを **Labfiles** に **MapNetworkdrive.txt** として保存する
11. **Notepad** 閉じる 
12.	これで、Azure ポータル Notepad を閉じることができます。

```shell
結果: これで、SQL Server データベースのバックアップ ファイルの共有アクセス場所として使用される Azure Fileshare が正常に作成されました。次の演習では、共有場所にアクセスできるように SQL インスタンスを構成します。
```

## 演習 3: SQL Server インスタンスの接続を作成して、Azure Fileshare に接続します

この演習では、Azure Fileshare にアクセスできるように SQL Server 環境を構成します。
**予想時間：** 10 分

この演習の主なタスクは、次のとおりです。

1. ネットワーク ドライブをマッピングして SQL 管理スタジオからファイル共有を登録する
2. Azure Storage アカウントでファイル共有を作成する

### タスク 1：SQL Management Studio にサーバー インスタンスを登録する

1. SQL Server 2008 R2 ラボ環境で、SQL Management Studio を起動する
2. ローカル インスタンスに接続する (LONDON)
3. SQL Management Studio Object Explorer で、 を選択する **Connect | Database Engine** を使用し、 **Connect to Server** ダイアログ ボックスに次の情報を入力します。

    Servername：'SQL 2017 VM の完全修飾ドメイン名’
    たとえば、sql2017vmxxxx.centralus.cloudapp.azure.com
4. SQL Server 認証を選択し、次のユーザーとパスワードを指定する
    ログイン: **sqladmin**
    パスワード： **Pa55w.rd.123456789**

### タスク 2：SQL インスタンスをファイル共有に接続する

**注意：** SQL Server がファイル共有上のドライブ文字に接続できるようにするには、xp_cmdshell を実行しているネットワーク ドライブをマップする必要があります。セキュリティ上の理由から、SQL Server サービス アカウントのコマンド ライン アクセスを制限する必要があります。デフォルトでは、SQL コマンド ラインは無効になっています。

1. Notepad の演習の前の部分で作成した MapNetworkDrive.txt を開きます。
2.	テキストをコピーする
3.	SQL Management Studio で、ローカル サーバー (LONDON) に接続したまま新しいクエリを作成する
4.	MapNetworkDrive からクエリ ウィンドウにテキストを貼り付ける
5.	次のスクリーンショットを反映するようにクエリを変更し、xp_cmhdshell ストアド プロシージャを使用してコマンド行を実行します。
 ![sp_configure options](https://github.com/MicrosoftLearning/DP-050-Migrating-SQL-Workloads-to-Azure/blob/master/images/cmdshell.png)
6.	クエリを実行し、ネットワーク ドライブにアクセス可能であることを検証する
7.	Labfiles フォルダにクエリを **MapNetworkdrive.sql** として保存
8.	次のクエリを実行して、新しいクエリ ウィンドウを開始し、xp_cmdshell を無効にします。

```sql
    SP_CONFIGURE ‘xp_cmdshell’,1
```

9.	Object Explorer で、Azure VM SQL Server インスタンスを選択し、新しいクエリを作成する
10.	保存したファイル Mapnetworkdrive.sql を開き、SQLS Server 2017 インスタンスでスクリプトを実行する
11.	SQL Server 2017 インスタンスで xp_cmdshell を無効にする手順を繰り返します。


## 演習 4：SQL Server データを使用したデータベース移行の実行 

この演習では、Azure Portal を使用して、Azure 上に新しい仮想マシンを作成します。
**予想時間：** 10 分

この演習の主なタスクは、次のとおりです。

1. データベースを Data Migration Assistant で移行する
2. データベースの正常な移行を検証する

### タスク 1：Data Migration Assistant を使用して SQL データベースを移行する

1.	SQL Server 2008 R2 ラボ環境で、 **“Microsoft Data Migration Assistant”** アプリケーションを開く
2.	 **+** を選択すると、新しいプロジェクトのダイアログが開くので、次の情報を入力します。 
    a. プロジェクトの種類： **Migration**
    b. プロジェクト名：**SQL VM への移行**
    c. ソースサーバーの種類：**SQL Server**
    d. ターゲット サーバーの種類：**Azure Virtual Machines の SQL Server**
3.	 **Create** をクリックする
4.	 **Next** をクリックする
5.	ソース サーバーの詳細サーバー名ダイアログ ボックスに **Server name of localhost** を入力する
6.	ターゲット サーバーの詳細 サーバー名ダイアログ ボックスに、完全修飾名を使用して Azure SQL Virtual マシンのサーバー名を入力する
7.	ターゲット サーバー認証の種類ダイアログ ボックスで **Server name of localhost** を選択する
8.	ターゲット サーバーの詳細 SQL 認証資格情報で、 **sqladmin** と入力して、**Pa55.w.rd.123456789** をパスワードとして指定します。   
9.	接続プロパティで、 **Encrypt Connection** プロパティのチェックボックスをオフにします。
10.	 **Next** をクリックします。
11.	次のデータベースの選択を解除します。
    •	AdventureworksDW2008_4M
    •	Reportserver
    •	ReportServerTempDB
12.	共有場所ダイアログ ボックスの種類: **U:\**
13.	ログインの選択ウィンドウを確認する
14.	 **StartMigration** をクリックします。

**注意：** すべてのデータベースが共有ネットワーク ドライブ (Azure Fileshare) にバックアップされます。

15.	移行プロセスを監視します。
16.	完了後、データ移行アシスタントを閉じる

### タスク 2：正常な移行の検証

1.	SQL Server 2008 R2 ラボ環境で、 **“SQL Management Studio”** を開く
2.	Object Explorer で **Databases** リストを **SQL Server 2017 instance** に展開します。
3.	データベースが正常に移行されたことを確認する
4.	**New Query** を作成する
5.	次のクエリを入力して実行して、各データベースのデータベース互換性レベルを検証します。

```sql
SELECT name, compatibility_level FROM sys.databases
```

6.	クエリの結果を確認する
7.	**AdventureworksLT2008R2** データベースに対するデータベース互換レベルを変更するには、次のクエリを使用します。

```sql
ALTER DATABASE AdventureWorksLT2008R2
SET COMPATIBILITY_LEVEL = 110;
GO
```

8.	AdventureworksLT2008R2 データベースを次のクエリでバックアップします。
  
```sql
BACKUP DATABASE AdventureworksLT2008R2  
TO DISK = 'U:\AdventureworksLT2008R2'  
   WITH FORMAT,  
      MEDIANAME = 'AdventureworksLT2008R2',  
      NAME = 'Full Backup of AdventureworksLT2008R2;  
```


9.	バックアップが正常に完了したら、 **SQL Management Studio** を閉じる

```shell
結果: これで、Azure VM 内で実行されている SQL Server 2017 への SQL 2008R2 データベースの正常な移行が完了しました。
```
