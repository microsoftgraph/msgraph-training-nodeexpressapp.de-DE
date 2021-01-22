<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung verwenden Sie [Express zum](http://expressjs.com/) Erstellen einer Web-App.

1. Öffnen Sie Ihre CLI, navigieren Sie zu einem Verzeichnis, in dem Sie über die Rechte zum Erstellen von Dateien verfügen, und führen Sie den folgenden Befehl aus, um eine neue Express-App zu erstellen, die [Handlebars](http://handlebarsjs.com/) als Renderingmodul verwendet.

    ```Shell
    npx express-generator@4.16.1 --hbs graph-tutorial
    ```

    Der Express-Generator erstellt ein neues Verzeichnis namens `graph-tutorial` und erstellt ein Gerüst für eine Express-App.

1. Navigieren Sie zum `graph-tutorial` Verzeichnis, und geben Sie den folgenden Befehl ein, um Abhängigkeiten zu installieren.

    ```Shell
    npm install
    ```

1. Führen Sie den folgenden Befehl aus, um Knotenpakete mit gemeldeten Sicherheitsrisiken zu aktualisieren.

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

1. Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`. Wenn alles funktioniert, wird die Meldung "Willkommen bei Express" angezeigt. Wenn diese Meldung nicht angezeigt wird, lesen Sie die Anleitung für [die ersten Schritte von Express.](http://expressjs.com/starter/generator.html)

## <a name="install-node-packages"></a>Installieren von Knotenpaketen

Installieren Sie vor dem Wechsel einige zusätzliche Pakete, die Sie später verwenden werden:

- [dotenv](https://github.com/motdotla/dotenv) zum Laden von Werten aus einer ENV-Datei.
- [Moment](https://github.com/moment/moment/) für die Formatierung von Datums-/Uhrzeitwerten.
- [windows-iana](https://github.com/rubenillodo/windows-iana) für die Übersetzung von Windows-Zeitzonennamen in IANA-Zeitzonen-IDs.
- [connect-flash](https://github.com/jaredhanson/connect-flash) to flash error messages in the app.
- [express-session](https://github.com/expressjs/session) zum Speichern von Werten in einer serverseitigen Speichersitzung.
- [express-promise-router](https://github.com/express-promise-router/express-promise-router) to allow route handlers to return a Promise.
- [Express validator](https://github.com/express-validator/express-validator) für die Analyse und Validierung von Formulardaten.
- [msal-node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) zum Authentifizieren und Abrufen von Zugriffstoken.
- [uuid,](https://github.com/uuidjs/uuid) die von "msal-node" zum Generieren von GUIDs verwendet wird.
- [microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) for making calls to Microsoft Graph.
- [isomorph-fetch zum](https://github.com/matthew-andrews/isomorphic-fetch) Polyfilling des Abrufs für Node. Für die Bibliothek ist ein Polyfill zum Abrufen `microsoft-graph-client` erforderlich. Weitere Informationen finden Sie im [Microsoft Graph-JavaScript-Clientbibliothekswiki.](https://github.com/microsoftgraph/msgraph-sdk-javascript/wiki/Migration-from-1.x.x-to-2.x.x#polyfill-only-when-required)

1. Führen Sie den folgenden Befehl in Der CLI aus.

    ```Shell
    npm install dotenv@8.2.0 moment@2.29.1 moment-timezone@0.5.31 connect-flash@0.1.1 express-session@1.17.1 express-promise-router@4.0.1 isomorphic-fetch@3.0.0
    npm install @azure/msal-node@1.0.0-beta.0 @microsoft/microsoft-graph-client@2.1.1 windows-iana@4.2.1 express-validator@6.6.1 uuid@8.3.1
    ```

    > [!TIP]
    > Beim Versuch, diese Pakete unter Windows zu installieren, wird möglicherweise die folgende Fehlermeldung angezeigt.
    >
    > ```Shell
    > gyp ERR! stack Error: Can't find Python executable "python", you can set the PYTHON env variable.
    > ```
    >
    > Führen Sie zum Beheben des Fehlers den folgenden Befehl aus, um die Windows Build Tools mithilfe eines Terminalfensters mit erhöhten Rechten (Administrator) zu installieren, in dem die VS Build Tools und Python installiert werden.
    >
    > ```Shell
    > npm install --global --production windows-build-tools
    > ```

1. Aktualisieren Sie die Anwendung so, dass sie die `connect-flash` `express-session` Middleware verwendet. Öffnen **Sie ./app.js,** und fügen Sie die folgende Anweisung `require` am Oberen Rand der Datei hinzu.

    ```javascript
    const session = require('express-session');
    const flash = require('connect-flash');
    const msal = require('@azure/msal-node');
    ```

1. Fügen Sie den folgenden Code unmittelbar hinter der Zeile `var app = express();` hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="SessionSnippet":::

## <a name="design-the-app"></a>Entwerfen der App

In diesem Abschnitt implementieren Sie die Benutzeroberfläche für die App.

1. Öffnen **Sie ./views/layout.layoutss,** und ersetzen Sie den gesamten Inhalt durch den folgenden Code.

    :::code language="html" source="../demo/graph-tutorial/views/layout.hbs" id="LayoutSnippet":::

    Dieser Code fügt [Bootstrap](http://getbootstrap.com/) zur einfachen Formatierung hinzu, und [Font Awesome](https://fontawesome.com/) für einige einfache Symbole. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

1. Öffnen **Sie ./public/stylesheets/style.css,** und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="css" source="../demo/graph-tutorial/public/stylesheets/style.css":::

1. Öffnen **Sie ./views/index.rexs,** und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="html" source="../demo/graph-tutorial/views/index.hbs" id="IndexSnippet":::

1. Öffnen **Sie ./routes/index.js,** und ersetzen Sie den vorhandenen Code durch Folgendes.

    :::code language="javascript" source="../demo/graph-tutorial/routes/index.js" id="IndexRouterSnippet" highlight="6-10":::

1. Speichern Sie alle Änderungen, und starten Sie den Server neu. Jetzt sollte die App ganz anders aussehen.

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
