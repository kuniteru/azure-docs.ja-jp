Azure は、定期的に更新を行い、仮想マシンのホスト インフラストラクチャの信頼性、パフォーマンス、セキュリティの向上に努めています。 そうした更新は、ホスティング環境のソフトウェア コンポーネント (ホストにデプロイされているオペレーティング システム、ハイパーバイザー、各種エージェント) の修正から、ネットワーク コンポーネントのアップグレード、ハードウェアの使用停止に至るまで、さまざまな範囲に及んでいます。 これらの更新のほとんどは、ホストされている仮想マシンに影響を及ぼすことなく実行されます。 ただし、更新による影響が生じる場合もあります。

- Azure では、再起動を伴わないメンテナンスの場合、インプレース移行を使用して、ホストの更新中に VM を一時的に停止させます。

- 再起動を伴うメンテナンスの場合は、メンテナンスの予定日時が知らされます。 また、その場合、都合の良いときに自分でメンテナンスを開始できる時間枠が与えられます。

このページでは、2 種類のメンテナンスが Microsoft Azure でどのように実行されるかについて説明します。 計画外のイベント (停止) の詳細については、[Windows](../articles/virtual-machines/windows/manage-availability.md) 仮想マシンまたは [Linux](../articles/virtual-machines/linux/manage-availability.md) 仮想マシンの可用性の管理に関するページを参照してください。

仮想マシンで実行されるアプリケーションは、Azure Metadata Service for [Windows](../articles/virtual-machines/windows/instance-metadata-service.md) または Azure Metadata Service for [Linux](../articles/virtual-machines/linux/instance-metadata-service.md) を使って、今後の更新についての情報を収集できます。

## <a name="in-place-vm-migration"></a>インプレース VM 移行

完全な再起動が不要である更新については、インプレースのライブ移行が使用されます。 更新中、仮想マシンは約 30 秒間一時停止状態になり、RAM 内のメモリは保持されます。その間に、ホスティング環境によって必要な更新プログラムとパッチが適用されます。 その後仮想マシンが再開され、仮想マシンの時計が自動的に同期されます。

可用性セットの VM については、更新ドメインが 1 つずつ更新されます。 ある更新ドメイン (UD) 内のすべての VM が一時停止され、更新された後、再開されます。その後、引き続き、次の UD の計画済みメンテナンスが実行されます。

こうしたタイプの更新が一部のアプリケーションに影響を及ぼすことがあります。 リアルタイム イベント処理、メディア ストリーミングまたはコード変換、あるいは高スループットのネットワーク シナリオなどを実行するアプリケーションは、30 秒の一時停止を許容するようには設計されていない可能性があります。 <!-- sooooo, what should they do? --> 


## <a name="maintenance-requiring-a-reboot"></a>再起動を伴うメンテナンス

計画済みメンテナンスで VM の再起動が必要になる場合は、事前に通知が届きます。 計画済みメンテナンスには、"セルフサービス期間" と "予定メンテナンス期間" の 2 つのフェーズがあります。

ご利用の VM に対するメンテナンスは、**セルフサービス期間**に行うことができます。 この期間に、個々の VM にその状態を照会し、自分が最後に行ったメンテナンス要求の結果を確認します。

セルフサービス メンテナンスを開始するときは、既に更新されているノードにご利用の VM を移し、再度電源を投入します。 VM が再起動されるため、一時ディスクは失われ、仮想ネットワーク インターフェイスに関連付けられた動的 IP アドレスは更新されます。

セルフサービス メンテナンスを開始して、そのプロセス中にエラーが発生し、操作が停止された場合、VM は更新されません。また、計画済みメンテナンスのループからも外されます。 後ほど、Microsoft から新しいスケジュールをご連絡いたします。また別途、セルフサービス メンテナンスの機会が設けられます。 

セルフサービス期間が過ぎると、**予定メンテナンス期間**が始まります。 この期間でも、メンテナンス期間について照会することはできますが、この時点でセルフ メンテナンスを開始することはできません。

## <a name="availability-considerations-during-planned-maintenance"></a>計画済みメンテナンス実施時の可用性に関する考慮事項 

計画済みメンテナンス期間まで待つ場合、ご利用の VM の可用性を最大限に保つために考慮すべき事柄がいくつかあります。 

### <a name="paired-regions"></a>ペアになっているリージョン

各 Azure リージョンは、同じ geo 内の別のリージョンと組み合わせて、リージョン ペアにして使用します。 計画済みメンテナンス中は、リージョン ペアの一方のリージョンの VM だけが更新されます。 たとえば、米国中北部の仮想マシンが更新されるとき、同時に米国中南部の仮想マシンが更新されることはありません。 ただし、北ヨーロッパなどのその他のリージョンは、米国東部と同時にメンテナンスされる可能性があります。 各リージョンに対して適切に VM を分散させるためには、リージョン ペアの動作を理解することが大切です。 詳細については、[Azure リージョン ペア](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)に関するページを参照してください。

### <a name="availability-sets-and-scale-sets"></a>可用性セットとスケール セット

Azure VM にワークロードをデプロイするとき、可用性セット内に VM を作成することで、アプリケーションの高可用性を確保することができます。 そうすることで、障害発生時やメンテナンスの間であっても、少なくとも 1 つの仮想マシンを使用できるようになります。

可用性セット内の個々の VM は最大 20 個の更新ドメイン (UD) に分散されます。 計画メンテナンス中は、指定された時間に 1 つの更新ドメインのみ影響を受けます。 更新ドメインへの影響が必ずしも順番に生じるわけではないことに注意してください。 

Virtual Machine Scale Sets は、同一の VM のセットをデプロイして管理するために使用できる Azure コンピューティング リソースです。 スケール セットは、可用性セット内の VM と同じように、複数の更新ドメインに対して分散するように自動的にデプロイされます。 可用性セットと同様、スケール セットの場合も、影響を受ける更新ドメインは一度に 1 つだけです。

高可用性を実現するための仮想マシンの構成について詳しくは、[Windows](../articles/virtual-machines/windows/manage-availability.md) 仮想マシンの可用性管理に関するページまたは [Linux](../articles/virtual-machines/linux/manage-availability.md) 仮想マシンの可用性管理に関するページを参照してください。