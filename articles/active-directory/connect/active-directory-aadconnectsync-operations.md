---
title: "Azure AD Connect 同期: 操作タスクおよび考慮事項 | Microsoft Docs"
description: "このトピックでは、Azure AD Connect Sync の運用タスクと、このコンポーネントを操作するための準備方法について説明します。"
services: active-directory
documentationcenter: 
author: AndKjell
manager: femila
editor: 
ms.assetid: b29c1790-37a3-470f-ab69-3cee824d220d
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/13/2017
ms.author: billmath
ms.translationtype: HT
ms.sourcegitcommit: 349fe8129b0f98b3ed43da5114b9d8882989c3b2
ms.openlocfilehash: b7583a1556bb1113f349a78890768451e39c6878
ms.contentlocale: ja-jp
ms.lasthandoff: 07/26/2017

---
# <a name="azure-ad-connect-sync-operational-tasks-and-consideration"></a>Azure AD Connect Sync: 操作タスクおよび考慮事項
このトピックでは、Azure AD Connect Sync の操作タスクについて説明します。

## <a name="staging-mode"></a>ステージング モード
ステージング モードは、次のシナリオを含むいくつかのシナリオに使用できます。

* 高可用性:
* 新しい構成の変更をテストおよびデプロイする。
* 新しいサーバーを導入し、古いサーバーの使用を中止する。

ステージング モードのサーバーでは、構成を変更した後、そのサーバーをアクティブにする前に変更内容をプレビューできます。 また、フル インポートおよび完全同期を実行して、変更を運用環境に加える前に、すべての変更が予定どおりに加えられていることを確認できます。

インストール時、サーバーを **ステージング モード**に設定することを選択できます。 この操作により、サーバーでインポートと同期が有効になりますが、エクスポートは実行されません。 ステージング モードのサーバーでは、インストール時にパスワード同期やパスワード ライトバックを選択した場合でも、パスワード同期やパスワード ライトバックは実行されません。 ステージング モードを無効にすると、サーバーはエクスポートを開始し、パスワード同期とパスワード ライトバックが有効になります。

同期サービス マネージャーを使用して、引き続き強制的にエクスポートすることもできます。

ステージング モードのサーバーは、Active Directory と Azure AD から変更を受信し続けます。 サーバーには常に最新の変更のコピーが保持され、別のサーバーの役割を迅速に引き継ぐことができます。 プライマリ サーバーの構成を変更する場合、ステージング モードのサーバーに同じ変更を適用するのは管理者の責任です。

従来の同期テクノロジの知識を持つ管理者にとっては、サーバーが独自の SQL Database を持つ点で、ステージング モードは異なるテクノロジに思えることでしょう。 このアーキテクチャにより、ステージング モードのサーバーを別のデータ センターに配置できます。

### <a name="verify-the-configuration-of-a-server"></a>サーバーの構成の確認
この方法を適用するには、次の手順に従います。

1. [準備](#prepare)
2. [構成](#configuration)
3. [インポートおよび同期](#import-and-synchronize)
4. [確認](#verify)
5. [アクティブなサーバーの切り替え](#switch-active-server)

#### <a name="prepare"></a>準備
1. Azure AD Connect をインストールし、**[ステージング モード]** を選択します。インストール ウィザードの最後のページで、**[同期の開始]** を選択解除します。 このモードにより、同期エンジンを手動で実行することができます。
   ![ReadyToConfigure](./media/active-directory-aadconnectsync-operations/readytoconfigure.png)
2. いったんサインオフし、サインインし直してから、[スタート] メニューの **[Synchronization Service (同期サービス)]**を選択します。

#### <a name="configuration"></a>構成
プライマリ サーバーにカスタム変更を行い、構成をステージング サーバーと比較する場合は、[Azure AD Connect 構成データベース構造の解析](https://github.com/Microsoft/AADConnectConfigDocumenter)を使用します。

#### <a name="import-and-synchronize"></a>インポートおよび同期
1. **[コネクタ]** を選択します。種類が "**Active Directory Domain Services**" の 1 つ目のコネクタを選択します。 **[Run (実行)]**、**[Full import (フル インポート)]**、**[OK]** の順にクリックします。 この種類のすべてのコネクタに対して、これらの手順を繰り返します。
2. 種類が " **Azure Active Directory (Microsoft)**" のコネクタを選択します。 **[Run (実行)]**、**[Full import (フル インポート)]**、**[OK]** の順にクリックします。
3. [Connectors (コネクタ)] タブが選択されたままであることを確認します。 種類が "**Active Directory Domain Services**" の各コネクタに対し、**[Run (実行)]**、**[Delta Synchronization (差分同期)]**、**[OK]** の順にクリックします。
4. 種類が " **Azure Active Directory (Microsoft)**" のコネクタを選択します。 **[Run (実行)]**、**[Delta Synchronization (差分同期)]**、**[OK]** の順にクリックします。

これで、Azure AD とオンプレミスの AD (Exchange ハイブリッド デプロイを使用している) へのエクスポートの変更がステージングされました。 次の手順では、実際にディレクトリへのエクスポートを開始する前に、変更される内容を確認できます。

#### <a name="verify"></a>確認
1. コマンド プロンプトを起動し、`%ProgramFiles%\Microsoft Azure AD Sync\bin` に移動します。
2. `csexport "Name of Connector" %temp%\export.xml /f:x` を実行します。同期サービスにコネクタの名前があることを確認できます。 Azure AD の場合は、"contoso.com - AAD" のような名前が表示されます。
3. セクション [CSAnalyzer](#appendix-csanalyzer) から `csanalyzer.ps1` という名前のファイルに PowerShell スクリプトをコピーします。
4. PowerShell ウィンドウを開き、PowerShell スクリプトを作成したフォルダーを参照します。
5. `.\csanalyzer.ps1 -xmltoimport %temp%\export.xml` を実行します。
6. **processedusers1.csv** という名前のファイルが生成されます。このファイルは、Microsoft Excel で開くことができます。 Azure AD にエクスポートするためにステージングされたすべての変更は、このファイル内にあります。
7. データまたは構成に必要な変更を加え、エクスポートの対象となる変更が希望どおりになるまで、(「インポートおよび同期」と「確認」の) 手順を実行します。

#### <a name="switch-active-server"></a>アクティブなサーバーの切り替え
1. 現在アクティブなサーバーで、サーバー (DirSync、FIM、または Azure AD Sync) をオフにして Azure AD にエクスポートしないように設定するか、ステージング モード (Azure AD Connect) に設定します。
2. **ステージング モード**のサーバーでインストール ウィザードを実行して、**ステージング モード**を無効にします。
   ![ReadyToConfigure](./media/active-directory-aadconnectsync-operations/additionaltasks.png)

## <a name="disaster-recovery"></a>障害復旧
実装の設計には、同期サーバーを喪失する障害の発生時の対処方法を計画することが含まれます。 モデルにはさまざまなものがあり、どのモデルを使用するかは、次の要素を含むいくつかの要素に依存します。

* ダウンタイム中に Azure AD のオブジェクトを変更できないことに関してどれだけ許容できますか?
* パスワード同期を使用する場合、オンプレミスでパスワードを変更する場合に備えて Azure AD で古いパスワードを使用することが求められることについてユーザーの同意が得られますか?
* パスワード ライトバックなどのリアルタイムの操作に依存していますか?

これらの質問の回答と組織のポリシーに応じて、次の戦略のいずれかを実装することができます。

* 必要に応じて再構築する。
* 予備のスタンバイ サーバーを用意する (" **ステージング モード**" と呼ばれます)。
* 仮想マシンを使用する。

組み込みの SQL Express データベースを使用しない場合は、「 [SQL 高可用性](#sql-high-availability) 」セクションも確認してください。

### <a name="rebuild-when-needed"></a>必要に応じて再構築する
実行可能な戦略は、必要に応じてサーバーの再構築を計画することです。 通常、同期エンジンのインストールと最初のインポートおよび同期操作は、数時間以内に完了します。 予備のサーバーが利用できない場合は、一時的にドメイン コントローラーを使用して同期エンジンをホストすることができます。

オブジェクトに関する状態は同期エンジン サーバーには保存されないため、Active Directory と Azure AD 内のデータからデータベースを再構築することができます。 **sourceAnchor** 属性は、オンプレミスとクラウドからのオブジェクトを結合するために使用されます。 オンプレミスとクラウドの既存のオブジェクトを使ってサーバーを再構築する場合、同期エンジンは、再インストール時にこれらのオブジェクトをもう一度まとめて適合させます。 ドキュメント化して保存する必要があることは、フィルター規則、同期規則など、サーバーに行った構成の変更です。 同期を開始する前に、これらのカスタム構成を再適用する必要があります。

### <a name="have-a-spare-standby-server---staging-mode"></a>予備のスタンバイ サーバーを用意する - ステージング モード
環境がより複雑な場合は、1 つまたは複数のスタンバイ サーバーを持つことをお勧めします。 インストール時、サーバーを **ステージング モード**に設定できます。

詳しくは、「[ステージング モード](#staging-mode)」をご覧ください。

### <a name="use-virtual-machines"></a>仮想マシンを使用する
一般的なサポートされている方法は、仮想マシンで同期エンジンを実行する方法です。 ホストに問題が発生した場合、同期エンジン サーバーを含むイメージを別のサーバーに移行できます。

### <a name="sql-high-availability"></a>SQL 高可用性
Azure AD Connect に付属している SQL Server Express を使用しない場合は、SQL Server の高可用性も考慮する必要があります。 サポートされている高可用性ソリューションには、SQL クラスタリングおよび AOA (Always On 可用性グループ) が含まれます。 サポートされていないソリューションには、ミラーリングがあります。

SQL AOA のサポートが、Azure AD Connect のバージョン 1.1.524.0 に追加されました。 Azure AD Connect をインストールする前に SQL AOA を有効にする必要があります。 インストール中、指定された SQL インスタンスで SQL AOA が有効であるかどうかが Azure AD Connect によって検出されます。 SQL AOA が有効である場合、Azure AD Connect はさらに、SQL AOA が、同期レプリケーションまたは非同期レプリケーションを使用するように構成されているかどうかを調べます。 可用性グループ リスナーを設定するときは、RegisterAllProvidersIP プロパティを 0 に設定することをお勧めします。 Azure AD Connect は現在、SQL Native Client を使用して SQL に接続していますが、SQL Native Client は、MultiSubNetFailover プロパティの使用をサポートしていないためです。

## <a name="appendix-csanalyzer"></a>付録 CSAnalyzer
このスクリプトの使い方については、「[確認](#verify)」をご覧ください。

```
Param(
    [Parameter(Mandatory=$true, HelpMessage="Must be a file generated using csexport 'Name of Connector' export.xml /f:x)")]
    [string]$xmltoimport="%temp%\exportedStage1a.xml",
    [Parameter(Mandatory=$false, HelpMessage="Maximum number of users per output file")][int]$batchsize=1000,
    [Parameter(Mandatory=$false, HelpMessage="Show console output")][bool]$showOutput=$false
)

#LINQ isn't loaded automatically, so force it
[Reflection.Assembly]::Load("System.Xml.Linq, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089") | Out-Null

[int]$count=1
[int]$outputfilecount=1
[array]$objOutputUsers=@()

#XML must be generated using "csexport "Name of Connector" export.xml /f:x"
write-host "Importing XML" -ForegroundColor Yellow

#XmlReader.Create won't properly resolve the file location,
#so expand and then resolve it
$resolvedXMLtoimport=Resolve-Path -Path ([Environment]::ExpandEnvironmentVariables($xmltoimport))

#use an XmlReader to deal with even large files
$result=$reader = [System.Xml.XmlReader]::Create($resolvedXMLtoimport) 
$result=$reader.ReadToDescendant('cs-object')
do 
{
    #create the object placeholder
    #adding them up here means we can enforce consistency
    $objOutputUser=New-Object psobject
    Add-Member -InputObject $objOutputUser -MemberType NoteProperty -Name ID -Value ""
    Add-Member -InputObject $objOutputUser -MemberType NoteProperty -Name Type -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name DN -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name operation -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name UPN -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name displayName -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name sourceAnchor -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name alias -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name primarySMTP -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name onPremisesSamAccountName -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name mail -Value ""

    $user = [System.Xml.Linq.XElement]::ReadFrom($reader)
    if ($showOutput) {Write-Host Found an exported object... -ForegroundColor Green}

    #object id
    $outID=$user.Attribute('id').Value
    if ($showOutput) {Write-Host ID: $outID}
    $objOutputUser.ID=$outID

    #object type
    $outType=$user.Attribute('object-type').Value
    if ($showOutput) {Write-Host Type: $outType}
    $objOutputUser.Type=$outType

    #dn
    $outDN= $user.Element('unapplied-export').Element('delta').Attribute('dn').Value
    if ($showOutput) {Write-Host DN: $outDN}
    $objOutputUser.DN=$outDN

    #operation
    $outOperation= $user.Element('unapplied-export').Element('delta').Attribute('operation').Value
    if ($showOutput) {Write-Host Operation: $outOperation}
    $objOutputUser.operation=$outOperation

    #now that we have the basics, go get the details

    foreach ($attr in $user.Element('unapplied-export-hologram').Element('entry').Elements("attr"))
    {
        $attrvalue=$attr.Attribute('name').Value
        $internalvalue= $attr.Element('value').Value

        switch ($attrvalue)
        {
            "userPrincipalName"
            {
                if ($showOutput) {Write-Host UPN: $internalvalue}
                $objOutputUser.UPN=$internalvalue
            }
            "displayName"
            {
                if ($showOutput) {Write-Host displayName: $internalvalue}
                $objOutputUser.displayName=$internalvalue
            }
            "sourceAnchor"
            {
                if ($showOutput) {Write-Host sourceAnchor: $internalvalue}
                $objOutputUser.sourceAnchor=$internalvalue
            }
            "alias"
            {
                if ($showOutput) {Write-Host alias: $internalvalue}
                $objOutputUser.alias=$internalvalue
            }
            "proxyAddresses"
            {
                if ($showOutput) {Write-Host primarySMTP: ($internalvalue -replace "SMTP:","")}
                $objOutputUser.primarySMTP=$internalvalue -replace "SMTP:",""
            }
        }
    }

    $objOutputUsers += $objOutputUser

    Write-Progress -activity "Processing ${xmltoimport} in batches of ${batchsize}" -status "Batch ${outputfilecount}: " -percentComplete (($objOutputUsers.Count / $batchsize) * 100)

    #every so often, dump the processed users in case we blow up somewhere
    if ($count % $batchsize -eq 0)
    {
        Write-Host Hit the maximum users processed without completion... -ForegroundColor Yellow

        #export the collection of users as as CSV
        Write-Host Writing processedusers${outputfilecount}.csv -ForegroundColor Yellow
        $objOutputUsers | Export-Csv -path processedusers${outputfilecount}.csv -NoTypeInformation

        #increment the output file counter
        $outputfilecount+=1

        #reset the collection and the user counter
        $objOutputUsers = $null
        $count=0
    }

    $count+=1

    #need to bail out of the loop if no more users to process
    if ($reader.NodeType -eq [System.Xml.XmlNodeType]::EndElement)
    {
        break
    }

} while ($reader.Read)

#need to write out any users that didn't get picked up in a batch of 1000
#export the collection of users as as CSV
Write-Host Writing processedusers${outputfilecount}.csv -ForegroundColor Yellow
$objOutputUsers | Export-Csv -path processedusers${outputfilecount}.csv -NoTypeInformation
```

## <a name="next-steps"></a>次のステップ
**概要トピック**  

* [Azure AD Connect sync: 同期を理解してカスタマイズする](active-directory-aadconnectsync-whatis.md)  
* [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)  

