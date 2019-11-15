---
lab:
    title: 'ラボ 2 - データ移行に適したツールを選択する'
    module: 'モジュール 2: データ移行に適したツールの選択'
---

# DP-050 - SQL-Workloads から Azure への移行
# ラボ 2 - データ移行に適したツールを選択する

**予想時間**：45 分間

**前提条件**：この演習を実行するための前提条件の手順はありません。

**ラボ ファイル**：この演習にはラボファイルがありません

## ラボの概要

受講者は、所定のデータ プラットフォームのモダナイゼイション ステージで 2 つのツールを使用して、自動化された方法で環境の検出を実行します。また、移行前の互換性の問題を特定し、オンプレミス サーバーの移行を実行する前に問題に対処する方法に関する計画を定義します。最後に、対象バージョンの Azure SQL Database でワークロードがどのように実行されるかを評価します。

## ラボの目的
  
このラボを終了すると、次ができるようになります。

1. Map ツールキットを使用してデータ エステート インベントリを構築する 
2. Data Migration Assistant を使用して移行候補を特定する

## シナリオ
  
あなたは、AdventureWorks 社のシニア データ エンジニアであり、データ モダナイゼイション プロジェクトの準備をしています。まず、ツールを使用して環境内に存在する SQL Server のインベントリを実行します。次に、サーバーの評価を実行して、モダナイゼイション 作業を行う前に解決する必要がある問題を特定します。オンプレミス SQL Server の 1 つは、ビジネスの売上を処理しています、あなたは、移行先のターゲット サービスで、ワークロードを処理することが可能かどうか確かめたいと思っています。

## 演習 1：Map ツールキットを使用してデータ エステート インベントリを構築する

この演習では、Microsoft Assessment and Planning ツールキットを使用して、SQL Server 2008 R2 を使用してオンプレミス サーバー上で実行されている SQL Server インスタンスに関する情報を収集します。

予想時間：25 分

この演習の主なタスクは、次のとおりです。

1. Microsoft Assessment and Planning ツールキットのインストール
1. MAPS インベントリ データベースの作成
1. リモート管理を有効にする
1. 在庫データの収集
1. MAPS でマイクロソフト SQL Server の検出情報を表示する
1. SQL Server レポートの作成

### タスク 1：Microsoft Assessment and Planning ツールキットのインストール

1. Web ブラウザを開き、[Microsoft Assessment and Planning Toolkit download site](https://www.microsoft.com/en-us/download/details.aspx?id=7826) に移動します。
1. **Download** をクリックし、ファイルをローカル フォルダに保存します。 
1. ダウンロードが完了したら、フォルダを参照して、 **MapSetup.exe** をダブルクリックします。 
1. [Microsoft Assessment and Planning ツールキットのセットアップ ウィザードへようこそ] で、**Next** をクリックします。 
1. **I accept the terms in the License Agreement** の横のチェックボックスをクリックして、**次へ** をクリックします。
1. インストールフォルダ画面で、 **Next** をクリックします。 
1. インストールの開始画面で、 **Install** をクリックします。 
1. [Microsoft Assessment and Planning ツールキットのセットアップ ウィザード完了] で、 **Finish** をクリックします。 

### タスク 2：MAPS インベントリ データベースの作成

1. Windows デスクトップで  **Start** をクリックし、Microsoft Assessment と入力して、**Microsoft Assessment and Planning Toolkit** をクリックする
1. **[はい]** をクリックして、このアプリケーションを実行するユーザー アカウント コントロール (UAC) ダイアログを受け入れます。
1. データベースの作成]または選択ダイアログ ボックスで **Create an inventory database** をクリックし、新しいデータベース名として **MapsInventory** と入力し、**OK** をクリックします。

### タスク 3：リモート管理を有効にする

1. **Start** → **cmd** と入力して、**command prompt** に移動し、**Command Prompt** を右クリックして、**Command Prompt** を選択します。
1. 次のコマンドを入力し、 **Enter** を押します。
**netsh advfirewall set currentprofile settings remotemanagement enable**

### タスク 4：在庫データの収集

1. まだ開いていない場合は、MAP ツールキットを起動します。アプリケーションのサイズを全画面に変更することもできます。
1. **[はい]** をクリックして、このアプリケーションを実行するユーザー アカウント コントロール (UAC) ダイアログを受け入れます。
1. **Where to Start** セクションで、 **Perform an Inventory** をクリックして、インベントリ ウィザードを起動します。   
1. **Inventory Scenarios** ページで、**SQL Server** チェックボックスが選択されていることを確認し、**Next** をクリックします。
1. 検検出方法のページで、**Use Active Directory Domain Services (AD DS)** をオフにします。
1. **[Manually enter computer names and credentials]** をオンにし、**Next** をクリックします。
1. [All Computer Credentials] ページで、**Create** をクリックします。 
1. アカウント エントリ ダイアログボックスに、ローカル コンピュータ ユーザーに対応する **username** を入力します (たとえば、アカウント名フィールドの管理者)
1. **Password** と **Confirm Password** の両方に、ローカル コンピュータ ユーザー (パスワードなど) を入力し、**Save** をクリックします。
1. [すべてのコンピュータ資格情報] ページで、**[次へ]** をクリックします。 
1. [資格情報要求] ページで、**[次へ]** をクリックします。 
1. [コンピュータを手動で入力] ページで、**[作成]** をクリックします。 
1. コンピュータと資格情報の指定ページで、ローカル コンピュータの名前を入力します (たとえば、次のようにします。MAP-HOL をコンピュータ名とし、**Add** をクリックします。 
1. **[すべてのコンピュータの資格情報を使用]** をオンにし、**[保存]** をクリックします。
1. **[次へ]** をクリックし、[概要] ページに表示される情報を確認します。 　
1. **[完了]** をクリックして、ステータス ダイアログとインベントリ プロセスを起動します。
注意：仮想マシンで MAP を使用している場合、1つのマシンのみのインベントリを作成するように指定していても、MAP Toolkit は、それが仮想マシンであることを検出し、VMホストのインベントリも試行します。この場合、指定された資格情報では、このようなインベントリの実行が許可されない場合があります。
1. 完了したら、ステータス ダイアログ上の **[閉じる]** をクリックします。

### タスク 5：MAPS でマイクロソフト SQL Server の検出情報を表示する

1.  **MAPS** で、 メインメニューから **File** → **Create/Select a Database...** と選択し、**Create or select a database to use** ダイアログボックスを起動します。
1.  **Use an Existing Database** をクリックして、データベースメニューから **MapsInventory** を選択し、**OK** をクリックします。
1.  UI の左ペインにある  **Database scenario** グループをクリックし、**SQL Server Discovery** タイルをクリックします。
1.  シナリオの詳細ページを調べ、SQL Server 情報を使用して円グラフを確認します。  

### タスク 6：SQL Server レポートの作成

1. SQL Server の検出の画面で、オプションセクションの下、シナリオ詳細ページの上部にある **Generate SQL Server Database Details Reports** をクリックして、レポートの生成を開始し、ステータス ダイアログを起動します。 
1. 生成が完了したことを、ステータス ダイアログが報告してきたら、閉じた後に開いているレポート フォルダが選択されていることを確認し、**Close** をクリックします。 
1. **SQLServerDatabaseDetails-<date-and-time>** を開きます。 
1. SQLServerAssessment Excel レポートで、**SQLServerSummary** タブを表示して、ネットワーク内で見つかった SQL Server データベース コンポーネントの合計数を確認します。  スプレッドシートを閉じる
1. SQL Server の検出画面で、オプションセクションの下、シナリオ詳細ページの上部にある **Generate SQL Server Assessment Reports** をクリックして、レポートの生成を開始し、ステータス ダイアログを起動します。 
1. 生成が完了したことを、ステータス ダイアログが報告してきたら、閉じた後に開いているレポート フォルダが選択されていることを確認し、**Close** をクリックします。
1. **SQLServerDatabaseDetails-<date-and-time>** を開きます
1. データベース インスタンス タブを表示して、サーバーが仮想化されているかどうか (マシンの種類) を含む、サーバーの詳細が一覧表示されているすべてのデータベース インスタンスを表示します。
1. [コンポーネント] タブを表示して、インストールされているすべてのデータベース コンポーネントがサーバーの詳細と一覧表示されます。 　
1. 表示した後、レポートとファイルブラウザを閉じます。

> **結果**：この演習を完了した後、Microsoft Assessment and Planning ツールキットを実行して、SQL Serverインスタンスに関する情報を収集したことになります。

## 演習 2：Data Migration Assistant を使用して移行候補を特定する
  
この演習では、Data Migration Assistant を使用して、SQL Server と Azure SQL Database の間の SQL Server 機能パリティと互換性の問題を探します。

予想時間：20 分間
  
この演習の主なタスクは、次のとおりです。

1. Windows に Data Migration Assistant (DMA) をインストールする方法
1. Data Migration Assistant を使用します。

### タスク 1：Windows に Data Migration Assistant をインストールする

> **注意**：データ移行アシスタントが既にインストールされている場合は、このタスクをスキップできます。

1. [https://www.microsoft.com/en-us/download/details.aspx?id=53595](https://www.microsoft.com/en-us/download/details.aspx?id=53595) に移動する
1. 必要なシステム要件を確認して、環境がソフトウェアをサポートしていることを確認します。
1. **DataMigrationAssistant.msi** をダウンロードし、**Install** をクリックする：

    1. インストーラ **DataMigrationAssistant.msi** をダブルクリックする
    2. ようこそ画面で **[次へ]** をクリックします。
    3. ユーザー 使用許諾契約書を読み、必要に応じて同意し、**[Nex]** をクリックします。 
    4. プライバシーに関する声明を読み、**Install** をクリックします。 
    5. プロンプトが表示された場合は、このアプリケーションの UAC コントロールを許可します。 **[完了]** をクリックしてインストールを完了します。 

### タスク 2：Data Migration Assistant を使用します。

1. アプリケーション ”Microsoft Data Migration Assistant” を開きます

    1. **+** を選択すると、新しいプロジェクトのダイアログが開くので、次の情報を入力します。 

        1. プロジェクトの種類: **評価**
        2. プロジェクト名：**Migration Assessment SQL DB**
        3. ソース サーバーの種類：**SQL Server**
        4. ターゲット サーバーの種類：**Azure SQL Database**
        5. **Create** をクリックする
        6. **Next** をクリックする
        7. **サーバーに接続** ダイアログ ボックスで、 **localhost** の **Server name** に、localhost のサーバー名を入力します。
        8. **Connect** をクリックする
        9. 評価するすべてのデータベースをクリックし、 **Add** をクリックします。
        10. **[評価の開始]** をクリックします。

1. 見つかった SQL Server 機能のパリティと互換性の問題を確認します。機能パリティの問題の数に注意してください。

1. Data Migration Assistant を閉じます。Azure SQL Database Managed Instance を使用して、Microsoft Data Migration Assistantを再生します。

    1. **+** を選択すると、新しいプロジェクトのダイアログが開くので、次の情報を入力します。
        
        1. プロジェクトの種類: **評価**
        2. プロジェクト名：**Migration Assessment SQL DB MI**
        3. ソース サーバーの種類：**SQL Server**
        4. ターゲット サーバーの種類：**Azure SQL Database Managed Instance**
        5. **Create** をクリックする
        6. **Next** をクリックする
        7. **サーバーに接続** ダイアログ ボックスで、 **localhost** の **Server name** に、localhost のサーバー名を入力します。
        8. **Connect** をクリックする
        9. 評価するすべてのデータベースをクリックし、 **Add** をクリックします。
        10. **[評価の開始]** をクリックします。

1. 見つかった SQL Server 機能のパリティと互換性の問題を確認します。機能パリティの問題の数に注意してください。

1. Data Migration Assistant を閉じます。Azure Virtual Machines 上の SQL Server を使用して、Microsoft Data Migration Assistant を再生します。

    1. **+** を選択すると、新しいプロジェクトのダイアログが開くので、次の情報を入力します。
    
        1. プロジェクトの種類: **評価**
        2. プロジェクト名：**Migration Assessment Azure VM**
        3. ソース サーバーの種類：**SQL Server**
        4. ターゲット サーバーの種類：**Azure SQL Database Managed Instance**
        5. **Create** をクリックする
        6. **Next** をクリックする
        7. **Server name** を入力する:**localhost**
        8. **Connect** をクリックする
        9. 評価するすべてのデータベースをクリックし、 **Add** をクリックします。
        10. **[評価の開始]** をクリックします。

1. 見つかった SQL Server 機能のパリティと互換性の問題を確認します。機能パリティの問題の数に注意してください。

> **結果**：この演習を完了した後、SQL Server のインスタンスと Azure SQL Database の間で見つかった SQL Server 機能のパリティと互換性の問題に関する情報を収集したことになります。

## ラボ レビュー

約 45 分後、インストラクターがこのラボを終了します。クラスで、各グループの調査結果について話し合います。

