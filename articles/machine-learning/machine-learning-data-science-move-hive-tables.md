<properties 
	pageTitle="データを作成して BLOB ストレージ から Hive テーブルに読み込む | Microsoft Azure" 
	description="Hive テーブルを作成し、BLOB 内のデータを Hive テーブルに読み込む" 
	services="machine-learning,storage" 
	documentationCenter="" 
	authors="bradsev"
	manager="paulettm"
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/14/2016" 
	ms.author="hangzh;bradsev" />

 
#データを作成して Azure BLOB ストレージから Hive テーブルに読み込む

このトピックでは Hive テーブルを作成し、Azure Blob Storage からデータを読み込む汎用の Hive クエリを紹介しています。Hive テーブルをパーティション分割する方法や、Optimized Row Columnar (ORC) 形式を使用してクエリのパフォーマンスを向上させる方法についてのガイダンスも提供されます。

この**メニュー**は、Team Data Science Process (TDSP) でデータを保存および処理できるターゲット環境にデータを取り込む方法について説明するトピックにリンクしています。

[AZURE.INCLUDE [cap-ingest-data-selector](../../includes/cap-ingest-data-selector.md)]


## 前提条件
この記事では、以下のことを前提としています。
 
* Azure のストレージ アカウントが作成されている。手順については、「[Azure ストレージ アカウントの作成](../hdinsight-get-started.md#storage)」をご覧ください。 
* HDInsight サービスでカスタマイズされた Hadoop クラスターがプロビジョニングされている。手順については、「[Advanced Analytics Process and Technology 向けに HDInsight Hadoop クラスターをカスタマイズする](machine-learning-data-science-customize-hadoop-cluster.md)」をご覧ください。
* クラスターへのリモート アクセスを有効にし、ログインし、Hadoop コマンド ライン コンソールを開いている。手順については、「[Hadoop クラスターのヘッド ノードへのアクセス](machine-learning-data-science-customize-hadoop-cluster.md#headnode)」をご覧ください。 

## Azure BLOB ストレージにデータをアップロードする
[高度な分析のための Azure 仮想マシンのセットアップ](machine-learning-data-science-setup-virtual-machine.md)に関するページの指示に従って Azure 仮想マシンを作成した場合、このスクリプト ファイルは仮想マシンの *C:\\Users\\<ユーザー名>\\Documents\\Data Science Scripts* ディレクトリにダウンロードされています。これらの Hive クエリに必要なことは、独自のデータ スキーマと Azure BLOB ストレージの構成を適切なフィールドに接続し、送信できるようにすることだけです。

ここでは、Hive テーブルのデータが**圧縮されていない**表形式であることと、Hadoop クラスターが使用するストレージ アカウントの既定の (または追加の) コンテナーにデータがアップロードされていることを想定しています。

_NYC タクシー乗車データ_の実習を行う場合は、まず 24 個の <a href="http://www.andresmh.com/nyctaxitrips/" target="_blank">NYC タクシー乗車データ</a> ファイル (12 個の Trip ファイルと 12 個の Fare ファイル) をダウンロードし、すべてのファイルを .csv ファイルに**解凍**します。次に、「[Advanced Analytics Process and Technology 向けに Azure HDInsight Hadoop クラスターをカスタマイズする](machine-learning-data-science-customize-hadoop-cluster.md)」で説明する手順で作成された Azure ストレージ アカウントの既定のコンテナー (または適切なコンテナー) にそれらのファイルをアップロードする必要があります。ストレージ アカウントの既定のコンテナーに .csv ファイルをアップロードするプロセスについては、この[ページ](machine-learning-data-science-process-hive-walkthrough.md#upload)をご覧ください。


## <a name="submit"></a>Hive クエリを送信する方法

Hive クエリは、以下のものを使用して送信できます。

1. [Hadoop クラスターのヘッド ノードで Hadoop コマンド ラインを使用して Hive クエリを送信する](#headnode)
2. [Hive エディターで Hive クエリを送信する](#hive-editor)
3. [Azure PowerShell コマンドで Hive クエリを送信する](#ps)
 
Hive クエリは SQL に似ています。SQL を使い慣れているユーザーには、[Hive for SQL Users チート シート](http://hortonworks.com/wp-content/uploads/2013/05/hql_cheat_sheet.pdf)が役立つ場合があります。

Hive クエリの送信時、Hive クエリの出力先を、画面上、ヘッド ノード上のローカル ファイル、または Azure BLOB のどれにするか制御できます。


###<a name="headnode"></a> 1.Hadoop クラスターのヘッドノードで Hadoop コマンド ラインを使用して Hive クエリを送信する

Hive クエリが複雑な場合、Hadoop クラスターのヘッド ノードに直接クエリを送信することにより、通常、Hive エディターまたは Azure PowerShell スクリプトを使用して送信するよりも速度が上がります。

Hadoop クラスターのヘッド ノードにログインし、ヘッド ノードのデスクトップで Hadoop コマンド ラインを開き、コマンド `cd %hive_home%\bin` を入力します。

ユーザーには Hadoop コマンド ラインでハイブ クエリを送信する 3 つの方法があります。

* 直接
* .hql ファイルの使用
* Hive コマンド コンソールで

#### Hadoop コマンド ラインでハイブ クエリを直接送信する 

ユーザーは、`hive -e "<your hive query>;` のようなコマンドを実行することで、Hadoop コマンド ラインで単純な Hive クエリを直接送信できます。次に例を示します。ここで赤いボックスは Hive クエリを送信するコマンドを囲んでいます。緑色のボックスは Hive クエリの出力を囲んでいます。

![Create workspace](./media/machine-learning-data-science-move-hive-tables/run-hive-queries-1.png)

#### .hql ファイルで Hive クエリを送信する

ハイブ クエリがより複雑で、複数の行が存在する場合、コマンド ラインやハイブ コマンド コンソールでクエリを編集することは実際的ではありません。別の方法として、Hadoop クラスターのヘッド ノードでテキスト エディターを使用して、ヘッド ノードのローカル ディレクトリの中の .hql ファイルにハイブ クエリを保存します。次のように `-f` 引数を使用すると、.hql ファイルの Hive クエリを送信できます。
	
	hive -f "<path to the .hql file>"

![Create workspace](./media/machine-learning-data-science-move-hive-tables/run-hive-queries-3.png)


**Hive クエリの進行状況ステータス画面の出力を抑制する**

既定では、Hadoop コマンド ラインでハイブ クエリを送信した後に、マップ/縮小ジョブの進捗が画面に出力されます。マップ/縮小ジョブの進捗の画面出力を抑制するには、次のように、コマンド ラインで引数 `-S` ("S" は大文字) を使用します。

	hive -S -f "<path to the .hql file>"
	hive -S -e "<Hive queries>"

#### Hive コマンド コンソールで Hive クエリを送信する。

ユーザーが Hadoop コマンド ラインで `hive` コマンドを実行すると、まず Hive コマンド コンソールに入力できるようになります。その後、Hive コマンド コンソールで Hive クエリを送信します。たとえば次のようになります。この例では、2 つの赤いボックスは、それぞれ Hive コマンド コンソールに入るために使用するコマンドと、Hive コマンド コンソールで送信された Hive クエリを強調表示しています。緑色のボックスは、Hive クエリからの出力を強調表示しています。

![Create workspace](./media/machine-learning-data-science-move-hive-tables/run-hive-queries-2.png)

前の例では、画面上に直接 Hive クエリの結果を出力しています。ユーザーは、ヘッド ノードにあるローカル ファイルや Azure BLOB に出力を書き込むこともできます。次に、ユーザーはその他のツールを使用して、ハイブ クエリの出力をさらに分析することができます。

**Hive クエリの結果をローカル ファイルに出力する。**

Hive クエリの結果をヘッド ノード上のローカル ディレクトリに出力するには、次のように Hadoop コマンド ラインで Hive クエリを送信する必要があります。

	hive -e "<hive query>" > <local path in the head node>

次の例では、Hive クエリの出力が `C:\apps\temp` ディレクトリ内の `hivequeryoutput.txt` ファイルに書き込まれます。

![Create workspace](./media/machine-learning-data-science-move-hive-tables/output-hive-results-1.png)

**Hive クエリの結果を Azure BLOB に出力する**

ユーザーは、Hadoop クラスターの既定のコンテナー内にある Azure BLOB に Hive クエリの結果を出力することもできます。ハイブ クエリは、次のように指定する必要があります。

	insert overwrite directory wasb:///<directory within the default container> <select clause from ...>

次の例では、Hive クエリの出力が、Hadoop クラスターの既定のコンテナー内にある BLOB ディレクトリ `queryoutputdir` に書き込まれます。ここでは、ディレクトリ名のみを指定する必要があります (BLOB 名は必要ありません)。`wasb:///queryoutputdir/queryoutput.txt` のようにディレクトリと BLOB の両方の名前を指定すると、エラーがスローされます。

![Create workspace](./media/machine-learning-data-science-move-hive-tables/output-hive-results-2.png)

Azure ストレージ エクスプローラーなどのツールを使用して Hadoop クラスターの既定のコンテナーを開く場合、ハイブ クエリの出力が次のように表示されます。フィルター (赤色の四角形によって示されています) を適用して、名前に指定された文字を持つ BLOB のみを取得できます。

![Create workspace](./media/machine-learning-data-science-move-hive-tables/output-hive-results-3.png)

###<a name="hive-editor"></a> 2.Hive エディターで Hive クエリを送信する

ユーザーは、クエリのコンソール (Hive エディター) を使用するために、URL

*https://&#60;Hadoop クラスター名>.azurehdinsight.net/Home/HiveEditor*

を Web ブラウザーに入力することもできます。ログインするために、Hadoop クラスターの資格情報の入力を求められます。

###<a name="ps"></a> 3.Azure PowerShell コマンドで Hive クエリを送信する

ユーザーは PowerShell を使用して Hive クエリを送信することもできます。手順については、「[PowerShell を使用して Hive ジョブを送信する](../hdinsight/hdinsight-submit-hadoop-jobs-programmatically.md#hive-powershell)」を参照してください。


## <a name="create-tables"></a>Hive データベースとテーブルの作成。

Hive クエリは、[Github リポジトリ](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_db_tbls_load_data_generic.hql)内で共有されており、そこからダウンロードできます。

Hive テーブルを作成する Hive クエリを次に示します。

    create database if not exists <database name>;
	CREATE EXTERNAL TABLE if not exists <database name>.<table name>
	(
		field1 string, 
		field2 int, 
		field3 float, 
		field4 double, 
		...,
		fieldN string
	) 
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' lines terminated by '<line separator>' 
	STORED AS TEXTFILE LOCATION '<storage location>' TBLPROPERTIES("skip.header.line.count"="1");

ユーザーが接続する必要があるフィールドとその他の構成の説明を次に示します。

- **&#60;データベース名>**: ユーザーが作成するデータベースの名前。ユーザーが既定のデータベースを使用する場合、*create database...* クエリは省略してかまいません。 
- **&#60;テーブル名>**: ユーザーが指定されたデータベース内に作成するテーブルの名前。ユーザーが既定のデータベースを使用する場合、テーブルは *&#60;テーブル名>* で直接参照でき、&#60;データベース名> は不要です。
- **&#60;フィールド区切り記号>**: Hive テーブルにアップロードするデータ ファイル内のフィールドを区切る区切り記号。 
- **&#60;行区切り記号>**: データ ファイル内の行を区切る区切り記号。 
- **&#60;ストレージの場所>**: Hive テーブルのデータを保存する Azure Sorage の場所。ユーザーが *LOCATION &#60;ストレージの場所>* を指定しなかった場合、既定では、データベースとテーブルは、Hive クラスターの既定のコンテナー内の *hive/warehouse/* ディレクトリに格納されます。ユーザーがストレージの場所を指定する場合、ストレージの場所は、データベースとテーブルの既定のコンテナー内でなければなりません。この場所は、クラスターの既定のコンテナーを基準として、*'wasb:///&#60;ディレクトリ 1>/'* や *'wasb:///&#60;directory 1>/&#60;ディレクトリ 2>/'* などの形式で参照する必要があります。クエリが実行されると、既定のコンテナー内に相対ディレクトリが作成されます。 
- **TBLPROPERTIES("skip.header.line.count"="1")**: データ ファイルにヘッダー行が含まれる場合、ユーザーはこのプロパティを *create table* クエリの**最後に**追加する必要があります。それ以外の場合、ヘッダー行はテーブルへのレコードとして読み込まれます。データ ファイルにヘッダー行が含まれない場合は、クエリでこの構成を省略することができます。 

## <a name="load-data"></a>Hive テーブルへのデータの読み込み
Hive テーブルにデータを読み込む Hive クエリを次に示します。

    LOAD DATA INPATH '<path to blob data>' INTO TABLE <database name>.<table name>;

- **&#60;BLOB データのパス>**: Hive テーブルにアップロードする BLOB ファイルが HDInsight Hadoop クラスターの既定のコンテナーに存在する場合、*&#60;BLOB データのパス>* は *'wasb:///&#60;このコンテナー内のディレクトリ>/&#60;BLOB ファイル名>'* の形式にする必要があります。BLOB ファイルは、HDInsight Hadoop クラスターの追加コンテナーに配置することもできます。この場合、*&#60;BLOB データのパス>* は *'wasb://&#60;コンテナー名>@&#60;ストレージ アカウント名>.blob.core.windows.net/&#60;BLOB ファイル名>'* の形式にする必要があります。

	>[AZURE.NOTE] Hive テーブルにアップロードする BLOB データは、Hadoop クラスターのストレージ アカウントの既定のコンテナーまたは追加のコンテナーに配置されている必要があります。それ以外の場合、*LOAD DATA* クエリはデータにアクセスできないために失敗します。


## <a name="partition-orc"></a>高度なトピック: パーティション テーブルと ORC 形式での Hive データの格納

データのサイズが大きい場合、テーブルのいくつかのパーティションのみをスキャンすればよいクエリでは、テーブルをパーティション分割すると効果的です。たとえば、Web サイトのログ データを日付別にパーティション分割することができます。

Hive テーブルをパーティション分割することに加え、Optimized Row Columnar (ORC) 形式で Hive データを格納することも効果的です。詳細については、<a href="https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-ORCFiles" target="_blank">Hive によるデータの読み取り、書き込み、および処理時における ORC ファイルを使用したパフォーマンスの向上</a>に関するページをご覧ください。

### パーティション テーブル
パーティション テーブルを作成し、そこにデータを読み込む Hive クエリを次に示します。

    CREATE EXTERNAL TABLE IF NOT EXISTS <database name>.<table name>
	(field1 string,
	...
	fieldN string
	)
    PARTITIONED BY (<partitionfieldname> vartype) ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>'
		 lines terminated by '<line separator>' TBLPROPERTIES("skip.header.line.count"="1");
	LOAD DATA INPATH '<path to the source file>' INTO TABLE <database name>.<partitioned table name> 
		PARTITION (<partitionfieldname>=<partitionfieldvalue>);

パーティション テーブルの照会時には、検索の有効性が大幅に向上するように、`where` 句の**先頭**にパーティション条件を追加することをお勧めします。

    select 
		field1, field2, ..., fieldN
	from <database name>.<partitioned table name> 
	where <partitionfieldname>=<partitionfieldvalue> and ...;

### <a name="orc"></a>ORC 形式での Hive データの格納

ユーザーは ORC 形式で格納されているデータを直接 BLOB ストレージから Hive テーブルに読み込むことはできません。以下に、ORC 形式で格納されているデータを Azure BLOB から Hive テーブルに読み込むためにユーザーが実行する必要がある手順を示します。

1. 外部テーブル **STORED AS TEXTFILE** を作成し、データを BLOB ストレージからテーブルに読み込みます。

		CREATE EXTERNAL TABLE IF NOT EXISTS <database name>.<external textfile table name>
		(
			field1 string,
			field2 int,
			...
			fieldN date
		)
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' 
			lines terminated by '<line separator>' STORED AS TEXTFILE 
			LOCATION 'wasb:///<directory in Azure blob>' TBLPROPERTIES("skip.header.line.count"="1");

		LOAD DATA INPATH '<path to the source file>' INTO TABLE <database name>.<table name>;

2. 手順 1 の外部テーブルと同じスキーマを持つ内部テーブルを、同じフィールド区切り記号を使用して作成し、ORC 形式で Hive データを格納します。

		CREATE TABLE IF NOT EXISTS <database name>.<ORC table name> 
		(
			field1 string,
			field2 int,
			...
			fieldN date
		) 
		ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' STORED AS ORC;

3. 手順 1 の外部テーブルからデータを選択し、ORC テーブルに挿入します。

		INSERT OVERWRITE TABLE <database name>.<ORC table name>
            SELECT * FROM <database name>.<external textfile table name>;

	>[AZURE.NOTE] TEXTFILE テーブル *&#60;データベース名>.&#60;外部テキストファイル テーブル名>* にパーティションが含まれている場合、手順 3 で、`SELECT * FROM <database name>.<external textfile table name>` コマンドは、返されたデータ セット内のフィールドとしてパーティション変数を選択します。*&#60;データベース名>.&#60;ORC テーブル名>* にはテーブル スキーマのフィールドとしてパーティション変数が含まれないため、*&#60;データベース名>.&#60;ORC テーブル名>* に挿入すると失敗します。この場合、ユーザーは *&#60;データベース名>.&#60;ORC テーブル名>* に挿入するフィールドを次のように具体的に選択する必要があります。

		INSERT OVERWRITE TABLE <database name>.<ORC table name> PARTITION (<partition variable>=<partition value>)
		   SELECT field1, field2, ..., fieldN
		   FROM <database name>.<external textfile table name> 
		   WHERE <partition variable>=<partition value>;

4. すべてのデータが *&#60;データベース名>.&#60;ORC テーブル名>* に挿入された後に、次のクエリを使用して *&#60;外部テキストファイル テーブル名>* を削除するのが安全です。

		DROP TABLE IF EXISTS <database name>.<external textfile table name>;

この手順が終了すれば、すぐに使用できる ORC 形式のデータを含むテーブルが手に入ったことになります。

<!---HONumber=AcomDC_0622_2016-->