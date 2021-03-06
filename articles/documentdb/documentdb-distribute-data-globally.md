<properties
   pageTitle="DocumentDB を使用したデータのグローバル分散 | Microsoft Azure"
   description="完全に管理された NoSQL データベース サービスである Azure DocumentDB のグローバル データベースを使用した、地球規模の geo レプリケーション、フェールオーバー、データ復旧について説明します。"
   services="documentdb"
   documentationCenter=""
   authors="kiratp"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="documentdb"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="06/14/2016"
   ms.author="kipandya"/>
   
   
# DocumentDB を使用したデータのグローバル分散

Azure DocumentDB は、グローバルに分散された数百万のデバイスで構成される IoT アプリケーションのほか、世界中のユーザーに対して高い応答性を実現するインターネット規模のアプリケーションのニーズを満たすように設計されています。これらのデータベース システムには、明確に定義されたデータの一貫性と可用性を保証すると共に、複数の地理的リージョンからアプリケーションのデータに短い待機時間でアクセスできるようにするという課題があります。グローバルに分散したデータベース システムとして、DocumentDB はデータのグローバル分散を容易にします。そのために、対応する保証と共に一貫性、可用性、パフォーマンスの間の明確なトレードオフを提供する、完全に管理された複数リージョンのデータベース アカウントが用意されています。DocumentDB データベース アカウントには、高可用性、10 ミリ秒未満の遅延、複数の[明確に定義された一貫性レベル][consistency]、マルチホーミング API による透過的な地域フェールオーバー、世界規模でスループットとストレージを柔軟にスケールする機能が備わっています。

  
## 複数リージョンのアカウントの構成

Azure ポータルを使えば、1 分足らずで DocumentDB アカウントを構成して世界規模のスケールを行えます。必要なのは、サポートされている複数の明確に定義された一貫性レベルから適切な一貫性レベルを選択し、対象のデータベース アカウントに任意の数の Azure リージョンを関連付けることだけです。DocumentDB の一貫性レベルでは、特定の一貫性の保証とパフォーマンスの間の明確なトレードオフが提供されます。

![DocumentDB offers multiple, well defined (relaxed) consistency models to choose from][1]

DocumentDB には明確に定義された (緩やかな) 一貫性モデルが複数用意されており、その中から 1 つを選択できます。

適切な一貫性レベルは、アプリケーションに必要なデータの一貫性の保証に応じて選択します。DocumentDB によって、指定したすべてのリージョン間でデータが自動的にレプリケートされます。また、データベース アカウントに関して選択した一貫性が保証されます。


## 複数リージョンのフェールオーバーの使用 

Azure DocumentDB には複数の Azure リージョン間でデータベース アカウントを透過的にフェールオーバーする機能があります。新しい[マルチホーミング API][developingwithmultipleregions] によって、アプリが継続して論理エンドポイントを使用でき、フェールオーバーによって中断されないことが保証されます。フェールオーバーの制御は自身で行います。アプリケーション、インフラストラクチャ、サービス、またはリージョンの (実際またはシミュレーション上の) 障害など、考えられるさまざまな障害状態のいずれかが発生した場合にデータベース アカウントを変更できる柔軟性が確保されています。DocumentDB でリージョンの障害が発生した場合、サービスによってデータベース アカウントが透過的にフェールオーバーされ、アプリケーションは可用性を損なわれることなく引き続きデータにアクセスします。DocumentDB は[可用性が 99.99% の SLA][sla] を提供しますが、[プログラム][arm]と Azure ポータルの両方で、リージョンの障害をシミュレートしてアプリケーションのエンド ツー エンドの可用性プロパティをテストできます。


## 世界規模のスケーリング
DocumentDB を利用すると、対象のデータベース アカウントに関連付けられているすべてのリージョン間でグローバルに、あらゆる規模で DocumentDB コレクションごとに独立してスループットをプロビジョニングし、ストレージを使用できます。DocumentDB コレクションは自動的にグローバルに分散され、データベース アカウントに関連付けられたすべてのリージョン間で管理されます。データベース アカウント内のコレクションは、[DocumentDB サービスが利用できる][serviceregions]任意の Azure リージョン間で分散できます。

各 DocumentDB コレクションの購入済みスループットと使用済みストレージは、自動的にすべてのリージョン間で等しくプロビジョニングされます。これにより、[支払いの対象を各時間内で使用されるスループットとストレージのみに抑えながら][pricing]、アプリケーションを世界規模でシームレスにスケールできます。たとえば、ある DocumentDB コレクションについて 200 万 RU をプロビジョニングした場合、データベース アカウントに関連付けられているリージョンはそれぞれ、このコレクションに関して 200 万 RU を受け取ります。その例を次に示します。

![Scaling throughput for a DocumentDB collection across four regions][2]

DocumentDB では、読み取りについては 10 ミリ秒未満、書き込みについては 15 ミリ秒未満の待機時間が P99 で保証されます。読み取り要求はデータセンターの境界をまたぐことがないため、[選択した一貫性の要件が保証されます][consistency]。書き込みは常に、クライアントが認識する前にローカルでクォーラムによりコミットされます。各データベース アカウントは、書き込みリージョンの優先順位を使用して構成されます。指定した優先順位が最も高いリージョンは、アカウントの現在の書き込みリージョンとして動作します。データベース アカウントの書き込みは、すべての SDK によって現在の書き込みリージョンに透過的にルーティングされます。

最後に、DocumentDB は完全に[スキーマに依存しない][vldb]ため、複数のデータセンター間のスキーマまたはセカンダリ インデックスの管理/更新について心配する必要がありません。アプリケーションとデータ モデルの開発を進めながら、[SQL クエリ][sqlqueries]は引き続き使用できます。


## グローバル分散の有効化 

DocumentDB データベース アカウントに関連付ける Azure リージョンを 1 つにするか複数にするかによって、データをローカルに分散するかグローバルに分散するかを決定できます。任意のタイミングでデータベース アカウントに対してリージョンの追加または削除を行って、データをグローバルに分散するか単一のリージョンに制限するかを決定できます。複数リージョンの割り当てがサポートされる DocumentDB データベース アカウントは、[DocumentDB - 複数リージョンのデータベース アカウント] を選択して、Azure Marketplace を通じて作成できます。



## 次のステップ

DocumentDB を使用したデータのグローバル分散の詳細については、次の記事を参照してください。

* [コレクションのスループットとストレージのプロビジョニング][throughputandstorage]
* [REST、.NET SDK、Java SDK、Python SDK、Node SDK を使用したマルチホーミング API][developingwithmultipleregions]
* [DocumentDB の一貫性レベル][consistency]
* [可用性の SLA][sla]
* [データベース アカウントの管理][manageaccount]

[1]: ./media/documentdb-distribute-data-globally/consistency-tradeoffs.png
[2]: ./media/documentdb-distribute-data-globally/collection-regions.png

<!--Reference style links - using these makes the source content way more readable than using inline links-->
[pcolls]: https://azure.microsoft.com/documentation/articles/documentdb-partition-data/
[consistency]: https://azure.microsoft.com/documentation/articles/documentdb-consistency-levels/
[consistencytradeooffs]: ./documentdb-consistency-levels/#consistency-levels-and-tradeoffs
[developingwithmultipleregions]: https://azure.microsoft.com/documentation/articles/documentdb-developing-with-multiple-regions/
[createaccount]: https://azure.microsoft.com/documentation/articles/documentdb-create-account/
[manageaccount]: https://azure.microsoft.com/documentation/articles/documentdb-manage-account/
[manageaccount-consistency]: https://azure.microsoft.com/documentation/articles/documentdb-manage-account/#consistency
[manageaccount-addregion]: https://azure.microsoft.com/documentation/articles/documentdb-manage-account/#addregion
[throughputandstorage]: https://azure.microsoft.com/documentation/articles/documentdb-manage/
[arm]: https://azure.microsoft.com/documentation/articles/documentdb-automation-resource-manager-cli/
[regions]: https://azure.microsoft.com/regions/
[serviceregions]: https://azure.microsoft.com/regions/#services
[pricing]: https://azure.microsoft.com/pricing/details/documentdb/
[sla]: https://azure.microsoft.com/support/legal/sla/documentdb/
[vldb]: http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf
[sqlqueries]: https://azure.microsoft.com/documentation/articles/documentdb-sql-query/

<!---HONumber=AcomDC_0615_2016-->