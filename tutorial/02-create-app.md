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

1. Verwenden Sie den folgenden Befehl, um einen lokalen Webserver zu starten.

    ```Shell
    npm start
    ```

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`. Wenn alles funktioniert, wird die Nachricht "Willkommen bei Express" angezeigt. Wenn diese Meldung nicht angezeigt wird, überprüfen Sie das [Express-Handbuch Erste Schritte](http://expressjs.com/starter/generator.html).

## <a name="install-node-packages"></a>Installieren von Knoten Paketen

Bevor Sie fortfahren, installieren Sie einige zusätzliche Pakete, die Sie später verwenden werden:

- [dotenv](https://github.com/motdotla/dotenv) zum Laden von Werten aus einer env-Datei.
- [Zeitpunkt für die](https://github.com/moment/moment/) Formatierung von Datum/Uhrzeit-Werten.
- [Connect-Flash](https://github.com/jaredhanson/connect-flash) zu Flash-Fehlermeldungen in der app.
- [Express-Sitzung](https://github.com/expressjs/session) zum Speichern von Werten in einer serverseitigen Sitzung im Arbeitsspeicher.
- [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) zur Authentifizierung und zum Abrufen von Zugriffstoken.
- [Simple-oauth2](https://github.com/lelylan/simple-oauth2) für die Tokenverwaltung.
- [Microsoft-Graph-Client](https://github.com/microsoftgraph/msgraph-sdk-javascript) zum tätigen von Anrufen an Microsoft Graph.
- [isomorph-FETCH](https://github.com/matthew-andrews/isomorphic-fetch) to Polyfill der FETCH for-Knoten. Für die `microsoft-graph-client` Bibliothek ist eine Abruf Polyfüllung erforderlich. Weitere Informationen finden Sie im [Microsoft Graph-JavaScript-Clientbibliothek-wiki](https://github.com/microsoftgraph/msgraph-sdk-javascript/wiki/Migration-from-1.x.x-to-2.x.x#polyfill-only-when-required) .

1. Führen Sie den folgenden Befehl in der CLI aus.

    ```Shell
    npm install dotenv@8.2.0 moment@2.24.0 connect-flash@0.1.1 express-session@1.17.0 isomorphic-fetch@2.2.1
    npm install passport-azure-ad@4.2.1 simple-oauth2@3.3.0 @microsoft/microsoft-graph-client@2.0.0
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

1. Aktualisieren Sie die Anwendung, um `connect-flash` die `express-session` and-Middleware zu verwenden. Öffnen Sie `./app.js` die Datei, und fügen `require` Sie die folgende Anweisung am Anfang der Datei hinzu.

    ```javascript
    var session = require('express-session');
    var flash = require('connect-flash');
    ```

1. Fügen Sie den folgenden Code unmittelbar nach `var app = express();` der Codezeile hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="SessionSnippet":::

## <a name="design-the-app"></a>Entwerfen der App

In diesem Abschnitt werden Sie die Benutzeroberfläche für die APP implementieren.

1. Öffnen Sie `./views/layout.hbs` die Datei, und ersetzen Sie den gesamten Inhalt durch den folgenden Code.

    :::code language="html" source="../demo/graph-tutorial/views/layout.hbs" id="LayoutSnippet":::

    Dieser Code fügt [Bootstrap](http://getbootstrap.com/) zur einfachen Formatierung hinzu, und [Font Awesome](https://fontawesome.com/) für einige einfache Symbole. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

1. Öffnen `./public/stylesheets/style.css` Sie den gesamten Inhalt, und ersetzen Sie ihn durch Folgendes.

    :::code language="css" source="../demo/graph-tutorial/public/stylesheets/style.css":::

1. Öffnen Sie `./views/index.hbs` die Datei, und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/views/index.hbs" id="IndexSnippet":::

1. Öffnen Sie `./routes/index.js` die Datei, und ersetzen Sie den vorhandenen Code durch Folgendes.

    :::code language="javascript" source="../demo/graph-tutorial/routes/index.js" id="IndexRouterSnippet" highlight="6-10":::

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Nun sollte die APP sehr unterschiedlich aussehen.

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
