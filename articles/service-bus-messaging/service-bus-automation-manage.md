---
title: "Azure Automation を使用した Azure Service Bus の管理 | Microsoft Docs"
description: "Azure Automation サービスを使用して Azure Service Bus を管理する方法について説明します。"
services: service-bus-messaging, automation
documentationcenter: 
author: mgoedtel
manager: jwhit
editor: 
ms.assetid: 2a06e94b-92ef-4b5d-a2b7-fe827063df82
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/10/2017
ms.author: magoedte;csand
translationtype: Human Translation
ms.sourcegitcommit: 994a379129bffd7457912bc349f240a970aed253
ms.openlocfilehash: 4817cda757b9f85f6225237f79293860c990ca88


---
# <a name="managing-azure-service-bus-using-azure-automation"></a>Azure Automation を使用した Azure Service Bus の管理
このガイドでは、Azure Automation サービスと、このサービスを使用して Azure Service Bus の管理を簡略化する方法について紹介します。

## <a name="what-is-azure-automation"></a>Azure Automation とは
[Azure Automation](../automation/automation-intro.md) は、プロセスの自動化と必要な状態の構成によってクラウド管理を簡略化するための Azure サービスです。 Azure Automation を使用して手動タスク、繰り返しタスク、実行時間の長いタスク、エラーが発生しやすいタスクを自動化し、信頼性と効率性を向上して組織のゴール達成までの時間を短縮できます。

Azure Automation は、ニーズに対応可能な信頼性と可用性の高いワークフロー実行エンジンを提供します。 Azure Automation では、サード パーティ製のシステムによって手動でプロセスを開始したり、必要なときに正確にタスクが起動されるようにスケジュールされた間隔でプロセスを開始できます。

Azure Automation によってクラウド管理タスクを自動的に実行することにより、運用上のオーバーヘッドが削減され、ビジネス価値の向上に重点的に取り組む IT と DevOps スタッフの負担が軽くなります。

## <a name="how-can-azure-automation-help-manage-azure-service-bus"></a>Azure Automation を Azure Service Bus の管理に役立てる方法
[Service Bus REST API](https://docs.microsoft.com/rest/api/servicebus/)を使用することによって、Azure Automation で Service Bus を管理できます。 Azure Automation 内では、REST API を使用して Service Bus タスクの多くを処理する PowerShell スクリプトを実行できます。 このような Azure Automation 内の REST API 呼び出しと別の Azure サービスのコマンドレットを組み合わせて、Azure サービスとサード パーティ システム全体に及ぶ複雑なタスクを自動化することもできます。

PowerShell を使用して Azure Service Bus を管理するいくつかの例を次に示します。

* [Azure Service Bus キューを管理するカスタム PowerShell コマンドレット](https://blogs.technet.microsoft.com/uktechnet/2014/12/04/sample-of-custom-powershell-cmdlets-to-manage-azure-servicebus-queues)
* [PowerShell スクリプトを使用してService Bus キュー、トピック、サブスクリプションを作成する方法 (ブログの投稿)](http://blogs.msdn.com/b/paolos/archive/2014/12/02/how-to-create-a-service-bus-queues-topics-and-subscriptions-using-a-powershell-script.aspx)
* [PowerShell を使用して Azure Service Bus 名前空間を作成する](http://buildazure.com/2015/09/24/create-azure-service-bus-namespaces-using-powershell-and-x-plat-cli/)

Automation Runbook で Azure Service Bus を使用するための PowerShell モジュールは、 [PowerShell ギャラリー](https://www.powershellgallery.com/packages/AzureServiceBusCreation/1.0)からダウンロードできます。

## <a name="next-steps"></a>次のステップ
ここまで、Azure Automation の基本と Azure Automation を使用して Azure Service Bus を管理する方法について説明しました。Azure Automation の詳細については、以下のリンクを参照してください。

* [Azure Automation の使用](../automation/automation-first-runbook-graphical.md)
* [PowerShell での Service Bus の管理方法](service-bus-powershell-how-to-provision.md)




<!--HONumber=Jan17_HO2-->


