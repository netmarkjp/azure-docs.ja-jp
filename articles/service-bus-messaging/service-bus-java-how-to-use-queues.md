---
title: "Java での Service Bus キューの使用方法 | Microsoft Docs"
description: "Azure での Service Bus キューの使用方法を学習します。 コード サンプルは Java で記述されています。"
services: service-bus-messaging
documentationcenter: java
author: sethmanheim
manager: timlt
ms.assetid: f701439c-553e-402c-94a7-64400f997d59
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 10/04/2016
ms.author: sethm
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 29cab1dff7ffc0f42ee8c605e3817b855967eb53


---
# <a name="how-to-use-service-bus-queues"></a>Service Bus キューの使用方法
[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

この記事では、Service Bus キューの使用方法について説明します。 サンプルは Java で記述され、[Azure SDK for Java][Azure SDK for Java] を利用しています。 紹介するシナリオは、**キューの作成**、**メッセージの送受信**、**キューの削除**です。

## <a name="what-are-service-bus-queues"></a>Service Bus キューとは
Service Bus キューは、**ブローカー メッセージング通信**モデルをサポートしています。 キューを使用すると、分散アプリケーションのコンポーネントが互いに直接通信することがなくなり、仲介者 (ブローカー) の役割を果たすキューを介してメッセージをやり取りすることになります。 メッセージ プロデューサー (送信者) はキューにメッセージを送信した後で、それまでの処理を引き続き実行します。
メッセージ コンシューマー (受信者) は、キューからメッセージを非同期に受信して処理します。 メッセージ プロデューサーは、それ以降のメッセージの処理と送信を続ける場合、メッセージ コンシューマーからの応答を待つ必要がありません。 キューでは、コンシューマーが競合している場合のメッセージ配信に**先入れ先出し法 (FIFO)** を使用します。 つまり、通常はキューに追加された順番にメッセージが受信され、処理されます。このとき、メッセージを受信して処理できるメッセージ コンシューマーは、メッセージ 1 件につき 1 つだけです。

![QueueConcepts](./media/service-bus-java-how-to-use-queues/sb-queues-08.png)

Service Bus キューは汎用テクノロジであり、幅広いシナリオで使用できます。

* 多層 Azure アプリケーションでの Web ロールと Worker ロールとの間の通信。
* ハイブリッド ソリューションでのオンプレミスのアプリと Azure によってホストされるアプリケーションとの間の通信。
* 複数の組織で実行される分散アプリケーションまたは 1 つの組織内の異なる部署でオンプレミスで実行される分散アプリケーションのコンポーネント間の通信。

キューを使用すると、アプリケーションのスケールアウトがより簡単になり、アーキテクチャの復元性が得られます。

## <a name="create-a-service-namespace"></a>サービス名前空間の作成
Azure の Service Bus キューを使用するには、最初に名前空間を作成する必要があります。 名前空間は、アプリケーション内で Service Bus リソースをアドレス指定するためのスコープ コンテナーを提供します。

名前空間を作成するには:

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

## <a name="configure-your-application-to-use-service-bus"></a>Service Bus を使用するようにアプリケーションを構成する
このサンプルを作成する前に [Azure SDK for Java][Azure SDK for Java] がインストールされていることを確認してください。 Eclipse を使用している場合は、Azure SDK for Java が含まれている [Azure Toolkit for Eclipse][Azure Toolkit for Eclipse] をインストールできます。 これで **Microsoft Azure Libraries for Java** をプロジェクトに追加できます。

![](./media/service-bus-java-how-to-use-queues/eclipselibs.png)

次の `import` ステートメントを Java ファイルの先頭に追加します。

```
// Include the following imports to use Service Bus APIs
import com.microsoft.windowsazure.services.servicebus.*;
import com.microsoft.windowsazure.services.servicebus.models.*;
import com.microsoft.windowsazure.core.*;
import javax.xml.datatype.*;
```

## <a name="create-a-queue"></a>キューを作成する
Service Bus キューの管理処理は **ServiceBusContract** クラスを使用して実行できます。 **ServiceBusContract** オブジェクトは SAS をカプセル化する構成とそれを管理するアクセス許可で構築されます。**ServiceBusContract** クラスでのみ Azure Service Bus トピックとのやり取りが可能です。

**ServiceBusService** クラスには、キューの作成、列挙、削除のためのメソッドが用意されています。 次の例では、**ServiceBusService** オブジェクトを使用して、"HowToSample" 名前空間の "TestQueue" という名前のキューを作成する方法を示しています。

```
Configuration config =
    ServiceBusConfiguration.configureWithSASAuthentication(
            "HowToSample",
            "RootManageSharedAccessKey",
            "SAS_key_value",
            ".servicebus.windows.net"
            );

ServiceBusContract service = ServiceBusService.create(config);
QueueInfo queueInfo = new QueueInfo("TestQueue");
try
{
    CreateQueueResult result = service.createQueue(queueInfo);
}
catch (ServiceException e)
{
    System.out.print("ServiceException encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
```

**QueueInfo** には、キューのプロパティを調整できるメソッドが用意されています (たとえば、キューに送信されるメッセージに対して既定の有効期間 (TTL) 値が適用されるように設定できます)。 次の例では、名前が `TestQueue`、最大サイズが 5 GB であるキューを作成する方法を示しています。

````
long maxSizeInMegabytes = 5120;
QueueInfo queueInfo = new QueueInfo("TestQueue");
queueInfo.setMaxSizeInMegabytes(maxSizeInMegabytes);
CreateQueueResult result = service.createQueue(queueInfo);
````

**ServiceBusContract** オブジェクトの **listQueues** メソッドを使用すると、指定した名前のキューがサービス名前空間に既に存在するかどうかを確認できます。

## <a name="send-messages-to-a-queue"></a>メッセージをキューに送信する
メッセージを Service Bus キューに送信するには、アプリケーションで **ServiceBusContract** オブジェクトを取得します。 次のコードでは、上のコードで `HowToSample` 名前空間内で作成した `TestQueue` キューにメッセージを送信する方法を示しています。

```
try
{
    BrokeredMessage message = new BrokeredMessage("MyMessage");
    service.sendQueueMessage("TestQueue", message);
}
catch (ServiceException e)
{
    System.out.print("ServiceException encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
```

Service Bus キューに対して送受信されたメッセージは、[BrokeredMessage][BrokeredMessage] クラスのインスタンスになります。 [BrokeredMessage][BrokeredMessage] オブジェクトには、([Label](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx)、[TimeToLive](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx) などの) 一連の標準的なプロパティ、アプリケーションに特有のカスタム プロパティの保持に使用するディクショナリ、任意のアプリケーション データがあります。 アプリケーションでは、[BrokeredMessage][BrokeredMessage] のコンストラクターにシリアル化可能なオブジェクトを渡すことによってメッセージの本文を設定できます。その後で、適切なシリアライザーを使用してオブジェクトをシリアル化します。 または、**java.IO.InputStream** オブジェクトを提供することもできます。

以下の例では、上のコード スニペットで取得した `TestQueue` **MessageSender** にテスト メッセージを 5 件送信する方法を示しています。

```
for (int i=0; i<5; i++)
{
     // Create message, passing a string message for the body.
     BrokeredMessage message = new BrokeredMessage("Test message " + i);
     // Set an additional app-specific property.
     message.setProperty("MyProperty", i);
     // Send message to the queue
     service.sendQueueMessage("TestQueue", message);
}
```

Service Bus キューでサポートされているメッセージの最大サイズは、[Standard レベル](service-bus-premium-messaging.md)では 256 KB、[Premium レベル](service-bus-premium-messaging.md)では 1 MB です。 標準とカスタムのアプリケーション プロパティが含まれるヘッダーの最大サイズは 64 KB です。 キューで保持されるメッセージ数には上限がありませんが、キュー 1 つあたりが保持できるメッセージの合計サイズには上限があります。 このキュー サイズは作成時に定義され、上限は 5 GB です。

## <a name="receive-messages-from-a-queue"></a>キューからメッセージを受信する
キューからメッセージを受信する主な方法は、**ServiceBusContract** オブジェクトを使用することです。 メッセージは、**ReceiveAndDelete** と **PeekLock** の 2 つの異なるモードで受信できます。

**ReceiveAndDelete** モードを使用する場合、受信が 1 回ずつの動作になります。つまり、Service Bus はキュー内のメッセージに対する読み取り要求を受け取ると、メッセージを読み取り中としてマークし、アプリケーションに返します。 **ReceiveAndDelete** モード (既定) は最もシンプルなモデルであり、障害発生時にアプリケーション側でメッセージを処理しないことを許容できるシナリオに最適です。 このことを理解するために、コンシューマーが受信要求を発行した後で、メッセージを処理する前にクラッシュしたというシナリオを考えてみましょう。
Service Bus はメッセージを読み取り済みとしてマークするため、アプリケーションが再起動してメッセージの読み取りを再開すると、クラッシュ前に読み取られていたメッセージは見落とされることになります。

**PeekLock** モードでは、メッセージの受信処理が 2 段階の動作になり、メッセージが失われることが許容できないアプリケーションに対応することができます。 Service Bus は要求を受け取ると、次に読み取られるメッセージを検索して、他のコンシューマーが受信できないようロックしてから、アプリケーションにメッセージを返します。 アプリケーションがメッセージの処理を終えた後 (または後で処理するために確実に保存した後)、受信したメッセージに対して **Delete** を呼び出して受信処理の第 2 段階を完了します。 Service Bus が **Delete** の呼び出しを確認すると、メッセージが読み取り中としてマークされ、キューから削除されます。

次の例では、(既定モードではなく) **PeekLock** モードを使用したメッセージの受信および処理の方法を示しています。 次の例では、無限ループを使用して、"TestQueue" にメッセージが到着するごとに処理しています。

```
try
{
    ReceiveMessageOptions opts = ReceiveMessageOptions.DEFAULT;
    opts.setReceiveMode(ReceiveMode.PEEK_LOCK);

    while(true)  {
         ReceiveQueueMessageResult resultQM =
                 service.receiveQueueMessage("TestQueue", opts);
        BrokeredMessage message = resultQM.getValue();
        if (message != null && message.getMessageId() != null)
        {
            System.out.println("MessageID: " + message.getMessageId());
            // Display the queue message.
            System.out.print("From queue: ");
            byte[] b = new byte[200];
            String s = null;
            int numRead = message.getBody().read(b);
            while (-1 != numRead)
            {
                s = new String(b);
                s = s.trim();
                System.out.print(s);
                numRead = message.getBody().read(b);
            }
            System.out.println();
            System.out.println("Custom Property: " +
                message.getProperty("MyProperty"));
            // Remove message from queue.
            System.out.println("Deleting this message.");
            //service.deleteMessage(message);
        }  
        else  
        {
            System.out.println("Finishing up - no more messages.");
            break;
            // Added to handle no more messages.
            // Could instead wait for more messages to be added.
        }
    }
}
catch (ServiceException e) {
    System.out.print("ServiceException encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
catch (Exception e) {
    System.out.print("Generic exception encountered: ");
    System.out.println(e.getMessage());
    System.exit(-1);
}
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>アプリケーションのクラッシュと読み取り不能のメッセージを処理する方法
Service Bus には、アプリケーションにエラーが発生した場合や、メッセージの処理に問題がある場合に復旧を支援する機能が備わっています。 受信側のアプリケーションが何らかの理由によってメッセージを処理できない場合には、受信したメッセージについて (**deleteMessage** メソッドの代わりに) **unlockMessage** メソッドを呼び出すことができます。 このメソッドが呼び出されると、Service Bus によってキュー内のメッセージのロックが解除され、メッセージが再度受信できる状態に変わります。メッセージを受信するアプリケーションは、以前と同じものでも、別のものでもかまいません。

キュー内でロックされているメッセージにはタイムアウトも設定されています。アプリケーションがクラッシュした場合など、ロックがタイムアウトになる前にアプリケーションがメッセージの処理に失敗した場合には、Service Bus によりメッセージのロックが自動的に解除され、再度受信できる状態に変わります。

メッセージが処理された後、**deleteMessage** 要求が発行される前にアプリケーションがクラッシュした場合は、アプリケーションが再起動する際にメッセージが再配信されます。 一般的に、この動作は **1 回以上の処理** と呼ばれます。つまり、すべてのメッセージが 1 回以上処理されますが、特定の状況では、同じメッセージが再配信される可能性があります。 重複処理が許されないシナリオの場合、重複メッセージの配信を扱うロジックをアプリケーションに追加する必要があります。 通常、この問題はメッセージの **getMessageId** メソッドを使用して対処します。このプロパティは、配信が試行された後も同じ値を保持します。

## <a name="next-steps"></a>次のステップ
これで、Service Bus キューの基本を学習できました。詳細については、[「キュー、トピック、サブスクリプション」][キュー、トピック、サブスクリプション]を参照してください。

詳細については、 [Java デベロッパー センター](/develop/java/)を参照してください。

[Azure SDK for Java]: http://azure.microsoft.com/develop/java/
[Azure Toolkit for Eclipse]: https://msdn.microsoft.com/library/azure/hh694271.aspx
[キュー、トピック、サブスクリプション]: service-bus-queues-topics-subscriptions.md
[BrokeredMessage]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx




<!--HONumber=Nov16_HO3-->


