<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten. In diesem Schritt werden Sie die [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) -Bibliothek in die Anwendung integrieren.

1. Erstellen Sie im Stammverzeichnis `.env` der Anwendung eine neue Datei mit dem Namen file, und fügen Sie den folgenden Code hinzu.

    :::code language="ini" source="../demo/graph-tutorial/.env.example":::

    Ersetzen `YOUR APP ID HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR APP SECRET HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.

    > [!IMPORTANT]
    > Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die Datei `.env` aus der Quellcodeverwaltung auszuschließen, damit versehentlich keine APP-ID und Ihr Kennwort verloren gehen.

1. Öffnen `./app.js` Sie und fügen Sie die folgende Codezeile am Anfang der Datei hinzu, um `.env` die Datei zu laden.

    ```javascript
    require('dotenv').config();
    ```

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

1. Suchen Sie die `var indexRouter = require('./routes/index');` - `./app.js`Position in. Fügen Sie den folgenden Code **vor** dieser Codezeile ein.

    ```javascript
    var passport = require('passport');
    var OIDCStrategy = require('passport-azure-ad').OIDCStrategy;

    // Configure passport

    // In-memory storage of logged-in users
    // For demo purposes only, production apps should store
    // this in a reliable storage
    var users = {};

    // Passport calls serializeUser and deserializeUser to
    // manage users
    passport.serializeUser(function(user, done) {
      // Use the OID property of the user as a key
      users[user.profile.oid] = user;
      done (null, user.profile.oid);
    });

    passport.deserializeUser(function(id, done) {
      done(null, users[id]);
    });

    // Callback function called once the sign-in is complete
    // and an access token has been obtained
    async function signInComplete(iss, sub, profile, accessToken, refreshToken, params, done) {
      if (!profile.oid) {
        return done(new Error("No OID found in user profile."));
      }

      // Save the profile and tokens in user storage
      users[profile.oid] = { profile, accessToken };
      return done(null, users[profile.oid]);
    }

    // Configure OIDC strategy
    passport.use(new OIDCStrategy(
      {
        identityMetadata: `${process.env.OAUTH_AUTHORITY}${process.env.OAUTH_ID_METADATA}`,
        clientID: process.env.OAUTH_APP_ID,
        responseType: 'code id_token',
        responseMode: 'form_post',
        redirectUrl: process.env.OAUTH_REDIRECT_URI,
        allowHttpForRedirectUrl: true,
        clientSecret: process.env.OAUTH_APP_PASSWORD,
        validateIssuer: false,
        passReqToCallback: false,
        scope: process.env.OAUTH_SCOPES.split(' ')
      },
      signInComplete
    ));
    ```

    Dieser Code initialisiert die [Passport. js](http://www.passportjs.org/) -Bibliothek für die `passport-azure-ad` Verwendung der Bibliothek und konfiguriert Sie mit der APP-ID und dem Kennwort für die app.

1. Suchen Sie die `app.use('/', indexRouter);` - `./app.js`Position in. Fügen Sie den folgenden Code **vor** dieser Codezeile ein.

    ```javascript
    // Initialize passport
    app.use(passport.initialize());
    app.use(passport.session());
    ```

1. Erstellen Sie eine neue Datei im `./routes` Verzeichnis mit `auth.js` dem Namen, und fügen Sie den folgenden Code hinzu.

    ```javascript
    var express = require('express');
    var passport = require('passport');
    var router = express.Router();

    /* GET auth callback. */
    router.get('/signin',
      function  (req, res, next) {
        passport.authenticate('azuread-openidconnect',
          {
            response: res,
            prompt: 'login',
            failureRedirect: '/',
            failureFlash: true,
            successRedirect: '/'
          }
        )(req,res,next);
      }
    );

    router.post('/callback',
      function(req, res, next) {
        passport.authenticate('azuread-openidconnect',
          {
            response: res,
            failureRedirect: '/',
            failureFlash: true
          }
        )(req,res,next);
      },
      function(req, res) {
        // TEMPORARY!
        // Flash the access token for testing purposes
        req.flash('error_msg', {message: 'Access token', debug: req.user.accessToken});
        res.redirect('/');
      }
    );

    router.get('/signout',
      function(req, res) {
        req.session.destroy(function(err) {
          req.logout();
          res.redirect('/');
        });
      }
    );

    module.exports = router;
    ```

    Dadurch wird ein Router mit drei Routen definiert `signin`: `callback`, und `signout`.

    Die `signin` Route Ruft die `passport.authenticate` Methode auf, wodurch die APP an die Azure-Anmeldeseite umgeleitet wird.

    Auf `callback` der Route wird nach Abschluss der Anmeldung Azure umgeleitet. Der Code Ruft die `passport.authenticate` Methode erneut auf, wodurch `passport-azure-ad` die Strategie ein Zugriffstoken anfordert. Sobald das Token abgerufen wurde, wird der nächste Handler aufgerufen, der zurück zur Startseite mit dem Zugriffstoken im temporären Fehlerwert umgeleitet wird. Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren. Bevor wir testen, müssen wir die Express-App so konfigurieren, dass Sie den neuen `./routes/auth.js`Router verwendet.

    Die `signout` -Methode protokolliert den Benutzer und zerstört die Sitzung.

1. Öffnen `./app.js` Sie den folgenden Code, und fügen `var app = express();` Sie ihn **vor** die Codezeile ein.

    ```javascript
    var authRouter = require('./routes/auth');
    ```

1. Fügen Sie den folgenden **after** Code nach `app.use('/', indexRouter);` der Codezeile ein.

    ```javascript
    app.use('/auth', authRouter);
    ```

Starten Sie den Server, und `https://localhost:3000`navigieren Sie zu. Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser leitet zur App um, in der Sie das Token sehen.

### <a name="get-user-details"></a>Benutzerdetails abrufen

1. Erstellen Sie eine neue Datei im Stamm des Projekts mit dem `graph.js` Namen, und fügen Sie den folgenden Code hinzu.

    ```javascript
    var graph = require('@microsoft/microsoft-graph-client');
    require('isomorphic-fetch');

    module.exports = {
      getUserDetails: async function(accessToken) {
        const client = getAuthenticatedClient(accessToken);

        const user = await client.api('/me').get();
        return user;
      }
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

    Dadurch wird die `getUserDetails` Funktion exportiert, die das Microsoft Graph-SDK verwendet, `/me` um den Endpunkt aufzurufen und das Ergebnis zurückzugeben.

1. Öffnen `/app.js` Sie und fügen Sie `require` die folgenden Anweisungen am Anfang der Datei hinzu.

    ```javascript
    var graph = require('./graph');
    ```

1. Ersetzen Sie die vorhandene `signInComplete`-Funktion durch den folgenden Code.

    ```javascript
    async function signInComplete(iss, sub, profile, accessToken, refreshToken, params, done) {
      if (!profile.oid) {
        return done(new Error("No OID found in user profile."));
      }

      try{
        const user = await graph.getUserDetails(accessToken);

        if (user) {
          // Add properties to profile
          profile['email'] = user.mail ? user.mail : user.userPrincipalName;
        }
      } catch (err) {
        return done(err);
      }

      // Save the profile and tokens in user storage
      users[profile.oid] = { profile, accessToken };
      return done(null, users[profile.oid]);
    }
    ```

    Der neue Code aktualisiert den `profile` von Passport bereitgestellten `email` , um eine Eigenschaft mithilfe der von Microsoft Graph zurückgegebenen Daten hinzuzufügen.

1. Fügen Sie Folgendes **nach** der `app.use(passport.session());` -Verbindung hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="AddProfileSnippet":::

    Durch diesen Code wird das Benutzerprofil in `locals` die Eigenschaft der Antwort geladen. Dadurch wird es für alle Ansichten in der app verfügbar gemacht.

## <a name="storing-the-tokens"></a>Speichern des Tokens

Nun, da Sie Token abrufen können, ist es an der Zeit, eine Möglichkeit einzurichten, diese in der App zu speichern. Derzeit speichert die APP das unformatierte Zugriffstoken im Speicher des speicherresidenten Benutzers. Da es sich um eine Beispiel-App handelt, werden Sie Sie aus Gründen der Einfachheit weiterhin dort speichern. Eine echte App würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.

Wenn Sie jedoch nur das Zugriffstoken speichern, können Sie das Ablaufdatum überprüfen oder das Token aktualisieren. Um dies zu ermöglichen, aktualisieren Sie das Beispiel so, dass die Token in `AccessToken` einem Objekt aus `simple-oauth2` der Bibliothek umbrochen werden.

1. Öffnen `./app.js` Sie den folgenden Code, und fügen `signInComplete` Sie ihn **vor** der Funktion hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="ConfigureOAuth2Snippet":::

1. Ersetzen Sie die vorhandene `signInComplete`-Funktion durch Folgendes.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="SignInCompleteSnippet" highlight="17-18, 21":::

1. Ersetzen Sie die vorhandene Rückruf Route `./routes/auth.js` in durch Folgendes.

    :::code language="javascript" source="../demo/graph-tutorial/routes/auth.js" id="CallbackRouteSnippet" highlight="17-18":::

1. Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort. Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen. Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach seiner Ausgabe ab. An dieser Stelle kommt das Aktualisierungstoken ins Spiel. Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.

1. Erstellen `tokens.js` Sie eine neue Datei im Stamm des Projekts, in dem die Token-Verwaltungsfunktionen gespeichert sind. Fügen Sie den folgenden Code hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/tokens.js" id="TokensSnippet":::

Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen oder nahezu abläuft. Wenn dies der Fall ist, wird das Aktualisierungstoken verwendet, um neue Token abzurufen, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben. Sie verwenden diese Methode immer dann, wenn Sie das Zugriffstoken aus dem Speicher holen müssen.
