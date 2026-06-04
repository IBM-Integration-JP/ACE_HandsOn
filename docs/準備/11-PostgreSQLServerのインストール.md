# PostgreSQL Server のインストール手順

① インストール用ファイルを準備します。
本ハンズオン演習の準備では、PostgreSQL Server 15.x をデータベースに使用します。
[公式サイト](https://www.postgresql.org/download/)からWindows用のインストーラーをダウンロードします。

② ダウンロードした「postgresql-15.x-x-windows-x64.exe」ファイルを右クリックしてメニューを開き、「管理者として実行」を選択します。「Setup」のウィザードが開きます。「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/1.png)

③ インストール先ディレクトリはデフォルトのまま「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/2.png)

④ 「コンポーネントの選択」ダイアログではデフォルトのまま「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/3.png)

⑤ データディレクトリはデフォルトのまま「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/4.png)

⑥ ダイアログに表示されているとおり、データベースのスーパーユーザー名は「postgres」です。パスワードは「(パスワード)」に設定し、「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/5.png)

⑦ ポート番号はデフォルトのまま「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/6.png)

⑧ ロケールは DEFAULT のまま「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/7.png)

⑨ プリインストールサマリーの内容を確認し「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/8.png)

⑩ 「Ready to Install」ダイアログで「Next」をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/9.png)

⑪ インストールが完了したら 「Finish」 をクリックします。

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/10.png)

> [!NOTE]
>
> IBM ACE ツールキットからデータベースに接続するためには、JDBC ドライバーを別途ダウンロードする必要があります。
> JDBC ドライバーのダウンロードには 「Stack Builder」 を使用します。インストール完了画面で表示されるチェックボックスをオンにすると、Stack Builder を起動できます。

⑫ PostgreSQLが導入されたことを確認します。  
 　フォルダー：PostgreSQL 15

![スクリーンショット](../images/準備/11-PostgreSQLServerのインストール/11.png)

以上となります。
