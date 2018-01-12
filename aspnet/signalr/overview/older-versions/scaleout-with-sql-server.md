---
uid: signalr/overview/older-versions/scaleout-with-sql-server
title: SignalR mit horizontaler Skalierung mit SQLServer (SignalR 1.x) | Microsoft Docs
author: MikeWasson
description: 
ms.author: aspnetcontent
manager: wpickett
ms.date: 05/01/2013
ms.topic: article
ms.assetid: 1dca7967-8296-444a-9533-837eb284e78c
ms.technology: dotnet-signalr
ms.prod: .net-framework
msc.legacyurl: /signalr/overview/older-versions/scaleout-with-sql-server
msc.type: authoredcontent
ms.openlocfilehash: 7a589f5c6a5196444c3c616b39f501f3a7512471
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/10/2017
---
<a name="signalr-scaleout-with-sql-server-signalr-1x"></a><span data-ttu-id="9cdf6-102">SignalR mit horizontaler Skalierung mit SQLServer (SignalR 1.x)</span><span class="sxs-lookup"><span data-stu-id="9cdf6-102">SignalR Scaleout with SQL Server (SignalR 1.x)</span></span>
====================
<span data-ttu-id="9cdf6-103">durch [Mike Wasson](https://github.com/MikeWasson), [Patrick Fletcher](https://github.com/pfletcher)</span><span class="sxs-lookup"><span data-stu-id="9cdf6-103">by [Mike Wasson](https://github.com/MikeWasson), [Patrick Fletcher](https://github.com/pfletcher)</span></span>

<span data-ttu-id="9cdf6-104">In diesem Lernprogramm verwenden Sie SQL Server zum Verteilen von Nachrichten auf einer SignalR-Anwendung, die in zwei separaten IIS-Instanzen bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-104">In this tutorial, you will use SQL Server to distribute messages across a SignalR application that is deployed in two separate IIS instances.</span></span> <span data-ttu-id="9cdf6-105">Sie können auch in diesem Lernprogramm ausführen, auf einen einzelnen Testcomputer, aber um vollständige Transparenzeffekt zu erzielen, müssen Sie die SignalR-Anwendung auf zwei oder mehr Servern bereitzustellen.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-105">You can also run this tutorial on a single test machine, but to get the full effect, you need to deploy the SignalR application to two or more servers.</span></span> <span data-ttu-id="9cdf6-106">Sie müssen SQL Server auch auf einem Server oder auf einem separaten dedizierten Server installieren.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-106">You must also install SQL Server on one of the servers, or on a separate dedicated server.</span></span> <span data-ttu-id="9cdf6-107">Eine andere Möglichkeit ist das Lernprogramm mithilfe von virtuellen Computern in Azure ausgeführt.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-107">Another option is to run the tutorial using VMs on Azure.</span></span>

![](scaleout-with-sql-server/_static/image1.png)

## <a name="prerequisites"></a><span data-ttu-id="9cdf6-108">Erforderliche Komponenten</span><span class="sxs-lookup"><span data-stu-id="9cdf6-108">Prerequisites</span></span>

<span data-ttu-id="9cdf6-109">Microsoft SQLServer 2005 oder höher.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-109">Microsoft SQL Server 2005 or later.</span></span> <span data-ttu-id="9cdf6-110">Der Rückwand unterstützt Desktop- und Server-Editionen von SQL Server.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-110">The backplane supports both desktop and server editions of SQL Server.</span></span> <span data-ttu-id="9cdf6-111">Er unterstützt keine SQL Server Compact Edition oder Azure SQL-Datenbank.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-111">It does not support SQL Server Compact Edition or Azure SQL Database.</span></span> <span data-ttu-id="9cdf6-112">(Wenn die Anwendung in Azure gehostet wird, empfiehlt es sich die Service Bus-Rückwandplatine stattdessen.)</span><span class="sxs-lookup"><span data-stu-id="9cdf6-112">(If your application is hosted on Azure, consider the Service Bus backplane instead.)</span></span>

## <a name="overview"></a><span data-ttu-id="9cdf6-113">Übersicht</span><span class="sxs-lookup"><span data-stu-id="9cdf6-113">Overview</span></span>

<span data-ttu-id="9cdf6-114">Bevor wir auf die ausführliches Lernprogramm erhalten, ist hier ein schnellen Überblick darüber, was Sie tun können.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-114">Before we get to the detailed tutorial, here is a quick overview of what you will do.</span></span>

1. <span data-ttu-id="9cdf6-115">Erstellen Sie eine neue leere Datenbank ein.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-115">Create a new empty database.</span></span> <span data-ttu-id="9cdf6-116">Der Rückwand erstellt die erforderlichen Tabellen in dieser Datenbank.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-116">The backplane will create the necessary tables in this database.</span></span>
2. <span data-ttu-id="9cdf6-117">Ihre Anwendung diese NuGet-Pakete hinzugefügt werden:</span><span class="sxs-lookup"><span data-stu-id="9cdf6-117">Add these NuGet packages to your application:</span></span> 

    - [<span data-ttu-id="9cdf6-118">Microsoft.AspNet.SignalR</span><span class="sxs-lookup"><span data-stu-id="9cdf6-118">Microsoft.AspNet.SignalR</span></span>](http://nuget.org/packages/Microsoft.AspNet.SignalR)
    - [<span data-ttu-id="9cdf6-119">Microsoft.AspNet.SignalR.SqlServer</span><span class="sxs-lookup"><span data-stu-id="9cdf6-119">Microsoft.AspNet.SignalR.SqlServer</span></span>](http://nuget.org/packages/Microsoft.AspNet.SignalR.SqlServer)
3. <span data-ttu-id="9cdf6-120">Erstellen einer SignalR-Anwendung.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-120">Create a SignalR application.</span></span>
4. <span data-ttu-id="9cdf6-121">Fügen Sie den folgenden Code, um "Global.asax" So konfigurieren Sie die Backplane:</span><span class="sxs-lookup"><span data-stu-id="9cdf6-121">Add the following code to Global.asax to configure the backplane:</span></span> 

    [!code-csharp[Main](scaleout-with-sql-server/samples/sample1.cs)]

## <a name="configure-the-database"></a><span data-ttu-id="9cdf6-122">Konfigurieren Sie die Datenbank</span><span class="sxs-lookup"><span data-stu-id="9cdf6-122">Configure the Database</span></span>

<span data-ttu-id="9cdf6-123">Entscheiden Sie, ob die Anwendung Windows-Authentifizierung oder SQL Server-Authentifizierung für den Datenbankzugriff verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-123">Decide whether the application will use Windows authentication or SQL Server authentication to access the database.</span></span> <span data-ttu-id="9cdf6-124">Unabhängig davon, stellen Sie sicher, dass der Datenbankbenutzer hat Berechtigungen zum Anmelden, Erstellen von Schemas und Tabellen erstellen.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-124">Regardless, make sure the database user has permissions to log in, create schemas, and create tables.</span></span>

<span data-ttu-id="9cdf6-125">Erstellen Sie eine neue Datenbank für die Backplane verwenden.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-125">Create a new database for the backplane to use.</span></span> <span data-ttu-id="9cdf6-126">Sie können der Datenbank auf einen beliebigen Namen geben.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-126">You can give the database any name.</span></span> <span data-ttu-id="9cdf6-127">Sie müssen keine Tabellen in der Datenbank zu erstellen. der Rückwand erstellt die erforderlichen Tabellen.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-127">You don't need to create any tables in the database; the backplane will create the necessary tables.</span></span>

![](scaleout-with-sql-server/_static/image2.png)

## <a name="enable-service-broker"></a><span data-ttu-id="9cdf6-128">Aktivieren Sie Service Broker</span><span class="sxs-lookup"><span data-stu-id="9cdf6-128">Enable Service Broker</span></span>

<span data-ttu-id="9cdf6-129">Es wird empfohlen, zum Aktivieren von Service Broker für die Datenbank Rückwandplatine.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-129">It is recommended to enable Service Broker for the backplane database.</span></span> <span data-ttu-id="9cdf6-130">Service Broker bietet systemeigene Unterstützung für Messaging- und Warteschlangen in SQL Server, letztere der Rückwand Updates noch effizienter zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-130">Service Broker provides native support for messaging and queuing in SQL Server, which lets the backplane receive updates more efficiently.</span></span> <span data-ttu-id="9cdf6-131">(Allerdings funktioniert der Rückwand auch ohne Service Broker.)</span><span class="sxs-lookup"><span data-stu-id="9cdf6-131">(However, the backplane also works without Service Broker.)</span></span>

<span data-ttu-id="9cdf6-132">Um zu überprüfen, ob Service Broker aktiviert ist, Fragen Sie die **ist\_Broker\_aktiviert** Spalte in der **sys.databases** -Katalogsicht angezeigt.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-132">To check whether Service Broker is enabled, query the **is\_broker\_enabled** column in the **sys.databases** catalog view.</span></span>

[!code-sql[Main](scaleout-with-sql-server/samples/sample2.sql)]

![](scaleout-with-sql-server/_static/image3.png)

<span data-ttu-id="9cdf6-133">Um Service Broker zu aktivieren, verwenden Sie die folgende SQL-Abfrage:</span><span class="sxs-lookup"><span data-stu-id="9cdf6-133">To enable Service Broker, use the following SQL query:</span></span>

[!code-sql[Main](scaleout-with-sql-server/samples/sample3.sql)]

> [!NOTE]
> <span data-ttu-id="9cdf6-134">Wenn diese Abfrage angezeigt wird, ein Deadlock auftreten, stellen Sie sicher, dass keine Programme, die mit der Datenbank verbunden sind.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-134">If this query appears to deadlock, make sure there are no applications connected to the DB.</span></span>


<span data-ttu-id="9cdf6-135">Wenn Sie die Ablaufverfolgung aktiviert haben, die ablaufverfolgungen ebenfalls angezeigt, ob Service Broker aktiviert ist.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-135">If you have enabled tracing, the traces will also show whether Service Broker is enabled.</span></span>

## <a name="create-a-signalr-application"></a><span data-ttu-id="9cdf6-136">Erstellen einer SignalR-Anwendung</span><span class="sxs-lookup"><span data-stu-id="9cdf6-136">Create a SignalR Application</span></span>

<span data-ttu-id="9cdf6-137">Erstellen einer SignalR-Anwendung entweder mit diesen Lernprogrammen:</span><span class="sxs-lookup"><span data-stu-id="9cdf6-137">Create a SignalR application by following either of these tutorials:</span></span>

- [<span data-ttu-id="9cdf6-138">Erste Schritte mit SignalR</span><span class="sxs-lookup"><span data-stu-id="9cdf6-138">Getting Started with SignalR</span></span>](../getting-started/tutorial-getting-started-with-signalr.md)
- [<span data-ttu-id="9cdf6-139">Erste Schritte mit SignalR und MVC 4</span><span class="sxs-lookup"><span data-stu-id="9cdf6-139">Getting Started with SignalR and MVC 4</span></span>](tutorial-getting-started-with-signalr-and-mvc-4.md)

<span data-ttu-id="9cdf6-140">Ändern Sie als Nächstes die chatanwendung zur Unterstützung von Warteschlangen für horizontale Skalierung mit SQL Server.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-140">Next, we'll modify the chat application to support scaleout with SQL Server.</span></span> <span data-ttu-id="9cdf6-141">Fügen Sie zuerst das SignalR.SqlServer NuGet-Paket zum Projekt hinzu.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-141">First, add the SignalR.SqlServer NuGet package to your project.</span></span> <span data-ttu-id="9cdf6-142">In Visual Studio aus der **Tools** klicken Sie im Menü **Bibliothekspaket-Manager**, und wählen Sie dann **Package Manager Console**.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-142">In Visual Studio, from the **Tools** menu, select **Library Package Manager**, then select **Package Manager Console**.</span></span> <span data-ttu-id="9cdf6-143">Geben Sie im Fenster Paket-Manager-Konsole den folgenden Befehl aus:</span><span class="sxs-lookup"><span data-stu-id="9cdf6-143">In the Package Manager Console window, enter the following command:</span></span>

[!code-powershell[Main](scaleout-with-sql-server/samples/sample4.ps1)]

<span data-ttu-id="9cdf6-144">Als Nächstes öffnen Sie die Datei "Global.asax".</span><span class="sxs-lookup"><span data-stu-id="9cdf6-144">Next, open the Global.asax file.</span></span> <span data-ttu-id="9cdf6-145">Fügen Sie folgenden Code, der **Anwendung\_starten** Methode:</span><span class="sxs-lookup"><span data-stu-id="9cdf6-145">Add the following code to the **Application\_Start** method:</span></span>

[!code-csharp[Main](scaleout-with-sql-server/samples/sample5.cs)]

## <a name="deploy-and-run-the-application"></a><span data-ttu-id="9cdf6-146">Bereitstellen und Ausführen der Anwendung</span><span class="sxs-lookup"><span data-stu-id="9cdf6-146">Deploy and Run the Application</span></span>

<span data-ttu-id="9cdf6-147">Vorbereiten der Windows Server-Instanzen, die SignalR-Anwendung bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-147">Prepare your Windows Server instances to deploy the SignalR application.</span></span>

<span data-ttu-id="9cdf6-148">Fügen Sie die IIS-Rolle hinzu.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-148">Add the IIS role.</span></span> <span data-ttu-id="9cdf6-149">Schließen Sie "Anwendungsentwicklung"-Funktionen, einschließlich des WebSocket-Protokolls.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-149">Include "Application Development" features, including the WebSocket Protocol.</span></span>

![](scaleout-with-sql-server/_static/image4.png)

<span data-ttu-id="9cdf6-150">Außerdem enthalten Sie den Verwaltungsdienst (unter "Verwaltungstools" aufgeführt).</span><span class="sxs-lookup"><span data-stu-id="9cdf6-150">Also include the Management Service (listed under "Management Tools").</span></span>

![](scaleout-with-sql-server/_static/image5.png)

<span data-ttu-id="9cdf6-151">**Installieren Sie Web Deploy 3.0.**</span><span class="sxs-lookup"><span data-stu-id="9cdf6-151">**Install Web Deploy 3.0.**</span></span> <span data-ttu-id="9cdf6-152">Wenn Sie die IIS-Manager ausführen, werden Sie zum Installieren von Microsoft-Webplattform aufgefordert, oder Sie können [Herunterladen der Intstaller](https://go.microsoft.com/fwlink/?LinkId=255386).</span><span class="sxs-lookup"><span data-stu-id="9cdf6-152">When you run IIS Manager, it will prompt you to install Microsoft Web Platform, or you can [download the intstaller](https://go.microsoft.com/fwlink/?LinkId=255386).</span></span> <span data-ttu-id="9cdf6-153">Klicken Sie in der Webplattform-Installer Web Deploy suchen Sie, und installieren Sie Web Deploy 3.0</span><span class="sxs-lookup"><span data-stu-id="9cdf6-153">In the Platform Installer, search for Web Deploy and install Web Deploy 3.0</span></span>

![](scaleout-with-sql-server/_static/image6.png)

<span data-ttu-id="9cdf6-154">Überprüfen Sie, dass die Web-Management-Dienst ausgeführt wird.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-154">Check that the Web Management Service is running.</span></span> <span data-ttu-id="9cdf6-155">Wenn dies nicht der Fall ist, starten Sie den Dienst.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-155">If not, start the service.</span></span> <span data-ttu-id="9cdf6-156">(Wenn Webverwaltungsdienst in der Liste der Windows-Dienste nicht angezeigt wird, stellen Sie sicher, dass Sie den Management-Dienst installiert, wenn Sie die IIS-Rolle hinzugefügt.)</span><span class="sxs-lookup"><span data-stu-id="9cdf6-156">(If you don't see Web Management Service in the list of Windows services, make sure that you installed the Management Service when you added the IIS role.)</span></span>

<span data-ttu-id="9cdf6-157">Öffnen Sie zum Schluss Port 8172 für TCP.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-157">Finally, open port 8172 for TCP.</span></span> <span data-ttu-id="9cdf6-158">Dies ist der Port, den der Web Deploy-Tool verwendet.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-158">This is the port that the Web Deploy tool uses.</span></span>

<span data-ttu-id="9cdf6-159">Jetzt sind Sie bereit für die Bereitstellung von Visual Studio-Projekt vom Entwicklungscomputer auf den Server.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-159">Now you are ready to deploy the Visual Studio project from your development machine to the server.</span></span> <span data-ttu-id="9cdf6-160">Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste in der Projektmappe, und klicken Sie auf **veröffentlichen**.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-160">In Solution Explorer, right-click the solution and click **Publish**.</span></span>

<span data-ttu-id="9cdf6-161">Ausführlichere Dokumentation zu Web Deploy finden Sie unter [ASP.NET-Bereitstellung für Visual Studio und ASP.NET Web](../../../whitepapers/aspnet-web-deployment-content-map.md).</span><span class="sxs-lookup"><span data-stu-id="9cdf6-161">For more detailed documentation about web deployment, see [Web Deployment Content Map for Visual Studio and ASP.NET](../../../whitepapers/aspnet-web-deployment-content-map.md).</span></span>

<span data-ttu-id="9cdf6-162">Wenn Sie die Anwendung mit zwei Servern bereitstellen, können jede Instanz in einem separaten Browserfenster zu öffnen und sehen, dass jeder SignalR-Nachrichten aus den anderen erhält.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-162">If you deploy the application to two servers, you can open each instance in a separate browser window and see that they each receive SignalR messages from the other.</span></span> <span data-ttu-id="9cdf6-163">(Selbstverständlich würden in einer produktionsumgebung die beiden Servern hinter einem Lastenausgleich sit.)</span><span class="sxs-lookup"><span data-stu-id="9cdf6-163">(Of course, in a production environment, the two servers would sit behind a load balancer.)</span></span>

![](scaleout-with-sql-server/_static/image7.png)

<span data-ttu-id="9cdf6-164">Nachdem Sie die Anwendung ausführen, sehen Sie sich, dass SignalR automatisch Tabellen in der Datenbank erstellt wurde:</span><span class="sxs-lookup"><span data-stu-id="9cdf6-164">After you run the application, you can see that SignalR has automatically created tables in the database:</span></span>

![](scaleout-with-sql-server/_static/image8.png)

<span data-ttu-id="9cdf6-165">SignalR verwendet die Tabellen.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-165">SignalR manages the tables.</span></span> <span data-ttu-id="9cdf6-166">Solange die Anwendung bereitgestellt wurde, keine löschen Sie Zeilen, ändern Sie die Tabelle, usw.</span><span class="sxs-lookup"><span data-stu-id="9cdf6-166">As long as your application is deployed, don't delete rows, modify the table, and so forth.</span></span>