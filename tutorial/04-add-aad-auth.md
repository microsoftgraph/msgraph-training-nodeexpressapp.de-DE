<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten. In diesem Schritt werden Sie die [msal-Node-](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) Bibliothek in die Anwendung integrieren.

1. Erstellen Sie eine neue Datei mit dem Namen " **. env** " im Stammverzeichnis Ihrer Anwendung, und fügen Sie den folgenden Code hinzu.

    :::code language="ini" source="../demo/graph-tutorial/example.env":::

    Ersetzen `YOUR_CLIENT_SECRET_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, und ersetzen `YOUR_APP_SECRET_HERE` Sie durch den von Ihnen generierten Client Schlüssel.

    > [!IMPORTANT]
    > Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die **. env** -Datei aus der Quellcodeverwaltung auszuschließen, damit versehentlich Ihre APP-ID und Ihr Kennwort verloren gehen.

1. Öffnen Sie **./app.js** , und fügen Sie die folgende Codezeile zum Anfang der Datei hinzu, um die **env** -Datei zu laden.

    ```javascript
    require('dotenv').config();
    ```

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

1. Suchen Sie die-Position `var indexRouter = require('./routes/index');` in **./#b0**. Fügen Sie den folgenden Code **vor** dieser Codezeile ein.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="MsalInitSnippet":::

    Mit diesem Code wird die msal-Knoten Bibliothek mit der APP-ID und dem Kennwort für die APP initialisiert.

1. Erstellen Sie eine neue Datei im **./routes** -Verzeichnis mit dem Namen **auth.js** , und fügen Sie den folgenden Code hinzu.

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

    Dadurch wird ein Router mit drei Routen definiert: `signin` , `callback` und `signout` .

    Die `signin` Route Ruft die `getAuthCodeUrl` Funktion auf, um die Anmelde-URL zu generieren, und leitet dann den Browser an diese URL um.

    Auf der `callback` Route wird nach Abschluss der Anmeldung Azure umgeleitet. Der Code Ruft die `acquireTokenByCode` Funktion auf, um den Autorisierungscode für ein Zugriffstoken auszutauschen. Sobald das Token abgerufen wurde, wird es an die Startseite mit dem Zugriffstoken im temporären Fehlerwert zurückgeleitet. Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren. Bevor wir testen, müssen wir die Express-App so konfigurieren, dass Sie den neuen Router aus **./routes/auth.js** verwendet.

    Die `signout` -Methode protokolliert den Benutzer und zerstört die Sitzung.

1. Öffnen Sie **./app.js** , und fügen Sie den folgenden Code **vor** die- `var app = express();` Codezeile ein.

    ```javascript
    var authRouter = require('./routes/auth');
    ```

1. Fügen Sie den folgenden Code **nach** der `app.use('/', indexRouter);` Codezeile ein.

    ```javascript
    app.use('/auth', authRouter);
    ```

Starten Sie den Server, und navigieren Sie zu `https://localhost:3000` . Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser leitet zur App um, in der Sie das Token sehen.

### <a name="get-user-details"></a>Benutzerdetails abrufen

1. Erstellen Sie eine neue Datei im Stamm des Projekts mit dem Namen **graph.js** , und fügen Sie den folgenden Code hinzu.

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

    Dadurch wird die `getUserDetails` Funktion exportiert, die das Microsoft Graph-SDK verwendet, um den Endpunkt aufzurufen `/me` und das Ergebnis zurückzugeben.

1. Öffnen Sie **./routes/auth.js** , und fügen Sie die folgenden `require` Anweisungen am Anfang der Datei hinzu.

    ```javascript
    var graph = require('../graph');
    ```

1. Ersetzen Sie die vorhandene Rückruf Route durch den folgenden Code.

    :::code language="javascript" source="../demo/graph-tutorial/routes/auth.js" id="CallbackSnippet" highlight="13-23":::

    Der neue Code speichert die Konto-ID des Benutzers in der Sitzung, ruft die Details des Benutzers aus Microsoft Graph ab und speichert ihn im Benutzerspeicher der app.

1. Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort. Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen. Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a>Speichern und Aktualisieren von Token

Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach seiner Ausgabe ab. An dieser Stelle kommt das Aktualisierungstoken ins Spiel. Mit der OAuth-Spezifikation wird ein Aktualisierungstoken eingeführt, mit dem die APP ein neues Zugriffstoken anfordern kann, ohne dass sich der Benutzer erneut anmelden muss.

Da die APP das msal-Node-Paket verwendet, müssen Sie keine Token-Speicher-oder Aktualisierungslogik implementieren. Die APP verwendet den standardmäßigen msal-Token-Cache im Arbeitsspeicher, der für eine Beispielanwendung ausreicht. Produktionsanwendungen sollten ein eigenes [Cache-Plugin](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md) bereitstellen, um den Token-Cache in einem sicheren, zuverlässigen Speichermedium zu serialisieren.
