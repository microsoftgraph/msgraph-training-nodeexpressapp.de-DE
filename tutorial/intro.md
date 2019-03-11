<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6cf90-101">In diesem Lernprogramm erfahren Sie, wie Sie eine Web-App für Node. js Express erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="6cf90-101">This tutorial teaches you how to build a Node.js Express web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="6cf90-102">Wenn Sie das fertige Tutorial lieber herunterladen möchten, können Sie es auf zweierlei Weise herunterladen.</span><span class="sxs-lookup"><span data-stu-id="6cf90-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="6cf90-103">Laden Sie den [Schnellstart von Node. js](https://developer.microsoft.com/graph/quick-start?platform=option-node) herunter, um Arbeitscode in Minuten zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="6cf90-103">Download the [Node.js quick start](https://developer.microsoft.com/graph/quick-start?platform=option-node) to get working code in minutes.</span></span>
> - <span data-ttu-id="6cf90-104">Laden oder Klonen Sie das [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp).</span><span class="sxs-lookup"><span data-stu-id="6cf90-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6cf90-105">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="6cf90-105">Prerequisites</span></span>

<span data-ttu-id="6cf90-106">Bevor Sie mit dieser Demo beginnen, sollte [node. js](https://nodejs.org) auf dem Entwicklungscomputer installiert sein.</span><span class="sxs-lookup"><span data-stu-id="6cf90-106">Before you start this demo, you should have [Node.js](https://nodejs.org) installed on your development machine.</span></span> <span data-ttu-id="6cf90-107">Wenn Sie nicht über Node. js verfügen, besuchen Sie den vorherigen Link für Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="6cf90-107">If you do not have Node.js, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="6cf90-108">Dieses Lernprogramm wurde mit Node Version 10.7.0 geschrieben.</span><span class="sxs-lookup"><span data-stu-id="6cf90-108">This tutorial was written with Node version 10.7.0.</span></span> <span data-ttu-id="6cf90-109">Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.</span><span class="sxs-lookup"><span data-stu-id="6cf90-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="6cf90-110">Feedback</span><span class="sxs-lookup"><span data-stu-id="6cf90-110">Feedback</span></span>

<span data-ttu-id="6cf90-111">Feedback zu diesem Tutorial finden Sie im GitHub- [Repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp).</span><span class="sxs-lookup"><span data-stu-id="6cf90-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp).</span></span>