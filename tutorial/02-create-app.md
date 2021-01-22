<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="90ad7-101">In dieser Übung verwenden Sie [Express zum](http://expressjs.com/) Erstellen einer Web-App.</span><span class="sxs-lookup"><span data-stu-id="90ad7-101">In this exercise you will use [Express](http://expressjs.com/) to build a web app.</span></span>

1. <span data-ttu-id="90ad7-102">Öffnen Sie Ihre CLI, navigieren Sie zu einem Verzeichnis, in dem Sie über die Rechte zum Erstellen von Dateien verfügen, und führen Sie den folgenden Befehl aus, um eine neue Express-App zu erstellen, die [Handlebars](http://handlebarsjs.com/) als Renderingmodul verwendet.</span><span class="sxs-lookup"><span data-stu-id="90ad7-102">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Express app that uses [Handlebars](http://handlebarsjs.com/) as the rendering engine.</span></span>

    ```Shell
    npx express-generator@4.16.1 --hbs graph-tutorial
    ```

    <span data-ttu-id="90ad7-103">Der Express-Generator erstellt ein neues Verzeichnis namens `graph-tutorial` und erstellt ein Gerüst für eine Express-App.</span><span class="sxs-lookup"><span data-stu-id="90ad7-103">The Express generator creates a new directory called `graph-tutorial` and scaffolds an Express app.</span></span>

1. <span data-ttu-id="90ad7-104">Navigieren Sie zum `graph-tutorial` Verzeichnis, und geben Sie den folgenden Befehl ein, um Abhängigkeiten zu installieren.</span><span class="sxs-lookup"><span data-stu-id="90ad7-104">Navigate to the `graph-tutorial` directory and enter the following command to install dependencies.</span></span>

    ```Shell
    npm install
    ```

1. <span data-ttu-id="90ad7-105">Führen Sie den folgenden Befehl aus, um Knotenpakete mit gemeldeten Sicherheitsrisiken zu aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="90ad7-105">Run the following command to update Node packages with reported vulnerabilities.</span></span>

    ```Shell
    npm audit fix
    ```

1. <span data-ttu-id="90ad7-106">Führen Sie den folgenden Befehl aus, um die Version von Express und andere Abhängigkeiten zu aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="90ad7-106">Run the following command to update the version of Express and other dependencies.</span></span>

    ```Shell
    npm install express@4.17.1 http-errors@1.8.0 morgan@1.10.0 debug@4.1.1
    ```

1. <span data-ttu-id="90ad7-107">Verwenden Sie den folgenden Befehl, um einen lokalen Webserver zu starten.</span><span class="sxs-lookup"><span data-stu-id="90ad7-107">Use the following command to start a local web server.</span></span>

    ```Shell
    npm start
    ```

1. <span data-ttu-id="90ad7-108">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="90ad7-108">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="90ad7-109">Wenn alles funktioniert, wird die Meldung "Willkommen bei Express" angezeigt.</span><span class="sxs-lookup"><span data-stu-id="90ad7-109">If everything is working, you will see a "Welcome to Express" message.</span></span> <span data-ttu-id="90ad7-110">Wenn diese Meldung nicht angezeigt wird, lesen Sie die Anleitung für [die ersten Schritte von Express.](http://expressjs.com/starter/generator.html)</span><span class="sxs-lookup"><span data-stu-id="90ad7-110">If you don't see that message, check the [Express getting started guide](http://expressjs.com/starter/generator.html).</span></span>

## <a name="install-node-packages"></a><span data-ttu-id="90ad7-111">Installieren von Knotenpaketen</span><span class="sxs-lookup"><span data-stu-id="90ad7-111">Install Node packages</span></span>

<span data-ttu-id="90ad7-112">Installieren Sie vor dem Wechsel einige zusätzliche Pakete, die Sie später verwenden werden:</span><span class="sxs-lookup"><span data-stu-id="90ad7-112">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="90ad7-113">[dotenv](https://github.com/motdotla/dotenv) zum Laden von Werten aus einer ENV-Datei.</span><span class="sxs-lookup"><span data-stu-id="90ad7-113">[dotenv](https://github.com/motdotla/dotenv) for loading values from a .env file.</span></span>
- <span data-ttu-id="90ad7-114">[Moment](https://github.com/moment/moment/) für die Formatierung von Datums-/Uhrzeitwerten.</span><span class="sxs-lookup"><span data-stu-id="90ad7-114">[moment](https://github.com/moment/moment/) for formatting date/time values.</span></span>
- <span data-ttu-id="90ad7-115">[windows-iana](https://github.com/rubenillodo/windows-iana) für die Übersetzung von Windows-Zeitzonennamen in IANA-Zeitzonen-IDs.</span><span class="sxs-lookup"><span data-stu-id="90ad7-115">[windows-iana](https://github.com/rubenillodo/windows-iana) for translating Windows time zone names to IANA time zone IDs.</span></span>
- <span data-ttu-id="90ad7-116">[connect-flash](https://github.com/jaredhanson/connect-flash) to flash error messages in the app.</span><span class="sxs-lookup"><span data-stu-id="90ad7-116">[connect-flash](https://github.com/jaredhanson/connect-flash) to flash error messages in the app.</span></span>
- <span data-ttu-id="90ad7-117">[express-session](https://github.com/expressjs/session) zum Speichern von Werten in einer serverseitigen Speichersitzung.</span><span class="sxs-lookup"><span data-stu-id="90ad7-117">[express-session](https://github.com/expressjs/session) to store values in an in-memory server-side session.</span></span>
- <span data-ttu-id="90ad7-118">[express-promise-router](https://github.com/express-promise-router/express-promise-router) to allow route handlers to return a Promise.</span><span class="sxs-lookup"><span data-stu-id="90ad7-118">[express-promise-router](https://github.com/express-promise-router/express-promise-router) to allow route handlers to return a Promise.</span></span>
- <span data-ttu-id="90ad7-119">[Express validator](https://github.com/express-validator/express-validator) für die Analyse und Validierung von Formulardaten.</span><span class="sxs-lookup"><span data-stu-id="90ad7-119">[express-validator](https://github.com/express-validator/express-validator) for parsing and validating form data.</span></span>
- <span data-ttu-id="90ad7-120">[msal-node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) zum Authentifizieren und Abrufen von Zugriffstoken.</span><span class="sxs-lookup"><span data-stu-id="90ad7-120">[msal-node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) for authenticating and getting access tokens.</span></span>
- <span data-ttu-id="90ad7-121">[uuid,](https://github.com/uuidjs/uuid) die von "msal-node" zum Generieren von GUIDs verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="90ad7-121">[uuid](https://github.com/uuidjs/uuid) used by msal-node to generate GUIDs.</span></span>
- <span data-ttu-id="90ad7-122">[microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) for making calls to Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="90ad7-122">[microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="90ad7-123">[isomorph-fetch zum](https://github.com/matthew-andrews/isomorphic-fetch) Polyfilling des Abrufs für Node.</span><span class="sxs-lookup"><span data-stu-id="90ad7-123">[isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch) to polyfill the fetch for Node.</span></span> <span data-ttu-id="90ad7-124">Für die Bibliothek ist ein Polyfill zum Abrufen `microsoft-graph-client` erforderlich.</span><span class="sxs-lookup"><span data-stu-id="90ad7-124">A fetch polyfill is required for the `microsoft-graph-client` library.</span></span> <span data-ttu-id="90ad7-125">Weitere Informationen finden Sie im [Microsoft Graph-JavaScript-Clientbibliothekswiki.](https://github.com/microsoftgraph/msgraph-sdk-javascript/wiki/Migration-from-1.x.x-to-2.x.x#polyfill-only-when-required)</span><span class="sxs-lookup"><span data-stu-id="90ad7-125">See the [Microsoft Graph JavaScript client library wiki](https://github.com/microsoftgraph/msgraph-sdk-javascript/wiki/Migration-from-1.x.x-to-2.x.x#polyfill-only-when-required) for more information.</span></span>

1. <span data-ttu-id="90ad7-126">Führen Sie den folgenden Befehl in Der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="90ad7-126">Run the following command in your CLI.</span></span>

    ```Shell
    npm install dotenv@8.2.0 moment@2.29.1 moment-timezone@0.5.31 connect-flash@0.1.1 express-session@1.17.1 express-promise-router@4.0.1 isomorphic-fetch@3.0.0
    npm install @azure/msal-node@1.0.0-beta.0 @microsoft/microsoft-graph-client@2.1.1 windows-iana@4.2.1 express-validator@6.6.1 uuid@8.3.1
    ```

    > [!TIP]
    > <span data-ttu-id="90ad7-127">Beim Versuch, diese Pakete unter Windows zu installieren, wird möglicherweise die folgende Fehlermeldung angezeigt.</span><span class="sxs-lookup"><span data-stu-id="90ad7-127">Windows users may get the following error message when trying to install these packages on Windows.</span></span>
    >
    > ```Shell
    > gyp ERR! stack Error: Can't find Python executable "python", you can set the PYTHON env variable.
    > ```
    >
    > <span data-ttu-id="90ad7-128">Führen Sie zum Beheben des Fehlers den folgenden Befehl aus, um die Windows Build Tools mithilfe eines Terminalfensters mit erhöhten Rechten (Administrator) zu installieren, in dem die VS Build Tools und Python installiert werden.</span><span class="sxs-lookup"><span data-stu-id="90ad7-128">To resolve the error, run the following command to install the Windows Build Tools using an elevated (Administrator) terminal window which installs the VS Build Tools and Python.</span></span>
    >
    > ```Shell
    > npm install --global --production windows-build-tools
    > ```

1. <span data-ttu-id="90ad7-129">Aktualisieren Sie die Anwendung so, dass sie die `connect-flash` `express-session` Middleware verwendet.</span><span class="sxs-lookup"><span data-stu-id="90ad7-129">Update the application to use the `connect-flash` and `express-session` middleware.</span></span> <span data-ttu-id="90ad7-130">Öffnen **Sie ./app.js,** und fügen Sie die folgende Anweisung `require` am Oberen Rand der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="90ad7-130">Open **./app.js** and add the following `require` statement to the top of the file.</span></span>

    ```javascript
    const session = require('express-session');
    const flash = require('connect-flash');
    const msal = require('@azure/msal-node');
    ```

1. <span data-ttu-id="90ad7-131">Fügen Sie den folgenden Code unmittelbar hinter der Zeile `var app = express();` hinzu.</span><span class="sxs-lookup"><span data-stu-id="90ad7-131">Add the following code immediately after the `var app = express();` line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="SessionSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="90ad7-132">Entwerfen der App</span><span class="sxs-lookup"><span data-stu-id="90ad7-132">Design the app</span></span>

<span data-ttu-id="90ad7-133">In diesem Abschnitt implementieren Sie die Benutzeroberfläche für die App.</span><span class="sxs-lookup"><span data-stu-id="90ad7-133">In this section you will implement the UI for the app.</span></span>

1. <span data-ttu-id="90ad7-134">Öffnen **Sie ./views/layout.layoutss,** und ersetzen Sie den gesamten Inhalt durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="90ad7-134">Open **./views/layout.hbs** and replace the entire contents with the following code.</span></span>

    :::code language="html" source="../demo/graph-tutorial/views/layout.hbs" id="LayoutSnippet":::

    <span data-ttu-id="90ad7-135">Dieser Code fügt [Bootstrap](http://getbootstrap.com/) zur einfachen Formatierung hinzu, und [Font Awesome](https://fontawesome.com/) für einige einfache Symbole.</span><span class="sxs-lookup"><span data-stu-id="90ad7-135">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="90ad7-136">Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.</span><span class="sxs-lookup"><span data-stu-id="90ad7-136">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="90ad7-137">Öffnen **Sie ./public/stylesheets/style.css,** und ersetzen Sie den gesamten Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90ad7-137">Open **./public/stylesheets/style.css** and replace its entire contents with the following.</span></span>

    :::code language="css" source="../demo/graph-tutorial/public/stylesheets/style.css":::

1. <span data-ttu-id="90ad7-138">Öffnen **Sie ./views/index.rexs,** und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90ad7-138">Open **./views/index.hbs** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/views/index.hbs" id="IndexSnippet":::

1. <span data-ttu-id="90ad7-139">Öffnen **Sie ./routes/index.js,** und ersetzen Sie den vorhandenen Code durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="90ad7-139">Open **./routes/index.js** and replace the existing code with the following.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/index.js" id="IndexRouterSnippet" highlight="6-10":::

1. <span data-ttu-id="90ad7-140">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="90ad7-140">Save all of your changes and restart the server.</span></span> <span data-ttu-id="90ad7-141">Jetzt sollte die App ganz anders aussehen.</span><span class="sxs-lookup"><span data-stu-id="90ad7-141">Now, the app should look very different.</span></span>

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
