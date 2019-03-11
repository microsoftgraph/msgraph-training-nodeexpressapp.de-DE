<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1061c-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung zur Unterstützung der Authentifizierung mit Azure AD.</span><span class="sxs-lookup"><span data-stu-id="1061c-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="1061c-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken für den Aufruf von Microsoft Graph abzurufen.</span><span class="sxs-lookup"><span data-stu-id="1061c-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="1061c-103">In diesem Schritt integrieren Sie die [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) -Bibliothek in die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="1061c-103">In this step you will integrate the [passport-azure-ad](https://github.com/AzureAD/passport-azure-ad) library into the application.</span></span>

<span data-ttu-id="1061c-104">Erstellen Sie eine neue Datei `.env` mit dem Namen File im Stammverzeichnis Ihrer Anwendung, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="1061c-104">Create a new file named `.env` file in the root of your application, and add the following code.</span></span>

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

<span data-ttu-id="1061c-105">Ersetzen `YOUR APP ID HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR APP SECRET HERE` und ersetzen Sie es durch das von Ihnen generierte Kennwort.</span><span class="sxs-lookup"><span data-stu-id="1061c-105">Replace `YOUR APP ID HERE` with the application ID from the Application Registration Portal, and replace `YOUR APP SECRET HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="1061c-106">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, um die `.env` Datei aus der Quellcodeverwaltung auszuschließen, um versehentlich Ihre APP-ID und Ihr Kennwort zu verlieren.</span><span class="sxs-lookup"><span data-stu-id="1061c-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="1061c-107">Öffnen `./app.js` Sie, und fügen Sie die folgende Codezeile am Anfang der Datei hinzu, `.env` um die Datei zu laden.</span><span class="sxs-lookup"><span data-stu-id="1061c-107">Open `./app.js` and add the following line to the top of the file to load the `.env` file.</span></span>

```js
require('dotenv').config();
```

## <a name="implement-sign-in"></a><span data-ttu-id="1061c-108">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="1061c-108">Implement sign-in</span></span>

<span data-ttu-id="1061c-109">Suchen Sie die `var indexRouter = require('./routes/index');` Position `./app.js`in.</span><span class="sxs-lookup"><span data-stu-id="1061c-109">Locate the line `var indexRouter = require('./routes/index');` in `./app.js`.</span></span> <span data-ttu-id="1061c-110">Fügen Sie den folgenden Code **vor** dieser Leitung ein.</span><span class="sxs-lookup"><span data-stu-id="1061c-110">Insert the following code **before** that line.</span></span>

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

<span data-ttu-id="1061c-111">Dieser Code initialisiert die [Passport. js](http://www.passportjs.org/) -Bibliothek, um `passport-azure-ad` die Bibliothek zu verwenden, und konfiguriert Sie mit der APP-ID und dem Kennwort für die app.</span><span class="sxs-lookup"><span data-stu-id="1061c-111">This code initializes the [Passport.js](http://www.passportjs.org/) library to use the `passport-azure-ad` library, and configures it with the app ID and password for the app.</span></span>

<span data-ttu-id="1061c-112">Geben Sie das `passport` Objekt nun an die Express-App weiter.</span><span class="sxs-lookup"><span data-stu-id="1061c-112">Now pass the `passport` object to the Express app.</span></span> <span data-ttu-id="1061c-113">Suchen Sie die `app.use('/', indexRouter);` Position `./app.js`in.</span><span class="sxs-lookup"><span data-stu-id="1061c-113">Locate the line `app.use('/', indexRouter);` in `./app.js`.</span></span> <span data-ttu-id="1061c-114">Fügen Sie den folgenden Code **vor** dieser Leitung ein.</span><span class="sxs-lookup"><span data-stu-id="1061c-114">Insert the following code **before** that line.</span></span>

```js
// Initialize passport
app.use(passport.initialize());
app.use(passport.session());
```

<span data-ttu-id="1061c-115">Erstellen Sie eine neue Datei im `./routes` Verzeichnis mit `auth.js` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="1061c-115">Create a new file in the `./routes` directory named `auth.js` and add the following code.</span></span>

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

<span data-ttu-id="1061c-116">Dies definiert einen Router mit drei Routen: `signin`, `callback`und `signout`.</span><span class="sxs-lookup"><span data-stu-id="1061c-116">This defines a router with three routes: `signin`, `callback`, and `signout`.</span></span>

<span data-ttu-id="1061c-117">Die `signin` Route Ruft die `passport.authenticate` Methode auf, wodurch die APP zur Azure-Anmeldeseite umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="1061c-117">The `signin` route calls the `passport.authenticate` method, causing the app to redirect to the Azure login page.</span></span>

<span data-ttu-id="1061c-118">Bei `callback` der Route wird Azure umgeleitet, nachdem die signierin abgeschlossen wurde.</span><span class="sxs-lookup"><span data-stu-id="1061c-118">The `callback` route is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="1061c-119">Der Code Ruft die `passport.authenticate` -Methode erneut auf, `passport-azure-ad` wodurch die Strategie ein Zugriffstoken anfordert.</span><span class="sxs-lookup"><span data-stu-id="1061c-119">The code calls the `passport.authenticate` method again, causing the `passport-azure-ad` strategy to request an access token.</span></span> <span data-ttu-id="1061c-120">Sobald das Token abgerufen wurde, wird der nächste Handler aufgerufen, der zurück zur Startseite mit dem Zugriffstoken im temporären Fehlerwert umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="1061c-120">Once the token is obtained, the next handler is called, which redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="1061c-121">Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktioniert, bevor Sie fortfahren.</span><span class="sxs-lookup"><span data-stu-id="1061c-121">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="1061c-122">Bevor wir testen, müssen wir die Express-App so konfigurieren, dass der neue Router `./routes/auth.js`aus verwendet werden kann.</span><span class="sxs-lookup"><span data-stu-id="1061c-122">Before we test, we need to configure the Express app to use the new router from `./routes/auth.js`.</span></span>

<span data-ttu-id="1061c-123">Die `signout` -Methode meldet den Benutzer ab und zerstört die Sitzung.</span><span class="sxs-lookup"><span data-stu-id="1061c-123">The `signout` method logs the user out and destroys the session.</span></span>

<span data-ttu-id="1061c-124">Fügen Sie den folgenden \*\*\*\* Code vor `var app = express();` der Leitung ein.</span><span class="sxs-lookup"><span data-stu-id="1061c-124">Insert the following code **before** the `var app = express();` line.</span></span>

```js
var authRouter = require('./routes/auth');
```

<span data-ttu-id="1061c-125">Fügen Sie dann den folgenden \*\*\*\* Code nach `app.use('/', indexRouter);` der Leitung ein.</span><span class="sxs-lookup"><span data-stu-id="1061c-125">Then insert the following code **after** the `app.use('/', indexRouter);` line.</span></span>

```js
app.use('/auth', authRouter);
```

<span data-ttu-id="1061c-126">Starten Sie den Server, und `https://localhost:3000`navigieren Sie zu.</span><span class="sxs-lookup"><span data-stu-id="1061c-126">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="1061c-127">Klicken Sie auf die Anmeldeschaltfläche, und Sie sollten zu `https://login.microsoftonline.com`umgeleitet werden.</span><span class="sxs-lookup"><span data-stu-id="1061c-127">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="1061c-128">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den erforderlichen Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="1061c-128">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="1061c-129">Der Browser leitet zur App um, wobei das Token angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="1061c-129">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="1061c-130">Benutzer Details abrufen</span><span class="sxs-lookup"><span data-stu-id="1061c-130">Get user details</span></span>

<span data-ttu-id="1061c-131">Erstellen Sie zunächst eine neue Datei für alle Microsoft Graph-Aufrufe.</span><span class="sxs-lookup"><span data-stu-id="1061c-131">Start by creating a new file to hold all of your Microsoft Graph calls.</span></span> <span data-ttu-id="1061c-132">Erstellen Sie im Stamm des Projekts eine neue Datei `graph.js` , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="1061c-132">Create a new file in the root of the project named `graph.js` and add the following code.</span></span>

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

<span data-ttu-id="1061c-133">Dadurch wird die `getUserDetails` Funktion exportiert, die das Microsoft Graph-SDK zum Aufrufen `/me` des Endpunkts und zum Zurückgeben des Ergebnisses verwendet.</span><span class="sxs-lookup"><span data-stu-id="1061c-133">This exports the `getUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

<span data-ttu-id="1061c-134">Aktualisieren Sie `signInComplete` die- `/app.s` Methode in, um diese Funktion aufzurufen.</span><span class="sxs-lookup"><span data-stu-id="1061c-134">Update the `signInComplete` method in `/app.s` to call this function.</span></span> <span data-ttu-id="1061c-135">Fügen Sie zunächst die folgenden `require` Anweisungen am Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="1061c-135">First, add the following `require` statements to the top of the file.</span></span>

```js
var graph = require('./graph');
```

<span data-ttu-id="1061c-136">Ersetzen Sie die vorhandene `signInComplete`-Funktion durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="1061c-136">Replace the existing `signInComplete` function with the following code.</span></span>

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

<span data-ttu-id="1061c-137">Der neue Code aktualisiert den `profile` bereitgestellten durch Passport `email` , um eine Eigenschaft mithilfe der von Microsoft Graph zurückgegebenen Daten hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="1061c-137">The new code updates the `profile` provided by Passport to add an `email` property, using the data returned by Microsoft Graph.</span></span>

<span data-ttu-id="1061c-138">Fügen Sie schließlich Code `./app.js` hinzu, um das Benutzerprofil in die `locals` Eigenschaft der Antwort zu laden.</span><span class="sxs-lookup"><span data-stu-id="1061c-138">Finally, add code to `./app.js` to load the user profile into the `locals` property of the response.</span></span> <span data-ttu-id="1061c-139">Dadurch wird es für alle Ansichten in der App zur Verfügung gestellt.</span><span class="sxs-lookup"><span data-stu-id="1061c-139">This will make it available to all of the views in the app.</span></span>

<span data-ttu-id="1061c-140">Fügen Sie den \*\*\*\* folgenden nach `app.use(passport.session());` der Leitung hinzu.</span><span class="sxs-lookup"><span data-stu-id="1061c-140">Add the following **after** the `app.use(passport.session());` line.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="1061c-141">Speichern der Token</span><span class="sxs-lookup"><span data-stu-id="1061c-141">Storing the tokens</span></span>

<span data-ttu-id="1061c-142">Da Sie jetzt Tokens abrufen können, ist es an der Zeit, eine Möglichkeit zum Speichern in der APP zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="1061c-142">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="1061c-143">Derzeit speichert die APP das RAW-Zugriffstoken in der speicherresidenten Speicher der Benutzer.</span><span class="sxs-lookup"><span data-stu-id="1061c-143">Currently, the app is storing the raw access token in the in-memory user storage.</span></span> <span data-ttu-id="1061c-144">Da es sich um eine Beispiel-App handelt, werden Sie Sie auch weiterhin dort speichern.</span><span class="sxs-lookup"><span data-stu-id="1061c-144">Since this is a sample app, for simplicity's sake, you'll continue to store them there.</span></span> <span data-ttu-id="1061c-145">Eine echte APP würde eine zuverlässigere Secure Storage-Lösung wie eine Datenbank verwenden.</span><span class="sxs-lookup"><span data-stu-id="1061c-145">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="1061c-146">Das Speichern des Zugriffstokens ermöglicht jedoch nicht das Überprüfen des Ablaufs oder das Aktualisieren des Tokens.</span><span class="sxs-lookup"><span data-stu-id="1061c-146">However, storing just the access token doesn't allow you to check expiration or refresh the token.</span></span> <span data-ttu-id="1061c-147">Um dies zu ermöglichen, aktualisieren Sie das Beispiel, um die Token in einem `AccessToken` Objekt aus der `simple-oauth2` Bibliothek einzuschließen.</span><span class="sxs-lookup"><span data-stu-id="1061c-147">In order to enable that, update the sample to wrap the tokens in an `AccessToken` object from the `simple-oauth2` library.</span></span>

<span data-ttu-id="1061c-148">Fügen Sie zunächst `./app.js`in den folgenden Code **vor** der `signInComplete` Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="1061c-148">First, in `./app.js`, add the following code **before** the `signInComplete` function.</span></span>

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

<span data-ttu-id="1061c-149">Aktualisieren Sie dann die `signInComplete` -Funktion, um `AccessToken` eine aus den unformatierten Token zu erstellen, die übergeben werden, und speichern Sie diese im Benutzerspeicher.</span><span class="sxs-lookup"><span data-stu-id="1061c-149">Then, update the `signInComplete` function to create an `AccessToken` from the raw tokens passed in and store that in the user storage.</span></span> <span data-ttu-id="1061c-150">Ersetzen Sie die vorhandene `signInComplete`-Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="1061c-150">Replace the existing `signInComplete` function with the following.</span></span>

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

<span data-ttu-id="1061c-151">Aktualisieren Sie `callback` die Route `./routes/auth.js` in, um `req.flash` die Linie mit dem Zugriffstoken zu entfernen.</span><span class="sxs-lookup"><span data-stu-id="1061c-151">Update the `callback` route in `./routes/auth.js` to remove the `req.flash` line with the access token.</span></span> <span data-ttu-id="1061c-152">Die `callback` Route sollte wie folgt aussehen.</span><span class="sxs-lookup"><span data-stu-id="1061c-152">The `callback` route should look like the following.</span></span>

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

<span data-ttu-id="1061c-153">Starten Sie den Server neu, und führen Sie den Anmeldevorgang durch.</span><span class="sxs-lookup"><span data-stu-id="1061c-153">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="1061c-154">Sie sollten auf der Startseite zurückkehren, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="1061c-154">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Screenshot der Startseite nach der Anmeldung](./images/add-aad-auth-01.png)

<span data-ttu-id="1061c-156">Klicken Sie auf den Benutzer Avatar in der oberen rechten Ecke, \*\*\*\* um auf den Link abmelden zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="1061c-156">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="1061c-157">Wenn \*\*\*\* Sie auf Abmelden klicken, wird die Sitzung zurückgesetzt, und Sie kehren zur Startseite.</span><span class="sxs-lookup"><span data-stu-id="1061c-157">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Screenshot des Dropdownmenüs mit dem Link "abMelden"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="1061c-159">Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="1061c-159">Refreshing tokens</span></span>

<span data-ttu-id="1061c-160">Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="1061c-160">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="1061c-161">Dies ist das Token, mit dem die APP auf Microsoft Graph im Namen des Benutzers zugreifen kann.</span><span class="sxs-lookup"><span data-stu-id="1061c-161">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="1061c-162">Dieses Token ist jedoch kurzlebig.</span><span class="sxs-lookup"><span data-stu-id="1061c-162">However, this token is short-lived.</span></span> <span data-ttu-id="1061c-163">Das Token läuft eine Stunde nach der Ausgabe ab.</span><span class="sxs-lookup"><span data-stu-id="1061c-163">The token expires an hour after it is issued.</span></span> <span data-ttu-id="1061c-164">An dieser Stelle wird das Aktualisierungstoken nützlich.</span><span class="sxs-lookup"><span data-stu-id="1061c-164">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="1061c-165">Das Aktualisierungstoken ermöglicht der APP, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="1061c-165">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="1061c-166">Um dies zu verwalten, erstellen Sie eine neue Datei im Stamm des Projekts `tokens.js` , in dem die Tokenverwaltung-Funktionen aufbewahrt werden sollen.</span><span class="sxs-lookup"><span data-stu-id="1061c-166">To manage this, create a new file in the root of the project named `tokens.js` to hold token management functions.</span></span> <span data-ttu-id="1061c-167">Fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="1061c-167">Add the following code.</span></span>

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

<span data-ttu-id="1061c-168">Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen ist oder in Kürze abläuft.</span><span class="sxs-lookup"><span data-stu-id="1061c-168">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="1061c-169">Wenn dies der Fall ist, wird das Aktualisierungstoken zum Abrufen neuer Token verwendet, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="1061c-169">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span> <span data-ttu-id="1061c-170">Sie verwenden diese Methode immer dann, wenn das Zugriffstoken nicht mehr gespeichert werden muss.</span><span class="sxs-lookup"><span data-stu-id="1061c-170">You'll use this method whenever you need to get the access token out of storage.</span></span>