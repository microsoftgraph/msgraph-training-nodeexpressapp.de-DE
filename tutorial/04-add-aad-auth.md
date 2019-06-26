<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a7f42-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="a7f42-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="a7f42-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="a7f42-103">In diesem Schritt werden Sie die [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) -Bibliothek in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="a7f42-103">In this step you will integrate the [passport-azure-ad](https://github.com/AzureAD/passport-azure-ad) library into the application.</span></span>

<span data-ttu-id="a7f42-104">Erstellen Sie im Stammverzeichnis `.env` der Anwendung eine neue Datei mit dem Namen file, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-104">Create a new file named `.env` file in the root of your application, and add the following code.</span></span>

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

<span data-ttu-id="a7f42-105">Ersetzen `YOUR APP ID HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR APP SECRET HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.</span><span class="sxs-lookup"><span data-stu-id="a7f42-105">Replace `YOUR APP ID HERE` with the application ID from the Application Registration Portal, and replace `YOUR APP SECRET HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="a7f42-106">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die Datei `.env` aus der Quellcodeverwaltung auszuschließen, damit versehentlich keine APP-ID und Ihr Kennwort verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="a7f42-107">Öffnen `./app.js` Sie und fügen Sie die folgende Codezeile am Anfang der Datei hinzu, um `.env` die Datei zu laden.</span><span class="sxs-lookup"><span data-stu-id="a7f42-107">Open `./app.js` and add the following line to the top of the file to load the `.env` file.</span></span>

```js
require('dotenv').config();
```

## <a name="implement-sign-in"></a><span data-ttu-id="a7f42-108">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="a7f42-108">Implement sign-in</span></span>

<span data-ttu-id="a7f42-109">Suchen Sie die `var indexRouter = require('./routes/index');` - `./app.js`Position in.</span><span class="sxs-lookup"><span data-stu-id="a7f42-109">Locate the line `var indexRouter = require('./routes/index');` in `./app.js`.</span></span> <span data-ttu-id="a7f42-110">Fügen Sie den folgenden Code **vor** dieser Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="a7f42-110">Insert the following code **before** that line.</span></span>

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

<span data-ttu-id="a7f42-111">Dieser Code initialisiert die [Passport. js](http://www.passportjs.org/) -Bibliothek für die `passport-azure-ad` Verwendung der Bibliothek und konfiguriert Sie mit der APP-ID und dem Kennwort für die app.</span><span class="sxs-lookup"><span data-stu-id="a7f42-111">This code initializes the [Passport.js](http://www.passportjs.org/) library to use the `passport-azure-ad` library, and configures it with the app ID and password for the app.</span></span>

<span data-ttu-id="a7f42-112">Übergeben Sie nun `passport` das Objekt an die Express-App.</span><span class="sxs-lookup"><span data-stu-id="a7f42-112">Now pass the `passport` object to the Express app.</span></span> <span data-ttu-id="a7f42-113">Suchen Sie die `app.use('/', indexRouter);` - `./app.js`Position in.</span><span class="sxs-lookup"><span data-stu-id="a7f42-113">Locate the line `app.use('/', indexRouter);` in `./app.js`.</span></span> <span data-ttu-id="a7f42-114">Fügen Sie den folgenden Code **vor** dieser Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="a7f42-114">Insert the following code **before** that line.</span></span>

```js
// Initialize passport
app.use(passport.initialize());
app.use(passport.session());
```

<span data-ttu-id="a7f42-115">Erstellen Sie eine neue Datei im `./routes` Verzeichnis mit `auth.js` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-115">Create a new file in the `./routes` directory named `auth.js` and add the following code.</span></span>

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

<span data-ttu-id="a7f42-116">Dadurch wird ein Router mit drei Routen definiert `signin`: `callback`, und `signout`.</span><span class="sxs-lookup"><span data-stu-id="a7f42-116">This defines a router with three routes: `signin`, `callback`, and `signout`.</span></span>

<span data-ttu-id="a7f42-117">Die `signin` Route Ruft die `passport.authenticate` Methode auf, wodurch die APP an die Azure-Anmeldeseite umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="a7f42-117">The `signin` route calls the `passport.authenticate` method, causing the app to redirect to the Azure login page.</span></span>

<span data-ttu-id="a7f42-118">Auf `callback` der Route wird nach Abschluss der Anmeldung Azure umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="a7f42-118">The `callback` route is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="a7f42-119">Der Code Ruft die `passport.authenticate` Methode erneut auf, wodurch `passport-azure-ad` die Strategie ein Zugriffstoken anfordert.</span><span class="sxs-lookup"><span data-stu-id="a7f42-119">The code calls the `passport.authenticate` method again, causing the `passport-azure-ad` strategy to request an access token.</span></span> <span data-ttu-id="a7f42-120">Sobald das Token abgerufen wurde, wird der nächste Handler aufgerufen, der zurück zur Startseite mit dem Zugriffstoken im temporären Fehlerwert umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="a7f42-120">Once the token is obtained, the next handler is called, which redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="a7f42-121">Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren.</span><span class="sxs-lookup"><span data-stu-id="a7f42-121">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="a7f42-122">Bevor wir testen, müssen wir die Express-App so konfigurieren, dass Sie den neuen `./routes/auth.js`Router verwendet.</span><span class="sxs-lookup"><span data-stu-id="a7f42-122">Before we test, we need to configure the Express app to use the new router from `./routes/auth.js`.</span></span>

<span data-ttu-id="a7f42-123">Die `signout` -Methode protokolliert den Benutzer und zerstört die Sitzung.</span><span class="sxs-lookup"><span data-stu-id="a7f42-123">The `signout` method logs the user out and destroys the session.</span></span>

<span data-ttu-id="a7f42-124">Fügen Sie den folgenden \*\*\*\* Code vor `var app = express();` die-Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="a7f42-124">Insert the following code **before** the `var app = express();` line.</span></span>

```js
var authRouter = require('./routes/auth');
```

<span data-ttu-id="a7f42-125">Fügen Sie dann den folgenden \*\*\*\* Code nach `app.use('/', indexRouter);` der Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="a7f42-125">Then insert the following code **after** the `app.use('/', indexRouter);` line.</span></span>

```js
app.use('/auth', authRouter);
```

<span data-ttu-id="a7f42-126">Starten Sie den Server, und `https://localhost:3000`navigieren Sie zu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-126">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="a7f42-127">Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="a7f42-127">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="a7f42-128">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-128">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="a7f42-129">Der Browser wird an die APP umgeleitet, wobei das Token angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="a7f42-129">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="a7f42-130">Abrufen von Benutzer Details</span><span class="sxs-lookup"><span data-stu-id="a7f42-130">Get user details</span></span>

<span data-ttu-id="a7f42-131">Erstellen Sie zunächst eine neue Datei, in der alle Microsoft Graph-Anrufe gespeichert sind.</span><span class="sxs-lookup"><span data-stu-id="a7f42-131">Start by creating a new file to hold all of your Microsoft Graph calls.</span></span> <span data-ttu-id="a7f42-132">Erstellen Sie eine neue Datei im Stamm des Projekts mit dem `graph.js` Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-132">Create a new file in the root of the project named `graph.js` and add the following code.</span></span>

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

<span data-ttu-id="a7f42-133">Dadurch wird die `getUserDetails` Funktion exportiert, die das Microsoft Graph-SDK verwendet, `/me` um den Endpunkt aufzurufen und das Ergebnis zurückzugeben.</span><span class="sxs-lookup"><span data-stu-id="a7f42-133">This exports the `getUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

<span data-ttu-id="a7f42-134">Aktualisieren Sie `signInComplete` die- `/app.s` Methode in, um diese Funktion aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-134">Update the `signInComplete` method in `/app.s` to call this function.</span></span> <span data-ttu-id="a7f42-135">Fügen Sie zunächst die folgenden `require` Anweisungen am Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-135">First, add the following `require` statements to the top of the file.</span></span>

```js
var graph = require('./graph');
```

<span data-ttu-id="a7f42-136">Ersetzen Sie die vorhandene `signInComplete`-Funktion durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="a7f42-136">Replace the existing `signInComplete` function with the following code.</span></span>

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

<span data-ttu-id="a7f42-137">Der neue Code aktualisiert den `profile` von Passport bereitgestellten `email` , um eine Eigenschaft mithilfe der von Microsoft Graph zurückgegebenen Daten hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-137">The new code updates the `profile` provided by Passport to add an `email` property, using the data returned by Microsoft Graph.</span></span>

<span data-ttu-id="a7f42-138">Fügen Sie schließlich Code `./app.js` hinzu, um das Benutzerprofil in die `locals` Eigenschaft der Antwort zu laden.</span><span class="sxs-lookup"><span data-stu-id="a7f42-138">Finally, add code to `./app.js` to load the user profile into the `locals` property of the response.</span></span> <span data-ttu-id="a7f42-139">Dadurch wird es für alle Ansichten in der app verfügbar gemacht.</span><span class="sxs-lookup"><span data-stu-id="a7f42-139">This will make it available to all of the views in the app.</span></span>

<span data-ttu-id="a7f42-140">Fügen Sie Folgendes **nach** der `app.use(passport.session());` -Verbindung hinzu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-140">Add the following **after** the `app.use(passport.session());` line.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="a7f42-141">Speichern der Token</span><span class="sxs-lookup"><span data-stu-id="a7f42-141">Storing the tokens</span></span>

<span data-ttu-id="a7f42-142">Nachdem Sie nun Token erhalten können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="a7f42-142">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="a7f42-143">Derzeit speichert die APP das unformatierte Zugriffstoken im Speicher des speicherresidenten Benutzers.</span><span class="sxs-lookup"><span data-stu-id="a7f42-143">Currently, the app is storing the raw access token in the in-memory user storage.</span></span> <span data-ttu-id="a7f42-144">Da es sich um eine Beispiel-App handelt, werden Sie Sie aus Gründen der Einfachheit weiterhin dort speichern.</span><span class="sxs-lookup"><span data-stu-id="a7f42-144">Since this is a sample app, for simplicity's sake, you'll continue to store them there.</span></span> <span data-ttu-id="a7f42-145">Eine reale APP würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.</span><span class="sxs-lookup"><span data-stu-id="a7f42-145">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="a7f42-146">Wenn Sie jedoch nur das Zugriffstoken speichern, können Sie das Ablaufdatum überprüfen oder das Token aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="a7f42-146">However, storing just the access token doesn't allow you to check expiration or refresh the token.</span></span> <span data-ttu-id="a7f42-147">Um dies zu ermöglichen, aktualisieren Sie das Beispiel so, dass die Token in `AccessToken` einem Objekt aus `simple-oauth2` der Bibliothek umbrochen werden.</span><span class="sxs-lookup"><span data-stu-id="a7f42-147">In order to enable that, update the sample to wrap the tokens in an `AccessToken` object from the `simple-oauth2` library.</span></span>

<span data-ttu-id="a7f42-148">Fügen Sie zunächst `./app.js`in den folgenden Code **vor** der `signInComplete` Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-148">First, in `./app.js`, add the following code **before** the `signInComplete` function.</span></span>

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

<span data-ttu-id="a7f42-149">Aktualisieren Sie dann die `signInComplete` -Funktion, um `AccessToken` eine aus den unformatierten Token zu erstellen, die übergeben wurden, und speichern Sie diese im Benutzerspeicher.</span><span class="sxs-lookup"><span data-stu-id="a7f42-149">Then, update the `signInComplete` function to create an `AccessToken` from the raw tokens passed in and store that in the user storage.</span></span> <span data-ttu-id="a7f42-150">Ersetzen Sie die vorhandene `signInComplete`-Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="a7f42-150">Replace the existing `signInComplete` function with the following.</span></span>

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

<span data-ttu-id="a7f42-151">Aktualisieren Sie `callback` die Route `./routes/auth.js` in, um `req.flash` die Linie mit dem Zugriffstoken zu entfernen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-151">Update the `callback` route in `./routes/auth.js` to remove the `req.flash` line with the access token.</span></span> <span data-ttu-id="a7f42-152">Die `callback` Route sollte wie folgt aussehen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-152">The `callback` route should look like the following.</span></span>

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

<span data-ttu-id="a7f42-153">Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort.</span><span class="sxs-lookup"><span data-stu-id="a7f42-153">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="a7f42-154">Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="a7f42-154">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Ein Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

<span data-ttu-id="a7f42-156">Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers \*\*\*\* , um auf den Abmeldelink zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-156">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="a7f42-157">Durch \*\*\*\* klicken auf Abmelden wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="a7f42-157">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Screenshot des Dropdownmenüs mit dem Link zum Abmelden](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="a7f42-159">Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="a7f42-159">Refreshing tokens</span></span>

<span data-ttu-id="a7f42-160">Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="a7f42-160">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="a7f42-161">Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-161">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="a7f42-162">Dieses Token ist jedoch nur kurzlebig.</span><span class="sxs-lookup"><span data-stu-id="a7f42-162">However, this token is short-lived.</span></span> <span data-ttu-id="a7f42-163">Das Token läuft eine Stunde nach seiner Ausgabe ab.</span><span class="sxs-lookup"><span data-stu-id="a7f42-163">The token expires an hour after it is issued.</span></span> <span data-ttu-id="a7f42-164">Hier wird das Aktualisierungstoken nützlich.</span><span class="sxs-lookup"><span data-stu-id="a7f42-164">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="a7f42-165">Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass sich der Benutzer erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="a7f42-165">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="a7f42-166">Um dies zu verwalten, erstellen Sie eine neue Datei im Stamm des Projekts mit `tokens.js` dem Namen, in dem die Token-Verwaltungsfunktionen gespeichert sind.</span><span class="sxs-lookup"><span data-stu-id="a7f42-166">To manage this, create a new file in the root of the project named `tokens.js` to hold token management functions.</span></span> <span data-ttu-id="a7f42-167">Fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a7f42-167">Add the following code.</span></span>

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

<span data-ttu-id="a7f42-168">Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen oder nahezu abläuft.</span><span class="sxs-lookup"><span data-stu-id="a7f42-168">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="a7f42-169">Wenn dies der Fall ist, wird das Aktualisierungstoken verwendet, um neue Token abzurufen, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="a7f42-169">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span> <span data-ttu-id="a7f42-170">Sie verwenden diese Methode immer dann, wenn Sie das Zugriffstoken aus dem Speicher holen müssen.</span><span class="sxs-lookup"><span data-stu-id="a7f42-170">You'll use this method whenever you need to get the access token out of storage.</span></span>
