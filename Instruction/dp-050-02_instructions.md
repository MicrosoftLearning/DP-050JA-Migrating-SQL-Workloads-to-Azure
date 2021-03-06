﻿# DP 050 - SQL-Workloads から Azure への移行

# ラボ 2 - データ移行に適したツールを選択する

**推定時間**: 25 分

**前提条件**: この演習を実行するための前提条件の手順はありません。

**ラボ ファイル**: このラボにはラボ ファイルがありません

## ラボの概要

受講者は、所定のデータ プラットフォームの近代化段階でデータ移行アシスタントを使用して、自動化された方法で環境を学びます。また、移行前の互換性の問題を特定し、オンプレミス サーバーの移行を実行する前に問題に対処する方法に関する計画を定義します。最後に、対象バージョンの Azure SQL Database でワークロードがどのように実行されるかを評価します。

## ラボの目的
  
この課題を完了すると、次のことができるようになります。

1. Data Migration Assistant を使用して移行候補を特定する

## シナリオ
  
あなたは、AdventureWorks 社のシニア データ エンジニアであり、データ モダナイゼイション プロジェクトの準備をしています。まず、ツールを使用して環境内に存在する SQL Server のインベントリを実行します。次に、サーバーの評価を実行して、モダナイゼイション 作業を行う前に解決する必要がある問題を特定します。オンプレミスの SQL Server の1つを使用してビジネスの売上を処理し、新しい Azure サービスが移行後にワークロードをサポートできるようにする必要があります。

## 演習 1: Data Migration Assistant を使用して移行候補を特定する
  
この演習では、Data Migration Assistant を使用して、SQL Server と Azure SQL Database の間の SQL Server 機能パリティと互換性の問題を探します。

推定時間: 20 分
  
この演習の主なタスクは次のとおりです。

1. Windows に Data Migration Assistant をインストールする。
1. Data Migration Assistant を使用します。

### Windows に Data Migration Assistant をインストールする

> [!注]
> DMA がすでにインストールされている場合でも、このタスクの手順に従って、ツールの最新バージョンを使用していることを確認してください。

1. 教室環境で実行されている **LON-DEV-01** 仮想マシンにサインインします。ユーザー名は **administrator**、パスワードは **Pa55w.rd** です。
1. Web ブラウザーを開き、Microsoft Data Migration Assistant のダウンロードページ +++https://www.microsoft.com/download/details.aspx?id=53595+++ に移動します。
1. 要件リストを確認して、ご使用の環境がソフトウェアをサポートしていることを確認してください。
1. **DataMigrationAssistant.msi** をダウンロードするには、「**ダウンロード**」 を選択し、「**実行**」 を選択します。
1. 「**ようこそ**」 画面で、「**次へ**」 を選択します。
1. 「**使用許諾契約書の条項に同意する**」 を選択し、「**次へ**」 を選択します。
1. **プライバシーに関する声明**を読み、**Install** を選択します。
1. プロンプトが表示された場合は、このアプリケーションの UAC コントロールを許可します。
1. インストールを完了するには、「**完了**」 を選択します。

### データ移行アシスタントを使用して、SQL Server から Azure SQL Database への移行の準備をします。

1. 「**スタート**」 ボタンを選択し、「**Data Migration**」と入力して、「**Microsoft データ移行アシスタント**」を選択します。
1. 新しいプロジェクトを作成するには、左側で 「**+**」 を選択します。
1. 「**新規**」 ページで、次の値を入力し、「**作成**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | プロジェクト タイプ | 評価 |
    | プロジェクト名 | +++Migration Assessment SQL DB+++ |
    | 評価の種類 | データベース エンジン |
    | ソース サーバー名 | SQL Server |
    | ターゲット サーバー名 | Azure SQL Database |

1. 「**オプション**」 ページで、既定の設定値のままにして、「**次へ**」 を選択します。
1. 「**サーバーに接続する**」 ページで、次の値を入力し、「**接続**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | サーバー名 | localhost |
    | 認証タイプ | Windows 認証 |
    | 暗号化接続 | 選択解除 |
    | 「サーバー証明書を信頼する」 | 選択済み |

1. 「**ソースの追加**」 ページで、**AdventureWorks** データベースを選択し、「**追加**」 を選択します。
1. 「**ソースの選択**」 ページで、「**評価の開始**」 を選択します。
1. 見つかった SQL Server 機能のパリティと互換性の問題を確認します。機能パリティの問題の数に注意してください。

### データ移行アシスタントを使用して、SQLServer から AzureSQL データベース マネージド インスタンスへの移行の準備をします

1. 新しいプロジェクトを作成するには、左側で 「**+**」 を選択します。
1. 「**新規**」 ページで、次の値を入力し、「**作成**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | プロジェクト タイプ | 評価 |
    | プロジェクト名 | +++Migration Assessment SQL DB MI+++ |
    | 評価の種類 | データベース エンジン |
    | ソース サーバー名 | SQL Server |
    | ターゲット サーバー名 | Azure SQL Database マネージド インスタンス |

1. 「**オプション**」 ページで、既定の設定値のままにして、「**次へ**」 を選択します。
1. 「**サーバーに接続する**」 ページで、次の値を入力し、「**接続**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | サーバー名 | localhost |
    | 認証タイプ | Windows 認証 |
    | 暗号化接続 | 選択解除 |
    | 「サーバー証明書を信頼する」 | 選択済み |

1. 「**ソースの追加**」 ページで、**AdventureWorks** データベースを選択し、「**追加**」 を選択します。
1. 「**ソースの選択**」 ページで、「**評価の開始**」 を選択します。
1. 見つかった SQL Server 機能のパリティと互換性の問題を確認します。機能パリティの問題の数に注意してください。

### データ移行アシスタントを使用して、Azure 仮想マシン上の SQL Server から SQL Server への移行の準備をします

1. 新しいプロジェクトを作成するには、左側で 「**+**」 を選択します。
1. 「**新規**」 ページで、次の値を入力し、「**作成**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | プロジェクト タイプ | 評価 |
    | プロジェクト名 | +++Migration Assessment Azure VM+++ |
    | 評価の種類 | データベース エンジン |
    | ソース サーバー名 | SQL Server |
    | ターゲット サーバー名 | Azure Virtual Machines 上の SQL Server |

1. 「**オプション**」 ページで、既定の設定値のままにして、「**次へ**」 を選択します。
1. 「**サーバーに接続する**」 ページで、次の値を入力し、「**接続**」 を選択します。

    | プロパティ | 値 |
    | --- | --- |
    | サーバー名 | localhost |
    | 認証タイプ | Windows 認証 |
    | 暗号化接続 | 選択解除 |
    | 「サーバー証明書を信頼する」 | 選択済み |

1. 「**ソースの追加**」 ページで、**AdventureWorks** データベースを選択し、「**追加**」 を選択します。
1. 「**ソースの選択**」 ページで、「**評価の開始**」 を選択します。
1. 見つかった SQL Server 機能のパリティと互換性の問題を確認します。機能パリティの問題の数に注意してください。

この演習では、オンプレミスの SQL Server のインスタンスと Azure のさまざまな SQL Server のホスティング オプションの間で見つかった SQL Server 機能のパリティと互換性の問題に関する情報を収集します。

## ラボの復習

約 20 分後、インストラクターがこのラボを終了します。クラスで、各グループの調査結果について話し合います。