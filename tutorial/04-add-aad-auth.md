<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten. In diesem Schritt werden Sie die [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) -Bibliothek in die Anwendung integrieren.

Erstellen Sie im Stammverzeichnis `.env` der Anwendung eine neue Datei mit dem Namen file, und fügen Sie den folgenden Code hinzu.

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

Ersetzen `YOUR APP ID HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR APP SECRET HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.

> [!IMPORTANT]
> Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die Datei `.env` aus der Quellcodeverwaltung auszuschließen, damit versehentlich keine APP-ID und Ihr Kennwort verloren gehen.

Öffnen `./app.js` Sie und fügen Sie die folgende Codezeile am Anfang der Datei hinzu, um `.env` die Datei zu laden.

```js
require('dotenv').config();
```

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

Suchen Sie die `var indexRouter = require('./routes/index');` - `./app.js`Position in. Fügen Sie den folgenden Code **vor** dieser Codezeile ein.

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

Dieser Code initialisiert die [Passport. js](http://www.passportjs.org/) -Bibliothek für die `passport-azure-ad` Verwendung der Bibliothek und konfiguriert Sie mit der APP-ID und dem Kennwort für die app.

Übergeben Sie nun `passport` das Objekt an die Express-App. Suchen Sie die `app.use('/', indexRouter);` - `./app.js`Position in. Fügen Sie den folgenden Code **vor** dieser Codezeile ein.

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

Dadurch wird ein Router mit drei Routen definiert `signin`: `callback`, und `signout`.

Die `signin` Route Ruft die `passport.authenticate` Methode auf, wodurch die APP an die Azure-Anmeldeseite umgeleitet wird.

Auf `callback` der Route wird nach Abschluss der Anmeldung Azure umgeleitet. Der Code Ruft die `passport.authenticate` Methode erneut auf, wodurch `passport-azure-ad` die Strategie ein Zugriffstoken anfordert. Sobald das Token abgerufen wurde, wird der nächste Handler aufgerufen, der zurück zur Startseite mit dem Zugriffstoken im temporären Fehlerwert umgeleitet wird. Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren. Bevor wir testen, müssen wir die Express-App so konfigurieren, dass Sie den neuen `./routes/auth.js`Router verwendet.

Die `signout` -Methode protokolliert den Benutzer und zerstört die Sitzung.

Fügen Sie den folgenden **** Code vor `var app = express();` die-Codezeile ein.

```js
var authRouter = require('./routes/auth');
```

Fügen Sie dann den folgenden **** Code nach `app.use('/', indexRouter);` der Codezeile ein.

```js
app.use('/auth', authRouter);
```

Starten Sie den Server, und `https://localhost:3000`navigieren Sie zu. Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden. Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu. Der Browser wird an die APP umgeleitet, wobei das Token angezeigt wird.

### <a name="get-user-details"></a>Abrufen von Benutzer Details

Erstellen Sie zunächst eine neue Datei, in der alle Microsoft Graph-Anrufe gespeichert sind. Erstellen Sie eine neue Datei im Stamm des Projekts mit dem `graph.js` Namen, und fügen Sie den folgenden Code hinzu.

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

Dadurch wird die `getUserDetails` Funktion exportiert, die das Microsoft Graph-SDK verwendet, `/me` um den Endpunkt aufzurufen und das Ergebnis zurückzugeben.

Aktualisieren Sie `signInComplete` die- `/app.js` Methode in, um diese Funktion aufzurufen. Fügen Sie zunächst die folgenden `require` Anweisungen am Anfang der Datei hinzu.

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

Der neue Code aktualisiert den `profile` von Passport bereitgestellten `email` , um eine Eigenschaft mithilfe der von Microsoft Graph zurückgegebenen Daten hinzuzufügen.

Fügen Sie schließlich Code `./app.js` hinzu, um das Benutzerprofil in die `locals` Eigenschaft der Antwort zu laden. Dadurch wird es für alle Ansichten in der app verfügbar gemacht.

Fügen Sie Folgendes **nach** der `app.use(passport.session());` -Verbindung hinzu.

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

Nachdem Sie nun Token erhalten können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren. Derzeit speichert die APP das unformatierte Zugriffstoken im Speicher des speicherresidenten Benutzers. Da es sich um eine Beispiel-App handelt, werden Sie Sie aus Gründen der Einfachheit weiterhin dort speichern. Eine reale APP würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.

Wenn Sie jedoch nur das Zugriffstoken speichern, können Sie das Ablaufdatum überprüfen oder das Token aktualisieren. Um dies zu ermöglichen, aktualisieren Sie das Beispiel so, dass die Token in `AccessToken` einem Objekt aus `simple-oauth2` der Bibliothek umbrochen werden.

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

Aktualisieren Sie dann die `signInComplete` -Funktion, um `AccessToken` eine aus den unformatierten Token zu erstellen, die übergeben wurden, und speichern Sie diese im Benutzerspeicher. Ersetzen Sie die vorhandene `signInComplete`-Funktion durch Folgendes.

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

Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort. Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.

![Ein Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen. Durch Klicken auf **Abmelden** wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite zurück.

![Screenshot des Dropdownmenüs mit dem Link zum Abmelden](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Aktualisieren von Token

Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird. Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.

Dieses Token ist jedoch nur kurzlebig. Das Token läuft eine Stunde nach seiner Ausgabe ab. Hier wird das Aktualisierungstoken nützlich. Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass sich der Benutzer erneut anmelden muss.

Um dies zu verwalten, erstellen Sie eine neue Datei im Stamm des Projekts mit `tokens.js` dem Namen, in dem die Token-Verwaltungsfunktionen gespeichert sind. Fügen Sie den folgenden Code hinzu.

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

Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen oder nahezu abläuft. Wenn dies der Fall ist, wird das Aktualisierungstoken verwendet, um neue Token abzurufen, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben. Sie verwenden diese Methode immer dann, wenn Sie das Zugriffstoken aus dem Speicher holen müssen.
