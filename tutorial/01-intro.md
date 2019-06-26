<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b397c-101">In diesem Lernprogramm erfahren Sie, wie Sie eine Node. js Express-Webanwendung erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="b397c-101">This tutorial teaches you how to build a Node.js Express web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="b397c-102">Wenn Sie das fertige Lernprogramm einfach herunterladen möchten, können Sie es auf zwei Arten herunterladen.</span><span class="sxs-lookup"><span data-stu-id="b397c-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="b397c-103">Laden Sie den [Knoten. js Schnellstart](https://developer.microsoft.com/graph/quick-start?platform=option-node) herunter, um in wenigen Minuten Arbeitscode zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="b397c-103">Download the [Node.js quick start](https://developer.microsoft.com/graph/quick-start?platform=option-node) to get working code in minutes.</span></span>
> - <span data-ttu-id="b397c-104">Laden Sie das [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp)herunter, oder Klonen Sie es.</span><span class="sxs-lookup"><span data-stu-id="b397c-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="b397c-105">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="b397c-105">Prerequisites</span></span>

<span data-ttu-id="b397c-106">Bevor Sie mit dieser Demo beginnen, sollte [node. js](https://nodejs.org) auf Ihrem Entwicklungscomputer installiert sein.</span><span class="sxs-lookup"><span data-stu-id="b397c-106">Before you start this demo, you should have [Node.js](https://nodejs.org) installed on your development machine.</span></span> <span data-ttu-id="b397c-107">Wenn Sie nicht über Node. js verfügen, besuchen Sie den vorherigen Link für Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="b397c-107">If you do not have Node.js, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="b397c-108">Dieses Lernprogramm wurde mit Node Version 10.15.3 geschrieben.</span><span class="sxs-lookup"><span data-stu-id="b397c-108">This tutorial was written with Node version 10.15.3.</span></span> <span data-ttu-id="b397c-109">Die Schritte in diesem Leitfaden funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.</span><span class="sxs-lookup"><span data-stu-id="b397c-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="watch-the-tutorial"></a><span data-ttu-id="b397c-110">Das Lernprogramm ansehen</span><span class="sxs-lookup"><span data-stu-id="b397c-110">Watch the tutorial</span></span>

<span data-ttu-id="b397c-111">Dieses Modul wurde aufgezeichnet und steht im YouTube-Kanal für die Office-Entwicklung zur Verfügung.</span><span class="sxs-lookup"><span data-stu-id="b397c-111">This module has been recorded and is available in the Office Development YouTube channel.</span></span>

<!-- markdownlint-disable MD033 MD034 -->
<br/>

> [!VIDEO https://www.youtube-nocookie.com/embed/n6q8Cm-pTYY]
<!-- markdownlint-enable MD033 MD034 -->

## <a name="feedback"></a><span data-ttu-id="b397c-112">Feedback</span><span class="sxs-lookup"><span data-stu-id="b397c-112">Feedback</span></span>

<span data-ttu-id="b397c-113">Geben Sie Feedback zu diesem Lernprogramm im [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp)an.</span><span class="sxs-lookup"><span data-stu-id="b397c-113">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp).</span></span>
