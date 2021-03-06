<properties 
   pageTitle="Azure キューと Service Bus キューの比較 | Microsoft Azure"
   description="Azure によって提供される 2 種類のキューの相違点と共通点について説明します。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="tysonn" />
<tags 
   ms.service="service-bus"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="tbd"
   ms.date="11/18/2015"
   ms.author="sethm" />

# Azure キューと Service Bus キューの比較

この記事では、現在 Microsoft Azure によって提供されている Azure キューと Service Bus キューという 2 種類のキューの相違点と共通点について説明します。この情報を使用すると、それぞれのテクノロジを比較対照して、現在のニーズに最適なのはどちらのソリューションかを十分な情報に基づいて判断できるようになります。

## はじめに

Microsoft Azure では **Azure キュー**と **Service Bus キュー**の 2 種類のキュー メカニズムをサポートしています。

**Azure キュー**は、[Azure Storage](https://azure.microsoft.com/services/storage/) インフラストラクチャの一部であり、単純な REST ベースの Get/Put/Peek インターフェイスを使用して、サービス内およびサービス間で信頼性の高い永続的なメッセージングを提供します。

**Service Bus キュー**は、より大きな [Azure メッセージング](https://azure.microsoft.com/services/service-bus/) インフラストラクチャの一部です。このインフラストラクチャでは、キュー処理だけでなく、発行/サブスクライブ、Web サービスのリモート処理、統合のパターンがサポートされています。Service Bus キュー、トピック/サブスクリプション、リレーの詳細情報は、「[Service Bus メッセージングの概要](service-bus-messaging-overview.md)」をご覧ください。

この 2 つのキュー テクノロジは共存していますが、最初に Azure キューが、Azure ストレージ サービス上に構築された専用のキュー ストレージ メカニズムとして導入されました。Service Bus キューは、より大きな「ブローカー メッセージング」インフラストラクチャ上に構築されています。このインフラストラクチャは、複数の通信プロトコル、データ コントラクト、信用ドメイン、ネットワーク環境などにまたがるアプリケーションやアプリケーション コンポーネントの統合を目的としています。

この記事では、Azure によって提供されるこの 2 つのキュー テクノロジを比較するために、各テクノロジの機能の動作と実装の違いについて説明します。また、アプリケーション開発のニーズに最適な機能を選択する方法についても説明します。

## テクノロジの選択に関する考慮事項

Azure キューと Service Bus キューは、どちらも、現在 Microsoft Azure で提供されているメッセージ キュー サービスの実装です。機能に若干の違いがあるため、個々のソリューションの要件や、解決する必要があるビジネス上または技術上の問題に応じて、いずれかまたは両方を使用できます。

特定のソリューションの目的に合うキュー テクノロジを判断する場合、ソリューション設計者および開発者は次の推奨事項を検討する必要があります。詳細については、次のセクションをご覧ください。

ソリューション設計者または開発者として、次の場合に **Azure キューの使用を検討してください**。

- アプリケーションでキューに格納する必要があるメッセージの量が 80 GB を超えており、メッセージの有効期間が 7 日未満の場合。

- アプリケーションでキュー内のメッセージ処理の進行状況を追跡する必要がある場合。これは、メッセージを処理している worker がクラッシュした場合に役に立ちます。後続の worker でその情報を使用して、前の worker が中断した時点から処理を再開できます。

- キューに対して実行されたすべてのトランザクションのサーバー側のログが必要な場合。

ソリューション設計者または開発者として、次の場合に **Service Bus キューの使用を検討してください**。

- キューをポーリングせずにメッセージを受信できる必要がある場合。Service Bus では、Service Bus でサポートする TCP ベースのプロトコルを使用し、長いポーリングの受信操作を使用することによってこれを実現できます。

- キューによるメッセージの配信が先入れ先出し (FIFO) の順序で行われることが保証される必要がある場合。

- Azure と Windows Server (プライベート クラウド) 上で対称的なエクスペリエンスが必要な場合。詳細については、「[Service Bus for Windows Server](https://msdn.microsoft.com/library/dn282144.aspx)」をご覧ください。

- 自動重複検出をサポートする必要がある場合。

- アプリケーションでメッセージを実行時間の長い並列ストリームとして処理する必要がある場合 (メッセージは、自身の [SessionId](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.sessionid.aspx) プロパティを使用してストリームに関連付けられます)。このモデルでは、処理を行うアプリケーションの各ノードは、メッセージではなくストリームに対して競合します。処理を行うノードにストリームが渡されると、そのノードはトランザクションを使用してアプリケーション ストリームの状態を確認できます。

- 複数のメッセージをキューに送信したりキューから受信したりする際にトランザクション動作と原子性が必要な場合。

- アプリケーション固有のワークロードの TTL (time-to-live) が 7 日間を超える場合。

- アプリケーションで処理するメッセージのサイズが 64 KB を超えることはあっても 256 KB の制限に到達することはないと考えられる場合。

- ロールベースのアクセス モデルを提供して、キューの送信側と受信側に異なる権限/アクセス許可を与える必要がある場合。

- キューのサイズが 80 GB を超えることはない場合。

- AMQP 1.0 標準ベースのメッセージング ブローカーを使用する必要がある場合。AMQP の詳細情報については「[Service Bus AMQP の概要](service-bus-amqp-overview.md)」をご覧ください。

- いずれは、キュー ベースのポイント ツー ポイントの通信から、キューに送信されたメッセージの一部またはすべての独立したコピーを受信する追加の受信側 (サブスクライバー) をシームレスに統合できるメッセージ交換パターンに移行することを考えている場合。後者は、Service Bus によってネイティブで提供される発行/サブスクライブ機能を指します。

- 追加のインフラストラクチャ コンポーネントを作成しなくても「At-Most-Once」の配信保証をサポートできる必要がある場合。

- バッチ メッセージの発行および使用が必要な場合。

- Windows Communication Foundation (WCF) の .NET Framework 通信スタックとの完全な統合が必要な場合。

## Azure キューと Service Bus のキューの比較

次のセクションの表では、キューの機能を論理的にグループ化しており、Azure キューと Service Bus キューの両方で使用できる機能を一目で比較することができます。

## 基本的な機能

このセクションでは、Azure キューと Service Bus キューで提供される基本的なキュー機能の一部を比較します。

|比較条件|Azure キュー|Service Bus キュー|
|---|---|---|
|順序の保証|**いいえ** <br/><br>詳細については、追加情報セクションの最初のメモをご覧ください。</br>|**はい - 先入れ先出し (FIFO)**<br/><br> (メッセージング セッションを使用)|
|配信保証|**At-Least-Once**|**At-Least-Once**<br/><br/>**At-Most-Once**|
|分割不可能な操作のサポート|**いいえ**|**はい**<br/><br/>|
|受信動作|**非ブロッキング**<br/><br/>(新しいメッセージがない場合はすぐに完了します)|**ブロッキング (タイムアウトあり/なし)**<br/><br/>(長いポーリング ("[Comet 手法](http://go.microsoft.com/fwlink/?LinkId=613759)") を提供)<br/><br/>**非ブロッキング**<br/><br/>(.NET マネージ API のみを使用)|
|プッシュ型 API|**いいえ**|**はい**<br/><br/>[OnMessage](https://msdn.microsoft.com/library/azure/jj908682.aspx) と **OnMessage セッション** .NET API|
|受信モード|**Peek & Lease**|**Peek & Lock**<br/><br/>**Receive & Delete**|
|排他アクセス モード|**リース ベース**|**ロック ベース**|
|リース/ロックの期間|**30 秒 (既定)**<br/><br/>**7 日間 (最大)** ([UpdateMessage](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.updatemessage.aspx) API を使用してメッセージ リースを更新または変更できます)|**60 秒 (既定)**<br/><br/>[RenewLock](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.renewlock.aspx) API を使用してメッセージのロックを更新できます。|
|リース/ロックの粒度|**メッセージ レベル**<br/><br/>(各メッセージには異なるタイムアウト値を設定できます。この値は [UpdateMessage](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.updatemessage.aspx) API を使用して、メッセージの処理中に必要に応じて更新できます)|**キュー レベル**<br/><br/>(各キューのすべてのメッセージに同じロック粒度が適用されますが、[RenewLock](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.renewlock.aspx) API を使用してロックを更新できます)|
|一括受信|**はい**<br/><br/>(メッセージを取得するときに最大 32 のメッセージ数を明示的に指定します)|**はい**<br/><br/>(プレフェッチ プロパティを有効にして暗黙的に行うか、トランザクションを使用して明示的に行います)|
|一括送信|**いいえ**|**はい**<br/><br/>(トランザクションまたはクライアント側のバッチ処理を使用)|

### 追加情報

- Azure キューのメッセージは一般的に先入れ先出しですが、メッセージの表示タイムアウト期限を過ぎたとき (たとえば処理中にクライアント アプリケーションがクラッシュした場合) などに順番が変わることがあります。表示タイムアウトが過ぎると、別の worker がデキューするために、メッセージがキューに再度表示されます。そのとき、もともとエンキュー時に後ろにあったメッセージの後にメッセージが (再デキューのために) 配置されることがあります。

- Azure Storage の BLOB またはテーブルを既に使用している場合にキューの使用を開始すると、99.9% の可用性が保証されます。BLOB またはテーブルを Service Bus キューと使用する場合は、可用性が低下します。

- Service Bus キューで FIFO パターンを保証するには、メッセージング セッションを使用する必要があります。**Peek & Lock** モードで受信したメッセージの処理中にアプリケーションがクラッシュした場合、キューの受信側は、次にメッセージング セッションを受け取ったときに、TTL (time-to-live) 期間が経過した後、失敗したメッセージから開始します。

- Azure キューは、アプリケーション コンポーネントを分離してスケーラビリティや耐障害性を向上させ、負荷平準化やプロセス ワークフロー構築を容易にするなどの、標準的なキュー シナリオをサポートするように設計されています。

- Service Bus キューは、*At-Least-Once* の配信保証をサポートしています。また、セッション状態を使用してアプリケーションの状態を格納し、トランザクションを使用してメッセージの受信とセッション状態の更新の原子性を確保することにより、*At-Most-Once* セマンティクスをサポートすることもできます。Azure Workflow Service では、この方法を使用して At-Most-Once の配信を保証しています。

- Azure キューでは、開発者とオペレーション チームの双方に対し、キュー、テーブル、BLOB で一貫性のあるプログラミング モデルを提供します。

- Service Bus キューは、1 つのキューのコンテキストでローカル トランザクションをサポートします。

- Service Bus でサポートされている *Receive and Delete* モードを使用すると、メッセージング操作の数 (および関連するコスト) を削減できますが、配信の確実性が低下します。

- Azure キューではメッセージのリースを延長することのできるリースを提供します。これにより、メッセージのリースを短くして、worker がクラッシュした場合にすぐに別の worker がメッセージを再度処理できるようにしたり、メッセージで現在のリース時間より長い処理時間が必要になった場合にメッセージのリースを延長したりすることができます。

- Azure キューでは、メッセージをエンキューまたはデキューするときに表示のタイムアウトを設定できます。また、実行時にメッセージを別のリース値で更新したり、同じキューのメッセージを別の値で更新したりすることもできます。Service Busのロックのタイムアウトはキューのメタデータで定義されていますが、[RenewLock](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.renewlock.aspx) メソッドを呼び出してロックを更新できます。

- Service Bus キューのブロッキング受信操作の最大タイムアウトは 24 日です。ただし、REST ベースのタイムアウトの最大値は 55 秒です。

- Service Bus でサポートされているクライアント側のバッチ処理を使用すると、キュー クライアントで複数のメッセージをバッチ処理して 1 回の送信操作で処理を完了できます。バッチ処理は非同期送信操作でのみ使用できます。

- 200 TB が上限の Azure キュー (アカウントを仮想化すればさらに確保可能) や無制限キューのような機能により、Azure は SaaS プロバイダーにとって理想的なプラットフォームになります。

- Azure キューでは、柔軟性が高くパフォーマンスに優れた委任アクセス制御メカニズムを提供します。

## 高度な機能

このセクションでは、Azure キューと Service Bus キューで提供される高度な機能を比較します。

|比較条件|Azure キュー|Service Bus キュー|
|---|---|---|
|スケジュールされた配信|**はい**|**はい**|
|自動的な配信不能レタリング|**いいえ**|**はい**|
|キューの有効期間の増加|**はい**<br/><br/>(表示のタイムアウトのインプレース更新を使用)|**はい**<br/><br/>(専用の API 関数を使用)|
|有害なメッセージのサポート|**はい**|**はい**|
|インプレース更新|**はい**|**はい**|
|サーバー側のトランザクション ログ|**はい**|**いいえ**|
|Storage のメトリック|**はい**<br/><br/>**分単位のメトリック**: 可用性、TPS、API 呼び出し数、エラー数、その他のメトリックをすべてリアルタイムで提供します (分単位で集計され、運用環境での発生から数分以内にレポートされます)。詳細については、「[Storage 分析 メトリックスについて](https://msdn.microsoft.com/library/azure/hh343258.aspx)」を参照してください。|**はい**<br/><br/>([GetQueues](https://msdn.microsoft.com/library/azure/hh293128.aspx) の呼び出しによる一括クエリ)|
|状態管理|**いいえ**|**はい**<br/><br/>[Microsoft.ServiceBus.Messaging.EntityStatus.Active](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.entitystatus.aspx)、[Microsoft.ServiceBus.Messaging.EntityStatus.Disabled](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.entitystatus.aspx)、[Microsoft.ServiceBus.Messaging.EntityStatus.SendDisabled](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.entitystatus.aspx)、[Microsoft.ServiceBus.Messaging.EntityStatus.ReceiveDisabled](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.entitystatus.aspx)|
|メッセージの自動転送|**いいえ**|**はい**|
|キューの消去機能|**はい**|**いいえ**|
|メッセージ グループ|**いいえ**|**はい**<br/><br/>(メッセージング セッションを使用)|
|メッセージ グループ単位のアプリケーション状態|**いいえ**|**はい**|
|重複検出|**いいえ**|**はい**<br/><br/>(送信側で構成可能)|
|WCF の統合|**いいえ**|**はい**<br/><br/>(すぐに使用できる WCF バインドが用意されています)|
|WF の統合|**カスタム**<br/><br/>(カスタム WF アクティビティを作成する必要があります)|**ネイティブ**<br/><br/>(すぐに使用できる WF アクティビティが用意されています)|
|メッセージ グループの参照|**いいえ**|**はい**|
|ID によるメッセージ セッションの取得|**いいえ**|**はい**|

### 追加情報

- 両方のキュー テクノロジには、メッセージを後で配信するようにスケジュールできる機能があります。

- キューの自動転送を使用すると、数千のキューのメッセージを 1 つのキューに自動転送して、受信側アプリケーションはそのキューからメッセージを処理することができます。このメカニズムを使用して、各メッセージ パブリッシャー間でセキュリティの確保、フロー制御、ストレージ分離を実現できます。

- Azure キューでは、メッセージの内容の更新がサポートされています。この機能を使用すると、状態情報と進行状況の増分更新をメッセージに永続化して、メッセージの処理を最初からではなく前回のチェックポイントから開始することができます。Service Bus キューでは、メッセージ セッションを使用することで同じ機能を実現できます。セッションでは、アプリケーションの処理状態を保存および取得できます ([SetState](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagesession.setstate.aspx) および [GetState](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagesession.getstate.aspx) を使用)。

- Service Bus キューでのみサポートされている配信不能レタリングは、受信側のアプリケーションで正しく処理できないメッセージを分離する場合や、TTL (time-to-live) プロパティが期限切れになったためにメッセージが宛先に届かない場合に役立ちます。TTL の値は、メッセージがキューに保持される期間を指定します。Service Bus では、TTL が期限切れになると、メッセージが $DeadLetterQueue という特殊なキューに移動されます。

- Azure キューで「有害な」メッセージを検出するには、アプリケーションでメッセージをデキューするときにメッセージの DequeueCount プロパティを確認します。DequeueCount が一定のしきい値を超えている場合は、そのメッセージをアプリケーション定義の「配信不能」キューに移動します。

- Azure キューでは、キューに対して実行されたすべてのトランザクションの詳細なログとメトリックの集計値を取得できます。これらのオプションはいずれも、デバッグや、アプリケーションでの Azure キューの使い方を理解するのに役立ちます。また、アプリケーションのパフォーマンス チューニングやキューの使用コストの削減にも役立ちます。

- Service Bus でサポートされている「メッセージ セッション」の概念を使用すると、特定の論理グループに属するメッセージを特定の受信側に関連付けて、メッセージとその受信側の間にセッションのような関係を作成できます。Service Bus でこの高度な機能を有効にするには、メッセージの [SessionID](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.sessionid.aspx) プロパティを設定します。受信側では、特定のセッション ID をリッスンして、指定したセッション ID を共有するメッセージを受信できます。

- Service Bus キューでサポートされている重複検出機能は、[MessageID](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx) プロパティの値に基づいて、キューまたはトピックに送信された重複するメッセージを自動的に削除します。

## 容量およびクォータ

このセクションでは、容量および適用可能なクォータの観点から Azure キューと Service Bus キューを比較します。

|比較条件|Azure キュー|Service Bus キュー|
|---|---|---|
|最大キュー サイズ|**200 TB**<br/><br/>(1 つのストレージ アカウントの容量)|**1 GB ～ 80 GB**<br/><br/>(キューの作成時と [パーティション分割を有効化](service-bus-partitioning.md)するときに定義します。追加情報セクションをご覧ください)|
|最大メッセージ サイズ|**64 KB**<br/><br/>(**Base64** エンコードを使用する場合は 48 KB)<br/><br/>Azure では、キューと BLOB を組み合わせることでサイズの大きいメッセージをサポートし、1 つのアイテムに対して最大 200 GB までのメッセージをエンキューできます。|**256 KB** または **1 MB**<br/><br/> (ヘッダーと本文の両方を含む。ヘッダーの最大サイズは 64 KB)<br/><br/>[サービス レベル](service-bus-premium-messaging.md)により異なる。|
|メッセージの最大 TTL|**7 日**|**無制限**|
|キューの最大数|**無制限**|**10,000**<br/><br/>(サービス名前空間あたり。増やすことができます)|
|同時クライアントの最大数|**無制限**|**無制限**<br/><br/>(最大 100 の同時接続数の制限は TCP プロトコル ベースの通信にのみ適用されます)|

### 追加情報

- Service Bus では、キューのサイズが制限されます。キューの最大サイズは、キューの作成時に 1 ～ 80 GB の値を指定できます。キューの作成時に設定したキュー サイズの値に達すると、その後の受信メッセージは拒否され、呼び出し元のコードが例外を受け取ります。Service Bus でのクォータの詳細情報については、「[Service Bus のクォータ](service-bus-quotas.md)」をご覧ください。

- Service Bus キューは、1 GB、2 GB、3 GB、4 GB、5 GB で作成できます (既定値は 1 GB)。パーティション分割を有効にすると (既定)、Service Bus は指定した各 GB あたりに 16 個のパーティションを作成できます。そのため、5 GB のキューを作成すると、16 個のパーティションで、キューの最大サイズは (5 * 16) = 80 GB になります。パーティション分割したキューまたはトピックの最大サイズは、[Azure クラシック ポータル][]の各エントリで確認できます。

- Azure キューでは、内容が XML セーフでないメッセージに対しては **Base64** エンコードを使用する必要があります。メッセージを **Base64** エンコードを使用する場合、ユーザー ペイロードの上限は 64 KB ではなく 48 KB になります。

- Service Bus キューでは、キューに格納される各メッセージはヘッダーおよび本文で構成されます。メッセージの合計サイズが、サービス レベルでサポートされる最大メッセージ サイズを超えることはできません。

- クライアントが TCP プロトコルで Service Bus キューと通信する場合は、1 つの Service Bus キューに対する同時接続の最大数が 100 に制限されます。この数は送信側と受信側で共有されます。このクォータに達すると、その後の接続要求は拒否され、呼び出し元のコードが例外を受け取ります。この制限は、REST ベースの API を使用してキューに接続するクライアントには適用されません。

- 1 つの Service Bus Service の名前空間で 10,000 を超える数のキューが必要な場合は、Azure サポート チームに連絡してキューの数を増やすことができます。Service Bus でキューの数を 10,000 より多くするために、[Azure クラシック ポータル][]を使用して追加の名前空間を作成することもできます。

## 管理と操作

このセクションでは、Azure キューと Service Bus キューで提供される管理機能を比較します。

|比較条件|Azure キュー|Service Bus キュー|
|---|---|---|
|管理プロトコル|**HTTP/HTTPS 経由の REST**|**HTTPS 経由の REST**|
|ランタイム プロトコル|**HTTP/HTTPS 経由の REST**|**HTTPS 経由の REST**<br/><br/>**AMQP 1.0 Standard (TCP と TLS)**|
|.NET マネージ API|**はい**<br/><br/>(.NET 管理対象 Storage クライアント API)|**はい**<br/><br/>(.NET の仲介型メッセージング API)|
|ネイティブ C++|**はい**|**いいえ**|
|Java API|**はい**|**はい**|
|PHP API|**はい**|**はい**|
|Node.js API|**はい**|**はい**|
|任意のメタデータのサポート|**はい**|**いいえ**|
|キューの名前付け規則|**最大 63 文字**<br/><br/>(キューの名前は小文字で指定する必要があります)|**最大 260 文字**<br/><br/>(キューのパスと名前では大文字と小文字は区別されません)|
|キューの長さを取得する機能|**はい**<br/><br/>(メッセージが TTL を過ぎて期限切れになっても削除されていない場合は概算値となります)|**はい**<br/><br/>(特定の時点の正確な値)|
|Peek 機能|**はい**|**はい**|

### 追加情報

- Azure キューでは、キューの説明に名前と値のペアの形式の任意の属性を適用できます。

- 両方のキュー テクノロジには、メッセージをロックせずに表示できる機能も用意されています。この機能は、キューのエクスプローラー/ブラウザー ツールを実装する際に便利です。

- Service Bus の .NET ブローカー メッセージング API は、全二重 TCP 接続を使用して HTTP 経由の REST より高いパフォーマンスを発揮します。また、AMQP 1.0 標準プロトコルもサポートしています。

- Azure のキュー名は 3 ～ 63 文字の間で指定でき、小文字、数字、ハイフンを使用できます。詳細については、「[キューおよびメタデータの名前付け](https://msdn.microsoft.com/library/azure/dd179349.aspx)」をご覧ください。

- Service Bus のキュー名は最大 260 文字までの長さで指定でき、名前付け規則の制限は厳しくありません。Service Bus のキュー名には、文字、数字、ピリオド (.)、ハイフン (-)、アンダースコア (\_) を使用できます。

## パフォーマンス

このセクションでは、パフォーマンスの観点から Azure キューと Service Bus キューを比較します。

|比較条件|Azure キュー|Service Bus キュー|
|---|---|---|
|最大スループット|**2,000 メッセージ/秒**<br/><br/>(1 KB のメッセージによるベンチマークに基づく値)|**2,000 メッセージ/秒**<br/><br/>(1 KB のメッセージによるベンチマークに基づく値)|
|平均待機時間|**10 ms**<br/><br/>([TCP Nagle](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/06/25/nagle-s-algorithm-is-not-friendly-towards-small-requests.aspx) を無効にした場合)|**20 ～ 25 ms**|
|調整動作|**HTTP 503 コードによる拒否**<br/><br/>(調整された要求は課金可能な操作として扱われません)|**例外/HTTP 503 による拒否**<br/><br/>(調整された要求は課金可能な操作として扱われません)|

### 追加情報

- 1 つの Azure キューで 1 秒間に最大 2,000 トランザクションを処理できます。トランザクションとは、**Put**、**Get**、**Delete** のいずれかの操作です。キューに 1 つのメッセージを送信する操作 (**Put**) は 1 つのトランザクションとしてカウントされますが、メッセージを受信する操作は、多くの場合、メッセージを取得する操作 (**Get**) とその後にメッセージをキューから削除する操作 (**Delete**) から成る 2 段階のプロセスです。そのため、成功したデキュー操作には 2 つのトランザクションが含まれるのが一般的です。複数のメッセージを一括取得すると、この影響を軽減できます。この場合、1 つのトランザクションで最大 32 メッセージの **Get** 操作を実行し、その後、各メッセージに **Delete** 操作を実行できます。スループットを改善するために、複数のキューを作成することができます (1 つのストレージ アカウントで作成できるキューの数は無制限です)。

- アプリケーションが Azure キューの最大スループットに達すると、通常、Queue サービスから HTTP 503 (サーバーがビジー状態) 応答が返されます。この場合、アプリケーションで、指数関数的なバックオフ遅延を使用する再試行ロジックを開始する必要があります。

- ストレージ アカウントと同じ場所 (リージョン) にあるホストされるサービスからの小さなメッセージ (10 KB 未満) を処理する場合の Azure キューの待機時間は平均 10 ミリ秒です。

- Azure キューとService Bus キューの両方で、要求の拒否による調整動作が適用されます。ただし、どちらのキューでも、調整された要求は課金可能な操作として扱われません。

- Service Bus キューに対するベンチマークによると、1 つのキューのメッセージ スループットは、約 1 KB のメッセージで最大 2,000 メッセージ/秒です。より高いスループットを達成するには複数のキューを使用します。Service Bus によるパフォーマンスの最適化の詳細については、「[Service Bus の仲介型メッセージングを使用したパフォーマンス向上のためのベスト プラクティス](service-bus-performance-improvements.md)」をご覧ください。

- Service Bus キューが最大スループットに達すると、キューが調整されていることを示す [ServerBusyException](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.serverbusyexception.aspx) (.NET ブローカー メッセージング API を使用している場合) または HTTP 503 (REST ベースの API を使用している場合) の応答がキュー クライアントに返されます。

## 認証と権限承認

このセクションでは、Azure キューと Service Bus キューでサポートされる認証および承認機能について説明します。

|比較条件|Azure キュー|Service Bus キュー|
|---|---|---|
|認証|**対称キー**|**対称キー**|
|セキュリティ モデル|SAS トークンを介した委任アクセス。|SAS|
|ID プロバイダー フェデレーション|**いいえ**|**はい**|

### 追加情報

- どちらのキュー テクノロジでも、すべての要求を認証する必要があります。匿名アクセスを使用するパブリック キューはサポートされていません。SAS を使用すると、書き込み専用 SAS、読み取り専用 SAS、フルアクセス SAS を発行することで、このシナリオに対応できます。

- Azure キューによって提供される認証方式では対称キーが使用されます。対称キーとは、SHA-256 アルゴリズムを使用してコンピューティングされ、**Base64 **文字列としてエンコードされるハッシュ ベースのメッセージ認証コード (HMAC) です。各プロトコルの詳細情報については、「[Azure ストレージ サービスの認証](https://msdn.microsoft.com/library/azure/dd179428.aspx)」をご覧ください。Service Bus キューでは、対称キーを使用する類似のモデルをサポートします。詳細については、「[Service Bus での共有アクセス署名認証](service-bus-shared-access-signature-authentication.md)」をご覧ください。

## コスト

このセクションでは、コストの観点から Azure キューと Service Bus キューを比較します。

|比較条件|Azure キュー|Service Bus キュー|
|---|---|---|
|キュー トランザクションのコスト|**$0.0036**<br/><br/>(100,000 トランザクションあたり)|**Basic レベル**: **$0.05**<br/><br/>(100 万回の処理あたり)|
|課金可能な操作|**すべて**|**送信/受信のみ**<br/><br/>(他の操作には料金はかかりません)|
|アイドル状態のトランザクション|**課金対象**<br/><br/>(空のキューに対するクエリは課金可能なトランザクションと見なされます)|**課金対象**<br/><br/>(空のキューに対する受信は課金可能なメッセージと見なされます)|
|Storage コスト|**$0.07**<br/><br/>(GB/月あたり)|**$0.00**|
|送信データ転送のコスト|**$0.12～$0.19**<br/><br/>(地理的条件による)|**$0.12～$0.19**<br/><br/>(地理的条件による)|

### 追加情報

- データ転送の料金は、特定の課金期間に Azure データ センターからインターネット経由で送信されたデータの総量に基づいて決まります。

- 同じリージョンにある Azure サービス間のデータ転送には料金はかかりません。

- このドキュメントの作成時点では、受信データ転送には料金はかかりません。

- Service Bus キューでは長いポーリングがサポートされているため、待機時間の短い配信が必要な状況でこのキューを使用すると、コスト効率が向上する可能性があります。

>[AZURE.NOTE] すべてのコストは、変更されることがあります。上記の表は、このドキュメントの作成時点の価格を反映しています。ここには、現在利用できるキャンペーン プランは含まれていません。Azure の最新の料金情報については、「[Azure の料金](https://azure.microsoft.com/pricing/)」ページをご覧ください。Service Bus の料金の詳細については、「[Service Bus 料金](https://azure.microsoft.com/pricing/details/service-bus/)」を参照してください。

## まとめ

2 つのテクノロジをより深く理解することにより、使用するキュー テクノロジとその状況について、より多くの十分な情報を得たうえでの決定を行うことができます。Azure キューと Service Bus キューを使用する状況についての判断は明らかにさまざまな要因に依存します。それらの要因がアプリケーションとそのアーキテクチャの個々のニーズに大きく依存している場合もあります。アプリケーションで既に Microsoft Azure のコア機能を使用している場合 (特にサービス間の基本的な通信およびメッセージングや、80 GB を超えるサイズのキューが必要な場合) は、Azure キューを選択できます。

Service Bus キューには高度な機能が数多く用意されているため (セッション、トランザクション、重複検出、自動的な配信不能レタリング、持続性のある発行/サブスクライブの機能など)、ハイブリッド アプリケーションを作成する場合や、アプリケーションでこれらの機能が必要な場合は、Service Bus キューを選択できます。

## 次のステップ

次の記事では、Azure キューや Service Bus キューの使用に関する詳細情報を提供します。

- [Service Bus キューの使用方法](service-bus-dotnet-get-started-with-queues.md)
- [キュー Storage Service を使用する方法](../storage/storage-dotnet-how-to-use-queues.md)
- [Service Bus のブローカー メッセージングを使用したパフォーマンス向上のためのベスト プラクティス](service-bus-performance-improvements.md)
- [Introducing Queues and Topics in Azure Service Bus (Azure Service Bus のキューとトピックの概要)](http://www.code-magazine.com/article.aspx?quickid=1112041)
- [The Developer's Guide to Service Bus (Service Bus の開発者向けガイド)](http://www.cloudcasts.net/devguide/)
- ["Azure Tables and Queues Deep Dive (Azure のテーブルとキューの詳細)"](http://www.microsoftpdc.com/2009/SVC09)
- [Azure Storage のアーキテクチャ (ブログの投稿)](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)
- [Using the Queuing Service in Azure (Azure でのキュー サービスの使用)](http://www.developerfusion.com/article/120197/using-the-queuing-service-in-windows-azure/)
- [Azure Storage の課金について - 帯域幅、トランザクション、容量 (ブログの投稿)](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/07/09/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity.aspx)

[Azure クラシック ポータル]: http://manage.windowsazure.com
 

<!---HONumber=AcomDC_0608_2016-->