1. ポータル ページの左側の **+** をクリックし、検索ボックスに「Virtual Network ゲートウェイ」と入力します。 **[結果]** から **[Virtual Network ゲートウェイ]** を探してクリックします。
2. [仮想ネットワーク ゲートウェイ] ブレードの下部にある **[作成]** をクリックします。 **[仮想ネットワーク ゲートウェイの作成]** ブレードが開きます。

    ![[仮想ネットワーク ゲートウェイの作成] ブレードのフィールド](./media/vpn-gateway-add-gw-s2s-rm-portal-include/vnet_gw.png "新しいゲートウェイ")

3. **[仮想ネットワーク ゲートウェイの作成]** ブレードで、仮想ネットワーク ゲートウェイの値を指定します。

  - **[名前]**: ゲートウェイに名前を付けます。 これは、ゲートウェイ サブネットの名前付けと同じではありません。 作成するゲートウェイ オブジェクトの名前です。
  - **[ゲートウェイの種類]**: **[VPN]** を選択します。 VPN Gateway では、仮想ネットワーク ゲートウェイの種類として **VPN** を使用します。 
  - **[VPN の種類]**: 構成に指定されている VPN の種類を選択します。 ほとんどの構成で [VPN の種類] は [ルート ベース] にする必要があります。
  - **[SKU]**: ゲートウェイの SKU をドロップダウン リストから選択します。 ドロップダウン リストに表示される SKU は、選択した VPN の種類によって異なります。 ゲートウェイの SKU の詳細については、「[ゲートウェイの SKU](../articles/vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#gwsku)」を参照してください。
  - **[場所]**: [場所] が見えない場合はスクロールしてください。 仮想ネットワークの場所を指すように、 **[場所]** フィールドを調整します。 対象の仮想ネットワークが存在するリージョンをこの場所が指していない場合、次の手順で [仮想ネットワークの選択] ボックスの一覧に仮想ネットワークが表示されません。
  - **[仮想ネットワーク]**: このゲートウェイの追加先の仮想ネットワークを選択します。 **[仮想ネットワーク]** をクリックして [仮想ネットワークの選択] ブレードを開きます。 VNet を選択します。 VNet が表示されない場合は、実際の仮想ネットワークがあるリージョンが [場所] フィールドに指定されていることを確認してください。
  - **[パブリック IP アドレス]**: [パブリック IP アドレスの作成] ブレードで、パブリック IP アドレス オブジェクトを作成します。 パブリック IP アドレスは、VPN ゲートウェイの作成時に動的に割り当てられます。 VPN Gateway では現在、パブリック IP アドレスの "*動的*" 割り当てのみサポートしています。 もっとも、VPN ゲートウェイに割り当てられた IP アドレスが後から変わることは基本的にありません。 パブリック IP アドレスが変わるのは、ゲートウェイが削除され、再度作成されたときのみです。 VPN ゲートウェイのサイズ変更、リセット、その他の内部メンテナンス/アップグレードでは、IP アドレスは変わりません。

    - 最初に、**[パブリック IP アドレス]** をクリックして [パブリック IP アドレスの選択] ブレードを開き、次に、**[+新規作成]** をクリックして [パブリック IP アドレスの作成] ブレードを開きます。
    - 次に、パブリック IP アドレスの **[名前]** を入力し、ブレードの下部にある **[OK]** をクリックして変更を保存します。

      ![パブリック IP の作成](./media/vpn-gateway-add-gw-s2s-rm-portal-include/pip.png "PIP の作成")

4. 設定を確認します。 ゲートウェイをダッシュボードに表示する場合は、ブレードの下部にある **[ダッシュボードにピン留めする]** を選択します。 
5. **[作成]** をクリックして、VPN ゲートウェイの作成を開始します。 設定が検証され、ダッシュボードに [Deploying Virtual network gateway (仮想ネットワーク ゲートウェイのデプロイ)] タイルが表示されます。 ゲートウェイの作成には、最大で 45 分かかる場合があります。 完了状態を確認するために、ポータル ページの更新が必要な場合があります。

ゲートウェイの作成後、ポータルの仮想ネットワークを調べることで、ゲートウェイに割り当てられている IP アドレスを確認します。 ゲートウェイは、接続されたデバイスとして表示されます。 詳細を表示するには、接続したデバイス (仮想ネットワーク ゲートウェイ) をクリックします。