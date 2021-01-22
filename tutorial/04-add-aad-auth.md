<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph abzurufen. In diesem Schritt integrieren Sie die [msal-node-Bibliothek](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) in die Anwendung.

1. Erstellen Sie eine neue Datei mit dem Namen **".env"** im Stammverzeichnis Ihrer Anwendung, und fügen Sie den folgenden Code hinzu.

    :::code language="ini" source="../demo/graph-tutorial/example.env":::

    Ersetzen `YOUR_CLIENT_SECRET_HERE` Sie sie durch die Anwendungs-ID aus dem Anwendungsregistrierungsportal, und ersetzen Sie sie durch den von Ihnen `YOUR_APP_SECRET_HERE` generierten geheimen Client-Schlüssel.

    > [!IMPORTANT]
    > Wenn Sie quellcodeverwaltung wie Git verwenden, wäre es jetzt ein guter Zeitpunkt, die **env-Datei** aus der Quellcodeverwaltung auszuschließen, um zu vermeiden, dass Versehentliches Lecks für Ihre App-ID und Ihr Kennwort erforderlich sind.

1. Öffnen **Sie ./app.js,** und fügen Sie die folgende Zeile am oberen Rand der Datei hinzu, um die **.env-Datei zu** laden.

    ```javascript
    require('dotenv').config();
    ```

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

1. Suchen Sie die `var app = express();` Zeile in **./app.js**. Fügen Sie nach dieser **Zeile den folgenden** Code ein.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="MsalInitSnippet":::

    Dieser Code initialisiert die msal-node-Bibliothek mit der App-ID und dem Kennwort für die App.

1. Erstellen Sie eine neue Datei im Verzeichnis **./routes** mit **auth.js,** und fügen Sie den folgenden Code hinzu.

    ```javascript
    var router = require('express-promise-router')();

    /* GET auth callback. */
    router.get('/signin',
      async function (req, res) {
        const urlParameters = {
          scopes: process.env.OAUTH_SCOPES.split(','),
          redirectUri: process.env.OAUTH_REDIRECT_URI
        };

        try {
          const authUrl = await req.app.locals
            .msalClient.getAuthCodeUrl(urlParameters);
          res.redirect(authUrl);
        }
        catch (error) {
          console.log(`Error: ${error}`);
          req.flash('error_msg', {
            message: 'Error getting auth URL',
            debug: JSON.stringify(error, Object.getOwnPropertyNames(error))
          });
          res.redirect('/');
        }
      }
    );

    router.get('/callback',
      async function(req, res) {
        const tokenRequest = {
          code: req.query.code,
          scopes: process.env.OAUTH_SCOPES.split(','),
          redirectUri: process.env.OAUTH_REDIRECT_URI
        };

        try {
          const response = await req.app.locals
            .msalClient.acquireTokenByCode(tokenRequest);

          // TEMPORARY!
          // Flash the access token for testing purposes
          req.flash('error_msg', {
            message: 'Access token',
            debug: response.accessToken
          });
        } catch (error) {
          req.flash('error_msg', {
            message: 'Error completing authentication',
            debug: JSON.stringify(error, Object.getOwnPropertyNames(error))
          });
        }

        res.redirect('/');
      }
    );

    router.get('/signout',
      async function(req, res) {
        // Sign out
        if (req.session.userId) {
          // Look up the user's account in the cache
          const accounts = await req.app.locals.msalClient
            .getTokenCache()
            .getAllAccounts();

          const userAccount = accounts.find(a => a.homeAccountId === req.session.userId);

          // Remove the account
          if (userAccount) {
            req.app.locals.msalClient
              .getTokenCache()
              .removeAccount(userAccount);
          }
        }

        // Destroy the user's session
        req.session.destroy(function (err) {
          res.redirect('/');
        });
      }
    );

    module.exports = router;
    ```

    Dadurch wird ein Router mit drei Routen definiert: `signin` `callback` , und `signout` .

    Die `signin` Route ruft die Funktion `getAuthCodeUrl` auf, um die Anmelde-URL zu generieren, und leitet dann den Browser an diese URL um.

    Die `callback` Route leitet Azure um, nachdem die Anmeldung abgeschlossen ist. Der Code ruft die `acquireTokenByCode` Funktion auf, um den Autorisierungscode gegen ein Zugriffstoken zu tauschen. Nachdem das Token erhalten wurde, wird es mit dem Zugriffstoken im temporären Fehlerwert zurück zur Startseite umgeleitet. Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktioniert, bevor wir weiter arbeiten. Vor dem Testen müssen wir die Express-App für die Verwendung des neuen Routers aus **./routes/auth.js**.

    Die `signout` Methode meldet den Benutzer ab und zerstört die Sitzung.

1. Öffnen **Sie ./app.js,** und fügen Sie den folgenden Code **vor der** Zeile `var app = express();` ein.

    ```javascript
    var authRouter = require('./routes/auth');
    ```

1. Fügen Sie den folgenden Code **nach der** `app.use('/', indexRouter);` Zeile ein.

    ```javascript
    app.use('/auth', authRouter);
    ```

Starten Sie den Server, und navigieren Sie zu `https://localhost:3000` . Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser leitet zur App um, in der Sie das Token sehen.

### <a name="get-user-details"></a>Benutzerdetails abrufen

1. Erstellen Sie eine neue Datei im Stammverzeichnis des Projekts namens **graph.js,** und fügen Sie den folgenden Code hinzu.

    ```javascript
    var graph = require('@microsoft/microsoft-graph-client');
    require('isomorphic-fetch');

    module.exports = {
      getUserDetails: async function(accessToken) {
        const client = getAuthenticatedClient(accessToken);

        const user = await client
          .api('/me')
          .select('displayName,mail,mailboxSettings,userPrincipalName')
          .get();
        return user;
      },
    };

    function getAuthenticatedClient(accessToken) {
      // Initialize Graph client
      const client = graph.Client.init({
        // Use the provided access token to authenticate
        // requests
        authProvider: (done) => {
          done(null, accessToken);
        }
      });

      return client;
    }
    ```

    Dadurch wird die Funktion exportiert, die das Microsoft Graph SDK zum Aufrufen des Endpunkts und `getUserDetails` `/me` zum Zurückgeben des Ergebnisses verwendet.

1. Öffnen **Sie ./routes/auth.js,** und fügen Sie die folgenden Anweisungen am Oberen Rand `require` der Datei hinzu.

    ```javascript
    var graph = require('../graph');
    ```

1. Ersetzen Sie die vorhandene Rückrufroute durch den folgenden Code.

    :::code language="javascript" source="../demo/graph-tutorial/routes/auth.js" id="CallbackSnippet" highlight="13-23":::

    Der neue Code speichert die Konto-ID des Benutzers in der Sitzung, ruft die Details des Benutzers aus Microsoft Graph ab und speichert sie im Benutzerspeicher der App.

1. Starten Sie den Server neu, und gehen Sie den Anmeldevorgang durch. Sie sollten wieder auf der Startseite landen, aber die Benutzeroberfläche sollte geändert werden, um anzugeben, dass Sie angemeldet sind.

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den Link **"Abmelden" zu** klicken. Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a>Speichern und Aktualisieren von Token

An diesem Punkt verfügt Ihre Anwendung über ein Zugriffstoken, das im `Authorization` Header von API-Aufrufen gesendet wird. Dies ist das Token, mit dem die App im Namen des Benutzers auf Microsoft Graph zugreifen kann.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach seiner Entsprechung ab. An dieser Stelle kommt das Aktualisierungstoken ins Spiel. Die OAuth-Spezifikation führt ein Aktualisierungstoken ein, mit dem die App ein neues Zugriffstoken anfordern kann, ohne dass sich der Benutzer erneut anmelden muss.

Da die App das msal-node-Paket verwendet, müssen Sie keine Tokenspeicher- oder Aktualisierungslogik implementieren. Die App verwendet den standardmäßigen Speichertokencache "msal-node", der für eine Beispielanwendung ausreicht. Produktionsanwendungen sollten ihr eigenes Plug-In [zum](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md) Zwischenspeichern bereitstellen, um den Tokencache auf einem sicheren, zuverlässigen Speichermedium zu serialisieren.
