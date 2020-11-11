<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung verwenden Sie [Express](http://expressjs.com/) , um eine Webanwendung zu erstellen.

1. Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie über Berechtigungen zum Erstellen von Dateien verfügen, und führen Sie den folgenden Befehl aus, um eine neue Express-App zu erstellen, die als Rendering-Engine [Lenker](http://handlebarsjs.com/) verwendet.

    ```Shell
    npx express-generator@4.16.1 --hbs graph-tutorial
    ```

    Der Express-Generator erstellt ein neues Verzeichnis `graph-tutorial` mit dem Namen und ein Gerüst für eine Express-App.

1. Navigieren Sie zum `graph-tutorial` Verzeichnis, und geben Sie den folgenden Befehl ein, um Abhängigkeiten zu installieren.

    ```Shell
    npm install
    ```

1. Führen Sie den folgenden Befehl aus, um Knoten Pakete mit gemeldeten Sicherheitsanfälligkeiten zu aktualisieren.

    ```Shell
    npm audit fix
    ```

1. Führen Sie den folgenden Befehl aus, um die Version von Express und andere Abhängigkeiten zu aktualisieren.

    ```Shell
    npm install express@4.17.1 http-errors@1.8.0 morgan@1.10.0 debug@4.1.1
    ```

1. Verwenden Sie den folgenden Befehl, um einen lokalen Webserver zu starten.

    ```Shell
    npm start
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`. Wenn alles funktioniert, wird die Nachricht "Willkommen bei Express" angezeigt. Wenn diese Meldung nicht angezeigt wird, überprüfen Sie das [Express-Handbuch Erste Schritte](http://expressjs.com/starter/generator.html).

## <a name="install-node-packages"></a>Installieren von Knoten Paketen

Bevor Sie fortfahren, installieren Sie einige zusätzliche Pakete, die Sie später verwenden werden:

- [dotenv](https://github.com/motdotla/dotenv) zum Laden von Werten aus einer env-Datei.
- [Zeitpunkt für die](https://github.com/moment/moment/) Formatierung von Datum/Uhrzeit-Werten.
- [Windows-IANA](https://github.com/rubenillodo/windows-iana) für die Übersetzung von Windows-Zeitzonennamen in die IANA-Zeit Zonen-IDs.
- [Connect-Flash](https://github.com/jaredhanson/connect-flash) zu Flash-Fehlermeldungen in der app.
- [Express-Sitzung](https://github.com/expressjs/session) zum Speichern von Werten in einer serverseitigen Sitzung im Arbeitsspeicher.
- [Express-Promise-Router](https://github.com/express-promise-router/express-promise-router) , damit Routenhandler eine Zusage zurückgeben können.
- [Express-Validator](https://github.com/express-validator/express-validator) zum Analysieren und Validieren von Formulardaten.
- [msal-Knoten](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) zur Authentifizierung und zum Abrufen von Zugriffstoken.
- [UUID](https://github.com/uuidjs/uuid) , die von msal-Node zum Generieren von GUIDs verwendet wird.
- [Microsoft-Graph-Client](https://github.com/microsoftgraph/msgraph-sdk-javascript) zum tätigen von Anrufen an Microsoft Graph.
- [isomorph-FETCH](https://github.com/matthew-andrews/isomorphic-fetch) to Polyfill der FETCH for-Knoten. Für die Bibliothek ist eine Abruf Polyfüllung erforderlich `microsoft-graph-client` . Weitere Informationen finden Sie im [Microsoft Graph-JavaScript-Clientbibliothek-wiki](https://github.com/microsoftgraph/msgraph-sdk-javascript/wiki/Migration-from-1.x.x-to-2.x.x#polyfill-only-when-required) .

1. Führen Sie den folgenden Befehl in der CLI aus.

    ```Shell
    npm install dotenv@8.2.0 moment@2.29.1 moment-timezone@0.5.31 connect-flash@0.1.1 express-session@1.17.1 isomorphic-fetch@3.0.0
    npm install @azure/msal-node@1.0.0-beta.0 @microsoft/microsoft-graph-client@2.1.1 windows-iana@4.2.1 express-validator@6.6.1 uuid@8.3.1
    ```

    > [!TIP]
    > Windows-Benutzer erhalten möglicherweise die folgende Fehlermeldung, wenn Sie versuchen, diese Pakete unter Windows zu installieren.
    >
    > ```Shell
    > gyp ERR! stack Error: Can't find Python executable "python", you can set the PYTHON env variable.
    > ```
    >
    > Um den Fehler zu beheben, führen Sie den folgenden Befehl aus, um die Windows-Erstellungstools mithilfe eines erhöhten Terminalfensters (Administrator) zu installieren, das die vs-Erstellungstools und Python installiert.
    >
    > ```Shell
    > npm install --global --production windows-build-tools
    > ```

1. Aktualisieren Sie die Anwendung, um die `connect-flash` and-Middleware zu verwenden `express-session` . Öffnen Sie **./app.js** , und fügen Sie die folgende `require` Anweisung am Anfang der Datei hinzu.

    ```javascript
    const session = require('express-session');
    const flash = require('connect-flash');
    const msal = require('@azure/msal-node');
    ```

1. Fügen Sie den folgenden Code unmittelbar nach der `var app = express();` Codezeile hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="SessionSnippet":::

## <a name="design-the-app"></a>Entwerfen der App

In diesem Abschnitt werden Sie die Benutzeroberfläche für die APP implementieren.

1. Öffnen Sie **./views/Layout.HBS** , und ersetzen Sie den gesamten Inhalt durch den folgenden Code.

    :::code language="html" source="../demo/graph-tutorial/views/layout.hbs" id="LayoutSnippet":::

    Dieser Code fügt [Bootstrap](http://getbootstrap.com/) zur einfachen Formatierung hinzu, und [Font Awesome](https://fontawesome.com/) für einige einfache Symbole. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

1. Öffnen Sie **/Public/Stylesheets/Style.CSS** , und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="css" source="../demo/graph-tutorial/public/stylesheets/style.css":::

1. Öffnen Sie **./views/Index.HBS** , und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/views/index.hbs" id="IndexSnippet":::

1. Öffnen Sie **./routes/index.js** und ersetzen Sie den vorhandenen Code durch Folgendes.

    :::code language="javascript" source="../demo/graph-tutorial/routes/index.js" id="IndexRouterSnippet" highlight="6-10":::

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Nun sollte die APP sehr unterschiedlich aussehen.

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
