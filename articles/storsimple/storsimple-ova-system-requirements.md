<properties
   pageTitle="StorSimple Virtual Array のシステム要件"
   description="StorSimple Virtual Array のソフトウェア要件とネットワーク要件について説明します。"
   services="storsimple"
   documentationCenter="NA"
   authors="alkohli"
   manager="carmonm"
   editor=""/>

<tags
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="04/28/2016"
   ms.author="alkohli"/>

# StorSimple Virtual Array のシステム要件

## 概要

この記事では、Microsoft Azure StorSimple Virtual Array (StorSimple オンプレミス仮想デバイスまたは StorSimple 仮想デバイスとも呼ばれます) とアレイにアクセスするストレージ クライアントの重要なシステム要件について説明します。StorSimple システムをデプロイする前に以下の情報を慎重に確認します。さらにデプロイと後続の操作中にも必要に応じて以下の情報を参照することをお勧めします。

システム要件は次のとおりです。

-   **ストレージ クライアントのソフトウェア要件** - サポートされている仮想化プラットフォーム、Web ブラウザー、iSCSI イニシエーター、SMB クライアント、仮想デバイスの最小要件、オペレーティング システムの追加要件について説明します。

-   **、StorSimple デバイスのネットワーク要件** - iSCSI、クラウド、または管理トラフィックを使用できるようにするため、ファイアウォールで開いておく必要があるポートについての情報を提供します。

この記事に記載されている StorSimple のシステム要件の情報は、StorSimple Virtual Array にのみ適用されます。

- 8000 シリーズ デバイスについては、「[StorSimple ソフトウェア、高可用性、ネットワークの要件](storsimple-system-requirements.md)」をご覧ください。
 
- 7000 シリーズ デバイスについては、「[StorSimple System Requirements](http://onlinehelp.storsimple.com/1_StorSimple_System_Requirements)」をご覧ください。


## ソフトウェア要件

ソフトウェア要件には、サポートされている Web ブラウザー、SMB のバージョン、仮想化プラットフォーム、仮想デバイスの最小要件に関する情報が含まれています。

### サポートされている仮想化プラットフォーム


| **ハイパーバイザー** | **バージョン** |
|----------------|--------------------------------------|
| Hyper-V | Windows Server 2008 R2 SP1 以降 |
| VMware ESXi | 5\.5 以降 |

### 仮想デバイスの要件

| **コンポーネント** | **要件** |
|----------------------------------------------|----------------------------|
| 最小仮想プロセッサ (コア) 数 | 4 |
| 最小メモリ (RAM) | 8 GB |
| ディスク領域<sup>1</sup> | OS ディスク - 80 GB <br></br>データ ディスク - 500 GB ～ 8 TB|
| 最小ネットワーク インターフェイス数 | 1 |
| 最小インターネット帯域幅<sup>2</sup> | 5 Mbps |

<sup>1</sup> - 仮想プロビジョニング対応

<sup>2</sup> - ネットワーク要件は、日次データ変化率によって異なる場合があります。たとえば、デバイスで 1 日に 10 GB 以上の変更をバックアップする必要がある場合、5 Mbps 接続での日次バックアップには最大 4.25 時間かかる可能性があります (データの圧縮も重複除去もできなかった場合)。

### サポートされている Web ブラウザー

| **コンポーネント** | **バージョン** | **その他の要件/注意事項** |
|-------------------|-----------------|-----------------------------------|
| Microsoft Edge | 最新バージョン | |
| Internet Explorer | 最新バージョン | Internet Explorer 11 でテスト済み |
| Google Chrome | 最新バージョン | Chrome 46 でテスト済み |

### サポートされている SMB のバージョン

| **バージョン** |
|-------------|
| SMB 2.x |
| SMB 3.0 |
| SMB 3.02 |

## ネットワーク要件 

iSCSI、SMB、クラウド、または管理トラフィックを許可するためにファイアウォールで開く必要があるポートを次の表に示します。この表では、*イン*または*受信*はデバイスにアクセスするクライアント要求が入ってくる方向を意味します。*アウト*または*送信*は StorSimple デバイスがデプロイを超えて外部に (たとえば、インターネットに) データを送信する方向を意味します

| **ポート番号<sup>1</sup>** | **インまたはアウト** | **ポート範囲** | **必須** | **メモ** |
|--------------------------|---------------|----------------|---------------------------|----------------------------------------------------------------------------------------------------------------------|
| TCP 80 (HTTP) | アウト | WAN | いいえ | 送信ポートは、更新プログラムを取得するためのインターネット アクセスに使用します。<br></br>送信 Web プロキシは、ユーザーが構成できます。 |
| TCP 443 (HTTPS) | アウト | WAN | あり | 送信ポートは、クラウドのデータへのアクセスに使用します。<br></br>送信 Web プロキシは、ユーザーが構成できます。 |
| UDP 53 (DNS) | アウト | WAN | 場合によっては、メモを参照してください。 | このポートは、インターネット ベースの DNS サーバーを使用する場合にのみ必要です。<br></br> **メモ**: ファイル サーバーをデプロイする場合は、ローカル DNS サーバーを使用することをお勧めします。|
| UDP 123 (NTP) | アウト | WAN | 場合によっては、メモを参照してください。 | このポートは、インターネット ベースの NTP サーバーを使用する場合にのみ必要です。<br></br> **メモ**: ファイル サーバーをデプロイする場合は、Active Directory ドメイン コントローラーと時刻を同期することをお勧めします。 |
| TCP 80 (HTTP) | イン | LAN | あり | これは、ローカル管理に使用する StorSimple デバイスのローカル UI の受信ポートです。<br></br>**注**: HTTP 経由でのローカル UI へのアクセスは、自動的に HTTPS にリダイレクトされます。|
| TCP 443 (HTTPS) | イン | LAN | あり | これは、ローカル管理に使用する StorSimple デバイスのローカル UI の受信ポートです。|
| TCP 3260 (iSCSI) | イン | LAN | いいえ | このポートは、iSCSI を介してデータにアクセスするために使用されます。|

<sup>1</sup> 受信ポートがパブリック インターネットで開かれている必要はありません。

### ファイアウォール ルールの URL パターン 

多くの場合、ネットワーク管理者は、受信トラフィックと送信トラフィックをフィルターする URL パターンに基づいて、高度なファイアウォール ルールを構成できます。Virtual Array と StorSimple Manager サービスは、Azure Service Bus、Azure Active Directory Access Control、ストレージ アカウント、Microsoft Update サーバーなどの他の Microsoft アプリケーションに依存しています。その Microsoft アプリケーションと関連付けられた URL パターンを使用してファイアウォール ルールを構成できます。Microsoft アプリケーションに関連付けられた URL パターンは変化する可能性がある点を理解することが重要です。これにより、ネットワーク管理者は必要に応じて StorSimple のファイアウォール ルールを監視し更新する必要があります。

ほとんどの場合、StorSimple 固定 IP アドレスに基づき、送信トラフィックのファイアウォール ルールを設定することが推奨されます。ただし、次の情報を使用して、セキュリティで保護された環境を作成するのにために必要な高度なファイアウォール ルールを設定することもできます。

> [AZURE.NOTE] 
> 
> - デバイスの (送信元) IP は、常にすべてのクラウド対応ネットワーク インターフェイスに合わせて設定します。 
> - 宛先 IP は、[Azure データセンターの IP 範囲](https://www.microsoft.com/ja-JP/download/confirmation.aspx?id=41653)に合わせて設定します。


| URL パターン | コンポーネント/機能 |
|------------------------------------------------------------------|---------------------------------------------------------------|
| `https://*.storsimple.windowsazure.com/*`<br>`https://*.accesscontrol.windows.net/*`<br>`https://*.servicebus.windows.net/*` | StorSimple Manager サービス<br>Access Control Service<br>Azure Service Bus|
|`http://*.backup.windowsazure.com`|デバイス登録|
|`http://crl.microsoft.com/pki/*`<br>`http://www.microsoft.com/pki/*`|証明書の失効 |
| `https://*.core.windows.net/*` | Azure ストレージ アカウントと監視 |
| `http://*.windowsupdate.microsoft.com`<br>`https://*.windowsupdate.microsoft.com`<br>`http://*.update.microsoft.com`<br> `https://*.update.microsoft.com`<br>`http://*.windowsupdate.com`<br>`http://download.microsoft.com`<br>`http://wustat.windows.com`<br>`http://ntservicepack.microsoft.com`| Microsoft Update サーバー<br> |
| `http://*.deploy.akamaitechnologies.com` |Akamai CDN |
| `https://*.partners.extranet.microsoft.com/*` | サポート パッケージ |
| `http://*.data.microsoft.com ` | Windows の Telemetry Service (「[Update for customer experience and diagnostic telemetry (カスタマー エクスペリエンスおよび診断テレメトリの更新プログラム)](https://support.microsoft.com/ja-JP/kb/3068708)」を参照) |

## 次のステップ

-   [StorSimple Virtual Array をデプロイするためにポータルを準備します。](storsimple-ova-deploy1-portal-prep.md)

<!---HONumber=AcomDC_0504_2016-->