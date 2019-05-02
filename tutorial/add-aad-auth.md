<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung zur Unterstützung der Authentifizierung mit Azure AD. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken für den Aufruf von Microsoft Graph abzurufen. In diesem Schritt integrieren Sie die [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) -Bibliothek in die Anwendung.

Erstellen Sie eine neue Datei `.env` mit dem Namen File im Stammverzeichnis Ihrer Anwendung, und fügen Sie den folgenden Code hinzu.

```text
OAUTH_APP_ID=YOUR_APP_ID_HERE
OAUTH_APP_PASSWORD=YOUR_APP_PASSWORD_HERE
OAUTH_REDIRECT_URI=http://localhost:3000/auth/callback
OAUTH_SCOPES='profile offline_access user.read calendars.read'
OAUTH_AUTHORITY=https://login.microsoftonline.com/common
OAUTH_ID_METADATA=/v2.0/.well-known/openid-configuration
OAUTH_AUTHORIZE_ENDPOINT=/oauth2/v2.0/authorize
OAUTH_TOKEN_ENDPOINT=/oauth2/v2.0/token
```

Ersetzen `YOUR APP ID HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR APP SECRET HERE` und ersetzen Sie es durch das von Ihnen generierte Kennwort.

> [!IMPORTANT]
> Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, um die `.env` Datei aus der Quellcodeverwaltung auszuschließen, um versehentlich Ihre APP-ID und Ihr Kennwort zu verlieren.

Öffnen `./app.js` Sie, und fügen Sie die folgende Codezeile am Anfang der Datei hinzu, `.env` um die Datei zu laden.

```js
require('dotenv').config();
```

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

Suchen Sie die `var indexRouter = require('./routes/index');` Position `./app.js`in. Fügen Sie den folgenden Code **vor** dieser Leitung ein.

```js
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
    return done(new Error("No OID found in user profile."), null);
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

Dieser Code initialisiert die [Passport. js](http://www.passportjs.org/) -Bibliothek, um `passport-azure-ad` die Bibliothek zu verwenden, und konfiguriert Sie mit der APP-ID und dem Kennwort für die app.

Geben Sie das `passport` Objekt nun an die Express-App weiter. Suchen Sie die `app.use('/', indexRouter);` Position `./app.js`in. Fügen Sie den folgenden Code **vor** dieser Leitung ein.

```js
// Initialize passport
app.use(passport.initialize());
app.use(passport.session());
```

Erstellen Sie eine neue Datei im `./routes` Verzeichnis mit `auth.js` dem Namen, und fügen Sie den folgenden Code hinzu.

```js
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
        failureFlash: true
      }
    )(req,res,next);
  },
  function(req, res) {
    res.redirect('/');
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

Dies definiert einen Router mit drei Routen: `signin`, `callback`und `signout`.

Die `signin` Route Ruft die `passport.authenticate` Methode auf, wodurch die APP zur Azure-Anmeldeseite umgeleitet wird.

Bei `callback` der Route wird Azure umgeleitet, nachdem die signierin abgeschlossen wurde. Der Code Ruft die `passport.authenticate` -Methode erneut auf, `passport-azure-ad` wodurch die Strategie ein Zugriffstoken anfordert. Sobald das Token abgerufen wurde, wird der nächste Handler aufgerufen, der zurück zur Startseite mit dem Zugriffstoken im temporären Fehlerwert umgeleitet wird. Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktioniert, bevor Sie fortfahren. Bevor wir testen, müssen wir die Express-App so konfigurieren, dass der neue Router `./routes/auth.js`aus verwendet werden kann.

Die `signout` -Methode meldet den Benutzer ab und zerstört die Sitzung.

Fügen Sie den folgenden **** Code vor `var app = express();` der Leitung ein.

```js
var authRouter = require('./routes/auth');
```

Fügen Sie dann den folgenden **** Code nach `app.use('/', indexRouter);` der Leitung ein.

```js
app.use('/auth', authRouter);
```

Starten Sie den Server, und `https://localhost:3000`navigieren Sie zu. Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den erforderlichen Berechtigungen zu. Der Browser leitet zur App um, wobei das Token angezeigt wird.

### <a name="get-user-details"></a>Benutzer Details abrufen

Erstellen Sie zunächst eine neue Datei für alle Microsoft Graph-Aufrufe. Erstellen Sie im Stamm des Projekts eine neue Datei `graph.js` , und fügen Sie den folgenden Code hinzu.

```js
var graph = require('@microsoft/microsoft-graph-client');

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

Dadurch wird die `getUserDetails` Funktion exportiert, die das Microsoft Graph-SDK zum Aufrufen `/me` des Endpunkts und zum Zurückgeben des Ergebnisses verwendet.

Aktualisieren Sie `signInComplete` die- `/app.s` Methode in, um diese Funktion aufzurufen. Fügen Sie zunächst die folgenden `require` Anweisungen am Anfang der Datei hinzu.

```js
var graph = require('./graph');
```

Ersetzen Sie die vorhandene `signInComplete`-Funktion durch den folgenden Code.

```js
async function signInComplete(iss, sub, profile, accessToken, refreshToken, params, done) {
  if (!profile.oid) {
    return done(new Error("No OID found in user profile."), null);
  }

  try{
    const user = await graph.getUserDetails(accessToken);

    if (user) {
      // Add properties to profile
      profile['email'] = user.mail ? user.mail : user.userPrincipalName;
    }
  } catch (err) {
    done(err, null);
  }

  // Save the profile and tokens in user storage
  users[profile.oid] = { profile, accessToken };
  return done(null, users[profile.oid]);
}
```

Der neue Code aktualisiert den `profile` bereitgestellten durch Passport `email` , um eine Eigenschaft mithilfe der von Microsoft Graph zurückgegebenen Daten hinzuzufügen.

Fügen Sie schließlich Code `./app.js` hinzu, um das Benutzerprofil in die `locals` Eigenschaft der Antwort zu laden. Dadurch wird es für alle Ansichten in der App zur Verfügung gestellt.

Fügen Sie den **** folgenden nach `app.use(passport.session());` der Leitung hinzu.

```js
app.use(function(req, res, next) {
  // Set the authenticated user in the
  // template locals
  if (req.user) {
    res.locals.user = req.user.profile;
  }
  next();
});
```

## <a name="storing-the-tokens"></a>Speichern der Token

Da Sie jetzt Tokens abrufen können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren. Derzeit speichert die APP das RAW-Zugriffstoken in der speicherresidenten Speicher der Benutzer. Da es sich um eine Beispiel-App handelt, werden Sie Sie auch weiterhin dort speichern. Eine echte APP würde eine zuverlässigere Secure Storage-Lösung wie eine Datenbank verwenden.

Das Speichern des Zugriffstokens ermöglicht jedoch nicht das Überprüfen des Ablaufs oder das Aktualisieren des Tokens. Um dies zu ermöglichen, aktualisieren Sie das Beispiel, um die Token in einem `AccessToken` Objekt aus der `simple-oauth2` Bibliothek einzuschließen.

Fügen Sie zunächst `./app.js`in den folgenden Code **vor** der `signInComplete` Funktion hinzu.

```js
// Configure simple-oauth2
const oauth2 = require('simple-oauth2').create({
  client: {
    id: process.env.OAUTH_APP_ID,
    secret: process.env.OAUTH_APP_PASSWORD
  },
  auth: {
    tokenHost: process.env.OAUTH_AUTHORITY,
    authorizePath: process.env.OAUTH_AUTHORIZE_ENDPOINT,
    tokenPath: process.env.OAUTH_TOKEN_ENDPOINT
  }
});
```

Aktualisieren Sie dann die `signInComplete` -Funktion, um `AccessToken` eine aus den unformatierten Token zu erstellen, die übergeben werden, und speichern Sie diese im Benutzerspeicher. Ersetzen Sie die vorhandene `signInComplete`-Funktion durch Folgendes.

```js
async function signInComplete(iss, sub, profile, accessToken, refreshToken, params, done) {
  if (!profile.oid) {
    return done(new Error("No OID found in user profile."), null);
  }

  try{
    const user = await graph.getUserDetails(accessToken);

    if (user) {
      // Add properties to profile
      profile['email'] = user.mail ? user.mail : user.userPrincipalName;
    }
  } catch (err) {
    done(err, null);
  }

  // Create a simple-oauth2 token from raw tokens
  let oauthToken = oauth2.accessToken.create(params);

  // Save the profile and tokens in user storage
  users[profile.oid] = { profile, oauthToken };
  return done(null, users[profile.oid]);
}
```

Aktualisieren Sie `callback` die Route `./routes/auth.js` in, um `req.flash` die Linie mit dem Zugriffstoken zu entfernen. Die `callback` Route sollte wie folgt aussehen.

```js
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
    res.redirect('/');
  }
);
```

Starten Sie den Server neu, und führen Sie den Anmeldevorgang durch. Sie sollten auf der Startseite zurückkehren, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

![Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

Klicken Sie auf den Benutzer Avatar in der oberen rechten Ecke, **** um auf den Link abmelden zuzugreifen. Wenn **** Sie auf Abmelden klicken, wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite.

![Screenshot des Dropdownmenüs mit dem Link "Abmelden"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, mit dem die APP auf Microsoft Graph im Namen des Benutzers zugreifen kann.

Dieses Token ist jedoch kurzlebig. Das Token läuft eine Stunde nach der Ausgabe ab. An dieser Stelle wird das Aktualisierungstoken nützlich. Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.

Um dies zu verwalten, erstellen Sie eine neue Datei im Stamm des Projekts `tokens.js` , in dem die Tokenverwaltung-Funktionen aufbewahrt werden sollen. Fügen Sie den folgenden Code hinzu.

```js
module.exports = {
  getAccessToken: async function(req) {
    if (req.user) {
      // Get the stored token
      var storedToken = req.user.oauthToken;

      if (storedToken) {
        if (storedToken.expired()) {
          // refresh token
          var newToken = await storedToken.refresh();

          // Update stored token
          req.user.oauthToken = newToken;
          return newToken.token.access_token;
        }

        // Token still valid, just return it
        return storedToken.token.access_token;
      }
    }
  }
};
```

Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen ist oder in Kürze abläuft. Wenn dies der Fall ist, wird das Aktualisierungstoken zum Abrufen neuer Token verwendet, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben. Sie verwenden diese Methode immer dann, wenn das Zugriffstoken nicht mehr gespeichert werden muss.