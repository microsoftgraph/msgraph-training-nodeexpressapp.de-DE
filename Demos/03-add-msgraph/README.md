# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="abd50-101">Ausführen des abgeschlossenen Projekts</span><span class="sxs-lookup"><span data-stu-id="abd50-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="abd50-102">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="abd50-102">Prerequisites</span></span>

<span data-ttu-id="abd50-103">Zum Ausführen des abgeschlossenen Projekts in diesem Ordner benötigen Sie Folgendes:</span><span class="sxs-lookup"><span data-stu-id="abd50-103">To run the completed project in this folder, you need the following:</span></span>

- <span data-ttu-id="abd50-104">[Node. js](https://nodejs.org) ist auf dem Entwicklungscomputer installiert.</span><span class="sxs-lookup"><span data-stu-id="abd50-104">[Node.js](https://nodejs.org) installed on your development machine.</span></span> <span data-ttu-id="abd50-105">Wenn Sie nicht über Node. js verfügen, besuchen Sie den vorherigen Link für Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="abd50-105">If you do not have Node.js, visit the previous link for download options.</span></span> <span data-ttu-id="abd50-106">(**Hinweis:** dieses Tutorial wurde mit Node Version 10.7.0 geschrieben.</span><span class="sxs-lookup"><span data-stu-id="abd50-106">(**Note:** This tutorial was written with Node version 10.7.0.</span></span> <span data-ttu-id="abd50-107">Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, die jedoch nicht getestet wurden.)</span><span class="sxs-lookup"><span data-stu-id="abd50-107">The steps in this guide may work with other versions, but that has not been tested.)</span></span>
- <span data-ttu-id="abd50-108">Entweder ein persönliches Microsoft-Konto mit einem Postfach auf Outlook.com oder ein Microsoft-Geschäfts-oder Schulkonto.</span><span class="sxs-lookup"><span data-stu-id="abd50-108">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

<span data-ttu-id="abd50-109">Wenn Sie nicht über ein Microsoft-Konto verfügen, gibt es eine Reihe von Optionen, um ein kostenloses Konto zu erhalten:</span><span class="sxs-lookup"><span data-stu-id="abd50-109">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="abd50-110">Sie können [sich für ein neues persönliches Microsoft-Konto registrieren](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="abd50-110">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="abd50-111">Sie können [sich für das office 365-Entwicklerprogramm anmelden](https://developer.microsoft.com/office/dev-program) , um ein kostenloses Office 365-Abonnement zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="abd50-111">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a><span data-ttu-id="abd50-112">Registrieren einer Webanwendung mit dem Azure Active Directory Admin Center</span><span class="sxs-lookup"><span data-stu-id="abd50-112">Register a web application with the Azure Active Directory admin center</span></span>

1. <span data-ttu-id="abd50-113">Öffnen Sie einen Browser, und navigieren Sie zum [Azure Active Directory Admin Center](https://aad.portal.azure.com).</span><span class="sxs-lookup"><span data-stu-id="abd50-113">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="abd50-114">Melden Sie sich über ein **persönliches Konto** (aka: Microsoft-Konto) oder ein Geschäfts- **oder Schulkonto**an.</span><span class="sxs-lookup"><span data-stu-id="abd50-114">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="abd50-115">Wählen Sie **Azure Active Directory** in der linken Navigationsleiste aus, und wählen Sie dann **App-Registrierungen (Vorschau)** unter **Manage**aus.</span><span class="sxs-lookup"><span data-stu-id="abd50-115">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations (Preview)** under **Manage**.</span></span>

    ![<span data-ttu-id="abd50-116">Screenshot der APP-Registrierungen</span><span class="sxs-lookup"><span data-stu-id="abd50-116">A screenshot of the App registrations</span></span> ](/tutorial/images/aad-portal-app-registrations.png)

1. <span data-ttu-id="abd50-117">Wählen Sie **neue Registrierung**aus.</span><span class="sxs-lookup"><span data-stu-id="abd50-117">Select **New registration**.</span></span> <span data-ttu-id="abd50-118">Legen Sie auf der Seite **Anwendung registrieren** die Werte wie folgt fest.</span><span class="sxs-lookup"><span data-stu-id="abd50-118">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="abd50-119">Legen \*\*\*\* Sie Name `Node.js Graph Tutorial`auf fest.</span><span class="sxs-lookup"><span data-stu-id="abd50-119">Set **Name** to `Node.js Graph Tutorial`.</span></span>
    - <span data-ttu-id="abd50-120">Legen Sie **unterstützte Kontotypen** auf **Konten in einem beliebigen Organisations Verzeichnis und persönlichen Microsoft-Konten**fest.</span><span class="sxs-lookup"><span data-stu-id="abd50-120">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="abd50-121">Legen Sie unter umLeitungs- **URI**die erste Dropdown `Web` Liste auf fest, und `http://localhost:3000/auth/callback`legen Sie den Wert auf fest.</span><span class="sxs-lookup"><span data-stu-id="abd50-121">Under **Redirect URI**, set the first drop-down to `Web` and set the value to `http://localhost:3000/auth/callback`.</span></span>

    ![Screenshot der Seite "Registrieren einer Anwendung"](/tutorial/images/aad-register-an-app.png)

1. <span data-ttu-id="abd50-123">Wählen Sie **registrieren**aus.</span><span class="sxs-lookup"><span data-stu-id="abd50-123">Choose **Register**.</span></span> <span data-ttu-id="abd50-124">Kopieren Sie auf der Seite " **node. js Graph Tutorial** " den Wert der **Application (Client)-ID** , und speichern Sie Sie, Sie benötigen Sie im nächsten Schritt.</span><span class="sxs-lookup"><span data-stu-id="abd50-124">On the **Node.js Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Screenshot der Anwendungs-ID der neuen App-Registrierung](/tutorial/images/aad-application-id.png)

1. <span data-ttu-id="abd50-126">Wählen Sie unter **Manage**die Option **Authentication** aus.</span><span class="sxs-lookup"><span data-stu-id="abd50-126">Select **Authentication** under **Manage**.</span></span> <span data-ttu-id="abd50-127">Suchen Sie den **impliziten Grant** -Abschnitt, und aktivieren Sie **ID-Token**.</span><span class="sxs-lookup"><span data-stu-id="abd50-127">Locate the **Implicit grant** section and enable **ID tokens**.</span></span> <span data-ttu-id="abd50-128">Wählen Sie **Speichern** aus.</span><span class="sxs-lookup"><span data-stu-id="abd50-128">Choose **Save**.</span></span>

    ![Screenshot des impliziten Grant-Abschnitts](/tutorial/images/aad-implicit-grant.png)

1. <span data-ttu-id="abd50-130">Wählen Sie unter **Manage**die Option **Certificates & Secrets** aus.</span><span class="sxs-lookup"><span data-stu-id="abd50-130">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="abd50-131">Klicken Sie auf die Schaltfläche **neuen geheimen Client Schlüssel** .</span><span class="sxs-lookup"><span data-stu-id="abd50-131">Select the **New client secret** button.</span></span> <span data-ttu-id="abd50-132">Geben Sie einen Wert in **Description** ein, und wählen Sie eine der Optionen für **Expires** und wählen Sie **Hinzufügen**aus.</span><span class="sxs-lookup"><span data-stu-id="abd50-132">Enter a value in **Description** and select one of the options for **Expires** and choose **Add**.</span></span>

    ![Screenshot des Dialogfelds zum Hinzufügen eines geheimen Clients](/tutorial/images/aad-new-client-secret.png)

1. <span data-ttu-id="abd50-134">Kopieren Sie den Client geheimen Wert, bevor Sie diese Seite verlassen.</span><span class="sxs-lookup"><span data-stu-id="abd50-134">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="abd50-135">Sie benötigen Sie im nächsten Schritt.</span><span class="sxs-lookup"><span data-stu-id="abd50-135">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="abd50-136">Dieser geheime Client Schlüssel wird nie wieder angezeigt, stellen Sie daher sicher, dass Sie ihn jetzt kopieren.</span><span class="sxs-lookup"><span data-stu-id="abd50-136">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Screenshot des neu hinzugefügten geheimen Clients](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a><span data-ttu-id="abd50-138">Konfigurieren des Beispiels</span><span class="sxs-lookup"><span data-stu-id="abd50-138">Configure the sample</span></span>

1. <span data-ttu-id="abd50-139">Benennen Sie `.env.example` die Datei `.env`in um.</span><span class="sxs-lookup"><span data-stu-id="abd50-139">Rename the `.env.example` file to `.env`.</span></span>
1. <span data-ttu-id="abd50-140">Bearbeiten Sie `.env` die Datei, und nehmen Sie die folgenden Änderungen vor.</span><span class="sxs-lookup"><span data-stu-id="abd50-140">Edit the `.env` file and make the following changes.</span></span>
    1. <span data-ttu-id="abd50-141">Ersetzen `YOUR_APP_ID_HERE` Sie durch die **Anwendungs-ID** , die Sie im App-Registrierungs Portal erhalten haben.</span><span class="sxs-lookup"><span data-stu-id="abd50-141">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
    1. <span data-ttu-id="abd50-142">Ersetzen `YOUR_APP_PASSWORD_HERE` Sie durch das Kennwort, das Sie im App-Registrierungs Portal erhalten haben.</span><span class="sxs-lookup"><span data-stu-id="abd50-142">Replace `YOUR_APP_PASSWORD_HERE` with the password you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="abd50-143">Navigieren Sie in der Befehlszeilenschnittstelle zu diesem Verzeichnis, und führen Sie den folgenden Befehl aus, um die Anforderungen zu installieren.</span><span class="sxs-lookup"><span data-stu-id="abd50-143">In your command-line interface (CLI), navigate to this directory and run the following command to install requirements.</span></span>

    ```Shell
    npm install
    ```

## <a name="run-the-sample"></a><span data-ttu-id="abd50-144">Ausführen des Beispiels</span><span class="sxs-lookup"><span data-stu-id="abd50-144">Run the sample</span></span>

1. <span data-ttu-id="abd50-145">Führen Sie den folgenden Befehl in der CLI aus, um die Anwendung zu starten.</span><span class="sxs-lookup"><span data-stu-id="abd50-145">Run the following command in your CLI to start the application.</span></span>

    ```Shell
    npm start
    ```

1. <span data-ttu-id="abd50-146">Öffnen Sie einen Browser, und navigieren Sie zu `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="abd50-146">Open a browser and browse to `http://localhost:3000`.</span></span>