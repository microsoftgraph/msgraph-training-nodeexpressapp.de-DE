<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="4e9ac-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="4e9ac-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="4e9ac-103">In diesem Schritt werden Sie die [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) -Bibliothek in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-103">In this step you will integrate the [passport-azure-ad](https://github.com/AzureAD/passport-azure-ad) library into the application.</span></span>

1. <span data-ttu-id="4e9ac-104">Erstellen Sie im Stammverzeichnis `.env` der Anwendung eine neue Datei mit dem Namen file, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-104">Create a new file named `.env` file in the root of your application, and add the following code.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/.env.example":::

    <span data-ttu-id="4e9ac-105">Ersetzen `YOUR APP ID HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, `YOUR APP SECRET HERE` und ersetzen Sie durch das Kennwort, das Sie generiert haben.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-105">Replace `YOUR APP ID HERE` with the application ID from the Application Registration Portal, and replace `YOUR APP SECRET HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="4e9ac-106">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die Datei `.env` aus der Quellcodeverwaltung auszuschließen, damit versehentlich keine APP-ID und Ihr Kennwort verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="4e9ac-107">Öffnen `./app.js` Sie und fügen Sie die folgende Codezeile am Anfang der Datei hinzu, um `.env` die Datei zu laden.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-107">Open `./app.js` and add the following line to the top of the file to load the `.env` file.</span></span>

    ```javascript
    require('dotenv').config();
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="4e9ac-108">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="4e9ac-108">Implement sign-in</span></span>

1. <span data-ttu-id="4e9ac-109">Suchen Sie die `var indexRouter = require('./routes/index');` - `./app.js`Position in.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-109">Locate the line `var indexRouter = require('./routes/index');` in `./app.js`.</span></span> <span data-ttu-id="4e9ac-110">Fügen Sie den folgenden Code **vor** dieser Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-110">Insert the following code **before** that line.</span></span>

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

    <span data-ttu-id="4e9ac-111">Dieser Code initialisiert die [Passport. js](http://www.passportjs.org/) -Bibliothek für die `passport-azure-ad` Verwendung der Bibliothek und konfiguriert Sie mit der APP-ID und dem Kennwort für die app.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-111">This code initializes the [Passport.js](http://www.passportjs.org/) library to use the `passport-azure-ad` library, and configures it with the app ID and password for the app.</span></span>

1. <span data-ttu-id="4e9ac-112">Suchen Sie die `app.use('/', indexRouter);` - `./app.js`Position in.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-112">Locate the line `app.use('/', indexRouter);` in `./app.js`.</span></span> <span data-ttu-id="4e9ac-113">Fügen Sie den folgenden Code **vor** dieser Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-113">Insert the following code **before** that line.</span></span>

    ```javascript
    // Initialize passport
    app.use(passport.initialize());
    app.use(passport.session());
    ```

1. <span data-ttu-id="4e9ac-114">Erstellen Sie eine neue Datei im `./routes` Verzeichnis mit `auth.js` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-114">Create a new file in the `./routes` directory named `auth.js` and add the following code.</span></span>

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

    <span data-ttu-id="4e9ac-115">Dadurch wird ein Router mit drei Routen definiert `signin`: `callback`, und `signout`.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-115">This defines a router with three routes: `signin`, `callback`, and `signout`.</span></span>

    <span data-ttu-id="4e9ac-116">Die `signin` Route Ruft die `passport.authenticate` Methode auf, wodurch die APP an die Azure-Anmeldeseite umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-116">The `signin` route calls the `passport.authenticate` method, causing the app to redirect to the Azure login page.</span></span>

    <span data-ttu-id="4e9ac-117">Auf `callback` der Route wird nach Abschluss der Anmeldung Azure umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-117">The `callback` route is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="4e9ac-118">Der Code Ruft die `passport.authenticate` Methode erneut auf, wodurch `passport-azure-ad` die Strategie ein Zugriffstoken anfordert.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-118">The code calls the `passport.authenticate` method again, causing the `passport-azure-ad` strategy to request an access token.</span></span> <span data-ttu-id="4e9ac-119">Sobald das Token abgerufen wurde, wird der nächste Handler aufgerufen, der zurück zur Startseite mit dem Zugriffstoken im temporären Fehlerwert umgeleitet wird.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-119">Once the token is obtained, the next handler is called, which redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="4e9ac-120">Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-120">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="4e9ac-121">Bevor wir testen, müssen wir die Express-App so konfigurieren, dass Sie den neuen `./routes/auth.js`Router verwendet.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-121">Before we test, we need to configure the Express app to use the new router from `./routes/auth.js`.</span></span>

    <span data-ttu-id="4e9ac-122">Die `signout` -Methode protokolliert den Benutzer und zerstört die Sitzung.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-122">The `signout` method logs the user out and destroys the session.</span></span>

1. <span data-ttu-id="4e9ac-123">Öffnen `./app.js` Sie den folgenden Code, und fügen `var app = express();` Sie ihn **vor** die Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-123">Open `./app.js` and insert the following code **before** the `var app = express();` line.</span></span>

    ```javascript
    var authRouter = require('./routes/auth');
    ```

1. <span data-ttu-id="4e9ac-124">Fügen Sie den folgenden **after** Code nach `app.use('/', indexRouter);` der Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-124">Insert the following code **after** the `app.use('/', indexRouter);` line.</span></span>

    ```javascript
    app.use('/auth', authRouter);
    ```

<span data-ttu-id="4e9ac-125">Starten Sie den Server, und `https://localhost:3000`navigieren Sie zu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-125">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="4e9ac-126">Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-126">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="4e9ac-127">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-127">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="4e9ac-128">Der Browser leitet zur App um, in der Sie das Token sehen.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-128">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="4e9ac-129">Benutzerdetails abrufen</span><span class="sxs-lookup"><span data-stu-id="4e9ac-129">Get user details</span></span>

1. <span data-ttu-id="4e9ac-130">Erstellen Sie eine neue Datei im Stamm des Projekts mit dem `graph.js` Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-130">Create a new file in the root of the project named `graph.js` and add the following code.</span></span>

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

    <span data-ttu-id="4e9ac-131">Dadurch wird die `getUserDetails` Funktion exportiert, die das Microsoft Graph-SDK verwendet, `/me` um den Endpunkt aufzurufen und das Ergebnis zurückzugeben.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-131">This exports the `getUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="4e9ac-132">Öffnen `/app.js` Sie und fügen Sie `require` die folgenden Anweisungen am Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-132">Open `/app.js` and add the following `require` statements to the top of the file.</span></span>

    ```javascript
    var graph = require('./graph');
    ```

1. <span data-ttu-id="4e9ac-133">Ersetzen Sie die vorhandene `signInComplete`-Funktion durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-133">Replace the existing `signInComplete` function with the following code.</span></span>

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

    <span data-ttu-id="4e9ac-134">Der neue Code aktualisiert den `profile` von Passport bereitgestellten `email` , um eine Eigenschaft mithilfe der von Microsoft Graph zurückgegebenen Daten hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-134">The new code updates the `profile` provided by Passport to add an `email` property, using the data returned by Microsoft Graph.</span></span>

1. <span data-ttu-id="4e9ac-135">Fügen Sie Folgendes **nach** der `app.use(passport.session());` -Verbindung hinzu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-135">Add the following **after** the `app.use(passport.session());` line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="AddProfileSnippet":::

    <span data-ttu-id="4e9ac-136">Durch diesen Code wird das Benutzerprofil in `locals` die Eigenschaft der Antwort geladen.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-136">This code loads the user profile into the `locals` property of the response.</span></span> <span data-ttu-id="4e9ac-137">Dadurch wird es für alle Ansichten in der app verfügbar gemacht.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-137">This will make it available to all of the views in the app.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="4e9ac-138">Speichern des Tokens</span><span class="sxs-lookup"><span data-stu-id="4e9ac-138">Storing the tokens</span></span>

<span data-ttu-id="4e9ac-139">Nun, da Sie Token abrufen können, ist es an der Zeit, eine Möglichkeit einzurichten, diese in der App zu speichern.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-139">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="4e9ac-140">Derzeit speichert die APP das unformatierte Zugriffstoken im Speicher des speicherresidenten Benutzers.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-140">Currently, the app is storing the raw access token in the in-memory user storage.</span></span> <span data-ttu-id="4e9ac-141">Da es sich um eine Beispiel-App handelt, werden Sie Sie aus Gründen der Einfachheit weiterhin dort speichern.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-141">Since this is a sample app, for simplicity's sake, you'll continue to store them there.</span></span> <span data-ttu-id="4e9ac-142">Eine echte App würde eine zuverlässigere sichere Speicherlösung wie eine Datenbank verwenden.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-142">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="4e9ac-143">Wenn Sie jedoch nur das Zugriffstoken speichern, können Sie das Ablaufdatum überprüfen oder das Token aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-143">However, storing just the access token doesn't allow you to check expiration or refresh the token.</span></span> <span data-ttu-id="4e9ac-144">Um dies zu ermöglichen, aktualisieren Sie das Beispiel so, dass die Token in `AccessToken` einem Objekt aus `simple-oauth2` der Bibliothek umbrochen werden.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-144">In order to enable that, update the sample to wrap the tokens in an `AccessToken` object from the `simple-oauth2` library.</span></span>

1. <span data-ttu-id="4e9ac-145">Öffnen `./app.js` Sie den folgenden Code, und fügen `signInComplete` Sie ihn **vor** der Funktion hinzu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-145">Open `./app.js` and add the following code **before** the `signInComplete` function.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="ConfigureOAuth2Snippet":::

1. <span data-ttu-id="4e9ac-146">Ersetzen Sie die vorhandene `signInComplete`-Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-146">Replace the existing `signInComplete` function with the following.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="SignInCompleteSnippet" highlight="17-18, 21":::

1. <span data-ttu-id="4e9ac-147">Ersetzen Sie die vorhandene Rückruf Route `./routes/auth.js` in durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-147">Replace the existing callback route in  `./routes/auth.js` with the following.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/auth.js" id="CallbackRouteSnippet" highlight="17-18":::

1. <span data-ttu-id="4e9ac-148">Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-148">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="4e9ac-149">Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-149">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. <span data-ttu-id="4e9ac-151">Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-151">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="4e9ac-152">Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-152">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="4e9ac-154">Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="4e9ac-154">Refreshing tokens</span></span>

<span data-ttu-id="4e9ac-155">Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-155">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="4e9ac-156">Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-156">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="4e9ac-157">Dieses Token ist jedoch nur kurzzeitig verfügbar.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-157">However, this token is short-lived.</span></span> <span data-ttu-id="4e9ac-158">Das Token läuft eine Stunde nach seiner Ausgabe ab.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-158">The token expires an hour after it is issued.</span></span> <span data-ttu-id="4e9ac-159">An dieser Stelle kommt das Aktualisierungstoken ins Spiel.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-159">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="4e9ac-160">Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-160">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

1. <span data-ttu-id="4e9ac-161">Erstellen `tokens.js` Sie eine neue Datei im Stamm des Projekts, in dem die Token-Verwaltungsfunktionen gespeichert sind.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-161">Create a new file in the root of the project named `tokens.js` to hold token management functions.</span></span> <span data-ttu-id="4e9ac-162">Fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-162">Add the following code.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/tokens.js" id="TokensSnippet":::

<span data-ttu-id="4e9ac-163">Diese Methode überprüft zunächst, ob das Zugriffstoken abgelaufen oder nahezu abläuft.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-163">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="4e9ac-164">Wenn dies der Fall ist, wird das Aktualisierungstoken verwendet, um neue Token abzurufen, dann wird der Cache aktualisiert und das neue Zugriffstoken zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-164">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span> <span data-ttu-id="4e9ac-165">Sie verwenden diese Methode immer dann, wenn Sie das Zugriffstoken aus dem Speicher holen müssen.</span><span class="sxs-lookup"><span data-stu-id="4e9ac-165">You'll use this method whenever you need to get the access token out of storage.</span></span>
