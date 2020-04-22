<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d9366-101">In diesem Lernprogramm erfahren Sie, wie Sie eine Node. js Express-Webanwendung erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="d9366-101">This tutorial teaches you how to build a Node.js Express web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="d9366-102">Wenn Sie das fertige Lernprogramm einfach herunterladen möchten, können Sie es auf zwei Arten herunterladen.</span><span class="sxs-lookup"><span data-stu-id="d9366-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="d9366-103">Laden Sie den [Knoten. js Schnellstart](https://developer.microsoft.com/graph/quick-start?platform=option-node) herunter, um in wenigen Minuten Arbeitscode zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="d9366-103">Download the [Node.js quick start](https://developer.microsoft.com/graph/quick-start?platform=option-node) to get working code in minutes.</span></span>
> - <span data-ttu-id="d9366-104">Laden Sie das [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp)herunter, oder Klonen Sie es.</span><span class="sxs-lookup"><span data-stu-id="d9366-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d9366-105">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="d9366-105">Prerequisites</span></span>

<span data-ttu-id="d9366-106">Bevor Sie mit dieser Demo beginnen, sollte [node. js](https://nodejs.org) auf Ihrem Entwicklungscomputer installiert sein.</span><span class="sxs-lookup"><span data-stu-id="d9366-106">Before you start this demo, you should have [Node.js](https://nodejs.org) installed on your development machine.</span></span> <span data-ttu-id="d9366-107">Wenn Sie nicht über Node. js verfügen, besuchen Sie den vorherigen Link für Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="d9366-107">If you do not have Node.js, visit the previous link for download options.</span></span>

<span data-ttu-id="d9366-108">Sie sollten auch über ein persönliches Microsoft-Konto mit einem Postfach auf Outlook.com oder ein Microsoft-Arbeits-oder Schulkonto verfügen.</span><span class="sxs-lookup"><span data-stu-id="d9366-108">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="d9366-109">Wenn Sie kein Microsoft-Konto haben, gibt es mehrere Optionen, um ein kostenloses Konto zu erhalten:</span><span class="sxs-lookup"><span data-stu-id="d9366-109">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="d9366-110">Sie können [sich für ein neues persönliches Microsoft-Konto registrieren](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="d9366-110">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="d9366-111">Sie können sich [für das Office 365 Entwicklerprogramm registrieren](https://developer.microsoft.com/office/dev-program) , um ein kostenloses Office 365-Abonnement zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="d9366-111">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="d9366-112">Dieses Lernprogramm wurde mit Node Version 12.6.1 geschrieben.</span><span class="sxs-lookup"><span data-stu-id="d9366-112">This tutorial was written with Node version 12.6.1.</span></span> <span data-ttu-id="d9366-113">Die Schritte in diesem Leitfaden funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.</span><span class="sxs-lookup"><span data-stu-id="d9366-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="watch-the-tutorial"></a><span data-ttu-id="d9366-114">Das Lernprogramm ansehen</span><span class="sxs-lookup"><span data-stu-id="d9366-114">Watch the tutorial</span></span>

<span data-ttu-id="d9366-115">Dieses Modul wurde aufgezeichnet und steht im YouTube-Kanal für die Office-Entwicklung zur Verfügung.</span><span class="sxs-lookup"><span data-stu-id="d9366-115">This module has been recorded and is available in the Office Development YouTube channel.</span></span>

<!-- markdownlint-disable MD033 MD034 -->
<br/>

> [!VIDEO https://www.youtube-nocookie.com/embed/n6q8Cm-pTYY]
<!-- markdownlint-enable MD033 MD034 -->

## <a name="feedback"></a><span data-ttu-id="d9366-116">Feedback</span><span class="sxs-lookup"><span data-stu-id="d9366-116">Feedback</span></span>

<span data-ttu-id="d9366-117">Geben Sie Feedback zu diesem Lernprogramm im [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp)an.</span><span class="sxs-lookup"><span data-stu-id="d9366-117">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp).</span></span>
