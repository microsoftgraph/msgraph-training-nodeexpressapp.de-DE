<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="6691c-101">In dieser Übung verwenden Sie [Express](http://expressjs.com/) , um eine Webanwendung zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="6691c-101">In this exercise you will use [Express](http://expressjs.com/) to build a web app.</span></span>

1. <span data-ttu-id="6691c-102">Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie über Berechtigungen zum Erstellen von Dateien verfügen, und führen Sie den folgenden Befehl aus, um eine neue Express-App zu erstellen, die als Rendering-Engine [Lenker](http://handlebarsjs.com/) verwendet.</span><span class="sxs-lookup"><span data-stu-id="6691c-102">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Express app that uses [Handlebars](http://handlebarsjs.com/) as the rendering engine.</span></span>

    ```Shell
    npx express-generator@4.16.1 --hbs graph-tutorial
    ```

    <span data-ttu-id="6691c-103">Der Express-Generator erstellt ein neues Verzeichnis `graph-tutorial` mit dem Namen und ein Gerüst für eine Express-App.</span><span class="sxs-lookup"><span data-stu-id="6691c-103">The Express generator creates a new directory called `graph-tutorial` and scaffolds an Express app.</span></span>

1. <span data-ttu-id="6691c-104">Navigieren Sie zum `graph-tutorial` Verzeichnis, und geben Sie den folgenden Befehl ein, um Abhängigkeiten zu installieren.</span><span class="sxs-lookup"><span data-stu-id="6691c-104">Navigate to the `graph-tutorial` directory and enter the following command to install dependencies.</span></span>

    ```Shell
    npm install
    ```

1. <span data-ttu-id="6691c-105">Führen Sie den folgenden Befehl aus, um Knoten Pakete mit gemeldeten Sicherheitsanfälligkeiten zu aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="6691c-105">Run the following command to update Node packages with reported vulnerabilities.</span></span>

    ```Shell
    npm audit fix
    ```

1. <span data-ttu-id="6691c-106">Verwenden Sie den folgenden Befehl, um einen lokalen Webserver zu starten.</span><span class="sxs-lookup"><span data-stu-id="6691c-106">Use the following command to start a local web server.</span></span>

    ```Shell
    npm start
    ```

1. <span data-ttu-id="6691c-107">Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="6691c-107">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="6691c-108">Wenn alles funktioniert, wird die Nachricht "Willkommen bei Express" angezeigt.</span><span class="sxs-lookup"><span data-stu-id="6691c-108">If everything is working, you will see a "Welcome to Express" message.</span></span> <span data-ttu-id="6691c-109">Wenn diese Meldung nicht angezeigt wird, überprüfen Sie das [Express-Handbuch Erste Schritte](http://expressjs.com/starter/generator.html).</span><span class="sxs-lookup"><span data-stu-id="6691c-109">If you don't see that message, check the [Express getting started guide](http://expressjs.com/starter/generator.html).</span></span>

## <a name="install-node-packages"></a><span data-ttu-id="6691c-110">Installieren von Knoten Paketen</span><span class="sxs-lookup"><span data-stu-id="6691c-110">Install Node packages</span></span>

<span data-ttu-id="6691c-111">Bevor Sie fortfahren, installieren Sie einige zusätzliche Pakete, die Sie später verwenden werden:</span><span class="sxs-lookup"><span data-stu-id="6691c-111">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="6691c-112">[dotenv](https://github.com/motdotla/dotenv) zum Laden von Werten aus einer env-Datei.</span><span class="sxs-lookup"><span data-stu-id="6691c-112">[dotenv](https://github.com/motdotla/dotenv) for loading values from a .env file.</span></span>
- <span data-ttu-id="6691c-113">[Zeitpunkt für die](https://github.com/moment/moment/) Formatierung von Datum/Uhrzeit-Werten.</span><span class="sxs-lookup"><span data-stu-id="6691c-113">[moment](https://github.com/moment/moment/) for formatting date/time values.</span></span>
- <span data-ttu-id="6691c-114">[Connect-Flash](https://github.com/jaredhanson/connect-flash) zu Flash-Fehlermeldungen in der app.</span><span class="sxs-lookup"><span data-stu-id="6691c-114">[connect-flash](https://github.com/jaredhanson/connect-flash) to flash error messages in the app.</span></span>
- <span data-ttu-id="6691c-115">[Express-Sitzung](https://github.com/expressjs/session) zum Speichern von Werten in einer serverseitigen Sitzung im Arbeitsspeicher.</span><span class="sxs-lookup"><span data-stu-id="6691c-115">[express-session](https://github.com/expressjs/session) to store values in an in-memory server-side session.</span></span>
- <span data-ttu-id="6691c-116">[Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) zur Authentifizierung und zum Abrufen von Zugriffstoken.</span><span class="sxs-lookup"><span data-stu-id="6691c-116">[passport-azure-ad](https://github.com/AzureAD/passport-azure-ad) for authenticating and getting access tokens.</span></span>
- <span data-ttu-id="6691c-117">[Simple-oauth2](https://github.com/lelylan/simple-oauth2) für die Tokenverwaltung.</span><span class="sxs-lookup"><span data-stu-id="6691c-117">[simple-oauth2](https://github.com/lelylan/simple-oauth2) for token management.</span></span>
- <span data-ttu-id="6691c-118">[Microsoft-Graph-Client](https://github.com/microsoftgraph/msgraph-sdk-javascript) zum tätigen von Anrufen an Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="6691c-118">[microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="6691c-119">[isomorph-FETCH](https://github.com/matthew-andrews/isomorphic-fetch) to Polyfill der FETCH for-Knoten.</span><span class="sxs-lookup"><span data-stu-id="6691c-119">[isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch) to polyfill the fetch for Node.</span></span> <span data-ttu-id="6691c-120">Für die `microsoft-graph-client` Bibliothek ist eine Abruf Polyfüllung erforderlich.</span><span class="sxs-lookup"><span data-stu-id="6691c-120">A fetch polyfill is required for the `microsoft-graph-client` library.</span></span> <span data-ttu-id="6691c-121">Weitere Informationen finden Sie im [Microsoft Graph-JavaScript-Clientbibliothek-wiki](https://github.com/microsoftgraph/msgraph-sdk-javascript/wiki/Migration-from-1.x.x-to-2.x.x#polyfill-only-when-required) .</span><span class="sxs-lookup"><span data-stu-id="6691c-121">See the [Microsoft Graph JavaScript client library wiki](https://github.com/microsoftgraph/msgraph-sdk-javascript/wiki/Migration-from-1.x.x-to-2.x.x#polyfill-only-when-required) for more information.</span></span>

1. <span data-ttu-id="6691c-122">Führen Sie den folgenden Befehl in der CLI aus.</span><span class="sxs-lookup"><span data-stu-id="6691c-122">Run the following command in your CLI.</span></span>

    ```Shell
    npm install dotenv@8.2.0 moment@2.24.0 connect-flash@0.1.1 express-session@1.17.0 isomorphic-fetch@2.2.1
    npm install passport-azure-ad@4.2.1 simple-oauth2@3.3.0 @microsoft/microsoft-graph-client@2.0.0
    ```

    > [!TIP]
    > <span data-ttu-id="6691c-123">Windows-Benutzer erhalten möglicherweise die folgende Fehlermeldung, wenn Sie versuchen, diese Pakete unter Windows zu installieren.</span><span class="sxs-lookup"><span data-stu-id="6691c-123">Windows users may get the following error message when trying to install these packages on Windows.</span></span>
    >
    > ```Shell
    > gyp ERR! stack Error: Can't find Python executable "python", you can set the PYTHON env variable.
    > ```
    >
    > <span data-ttu-id="6691c-124">Um den Fehler zu beheben, führen Sie den folgenden Befehl aus, um die Windows-Erstellungstools mithilfe eines erhöhten Terminalfensters (Administrator) zu installieren, das die vs-Erstellungstools und Python installiert.</span><span class="sxs-lookup"><span data-stu-id="6691c-124">To resolve the error, run the following command to install the Windows Build Tools using an elevated (Administrator) terminal window which installs the VS Build Tools and Python.</span></span>
    >
    > ```Shell
    > npm install --global --production windows-build-tools
    > ```

1. <span data-ttu-id="6691c-125">Aktualisieren Sie die Anwendung, um `connect-flash` die `express-session` and-Middleware zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="6691c-125">Update the application to use the `connect-flash` and `express-session` middleware.</span></span> <span data-ttu-id="6691c-126">Öffnen Sie `./app.js` die Datei, und fügen `require` Sie die folgende Anweisung am Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="6691c-126">Open the `./app.js` file and add the following `require` statement to the top of the file.</span></span>

    ```javascript
    var session = require('express-session');
    var flash = require('connect-flash');
    ```

1. <span data-ttu-id="6691c-127">Fügen Sie den folgenden Code unmittelbar nach `var app = express();` der Codezeile hinzu.</span><span class="sxs-lookup"><span data-stu-id="6691c-127">Add the following code immediately after the `var app = express();` line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="SessionSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="6691c-128">Entwerfen der App</span><span class="sxs-lookup"><span data-stu-id="6691c-128">Design the app</span></span>

<span data-ttu-id="6691c-129">In diesem Abschnitt werden Sie die Benutzeroberfläche für die APP implementieren.</span><span class="sxs-lookup"><span data-stu-id="6691c-129">In this section you will implement the UI for the app.</span></span>

1. <span data-ttu-id="6691c-130">Öffnen Sie `./views/layout.hbs` die Datei, und ersetzen Sie den gesamten Inhalt durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="6691c-130">Open the `./views/layout.hbs` file and replace the entire contents with the following code.</span></span>

    :::code language="html" source="../demo/graph-tutorial/views/layout.hbs" id="LayoutSnippet":::

    <span data-ttu-id="6691c-131">Dieser Code fügt [Bootstrap](http://getbootstrap.com/) zur einfachen Formatierung hinzu, und [Font Awesome](https://fontawesome.com/) für einige einfache Symbole.</span><span class="sxs-lookup"><span data-stu-id="6691c-131">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="6691c-132">Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.</span><span class="sxs-lookup"><span data-stu-id="6691c-132">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="6691c-133">Öffnen `./public/stylesheets/style.css` Sie den gesamten Inhalt, und ersetzen Sie ihn durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="6691c-133">Open `./public/stylesheets/style.css` and replace its entire contents with the following.</span></span>

    :::code language="css" source="../demo/graph-tutorial/public/stylesheets/style.css":::

1. <span data-ttu-id="6691c-134">Öffnen Sie `./views/index.hbs` die Datei, und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="6691c-134">Open the `./views/index.hbs` file and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/views/index.hbs" id="IndexSnippet":::

1. <span data-ttu-id="6691c-135">Öffnen Sie `./routes/index.js` die Datei, und ersetzen Sie den vorhandenen Code durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="6691c-135">Open the `./routes/index.js` file and replace the existing code with the following.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/index.js" id="IndexRouterSnippet" highlight="6-10":::

1. <span data-ttu-id="6691c-136">Speichern Sie alle Änderungen, und starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="6691c-136">Save all of your changes and restart the server.</span></span> <span data-ttu-id="6691c-137">Nun sollte die APP sehr unterschiedlich aussehen.</span><span class="sxs-lookup"><span data-stu-id="6691c-137">Now, the app should look very different.</span></span>

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
