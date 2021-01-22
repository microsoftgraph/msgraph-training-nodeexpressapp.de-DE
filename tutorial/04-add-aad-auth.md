<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="cee48-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="cee48-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="cee48-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph abzurufen.</span><span class="sxs-lookup"><span data-stu-id="cee48-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="cee48-103">In diesem Schritt integrieren Sie die [msal-node-Bibliothek](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) in die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="cee48-103">In this step you will integrate the [msal-node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) library into the application.</span></span>

1. <span data-ttu-id="cee48-104">Erstellen Sie eine neue Datei mit dem Namen **".env"** im Stammverzeichnis Ihrer Anwendung, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="cee48-104">Create a new file named **.env** in the root of your application, and add the following code.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/example.env":::

    <span data-ttu-id="cee48-105">Ersetzen `YOUR_CLIENT_SECRET_HERE` Sie sie durch die Anwendungs-ID aus dem Anwendungsregistrierungsportal, und ersetzen Sie sie durch den von Ihnen `YOUR_APP_SECRET_HERE` generierten geheimen Client-Schlüssel.</span><span class="sxs-lookup"><span data-stu-id="cee48-105">Replace `YOUR_CLIENT_SECRET_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the client secret you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="cee48-106">Wenn Sie quellcodeverwaltung wie Git verwenden, wäre es jetzt ein guter Zeitpunkt, die **env-Datei** aus der Quellcodeverwaltung auszuschließen, um zu vermeiden, dass Versehentliches Lecks für Ihre App-ID und Ihr Kennwort erforderlich sind.</span><span class="sxs-lookup"><span data-stu-id="cee48-106">If you're using source control such as git, now would be a good time to exclude the **.env** file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="cee48-107">Öffnen **Sie ./app.js,** und fügen Sie die folgende Zeile am oberen Rand der Datei hinzu, um die **.env-Datei zu** laden.</span><span class="sxs-lookup"><span data-stu-id="cee48-107">Open **./app.js** and add the following line to the top of the file to load the **.env** file.</span></span>

    ```javascript
    require('dotenv').config();
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="cee48-108">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="cee48-108">Implement sign-in</span></span>

1. <span data-ttu-id="cee48-109">Suchen Sie die `var app = express();` Zeile in **./app.js**.</span><span class="sxs-lookup"><span data-stu-id="cee48-109">Locate the line `var app = express();` in **./app.js**.</span></span> <span data-ttu-id="cee48-110">Fügen Sie nach dieser **Zeile den folgenden** Code ein.</span><span class="sxs-lookup"><span data-stu-id="cee48-110">Insert the following code **after** that line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="MsalInitSnippet":::

    <span data-ttu-id="cee48-111">Dieser Code initialisiert die msal-node-Bibliothek mit der App-ID und dem Kennwort für die App.</span><span class="sxs-lookup"><span data-stu-id="cee48-111">This code initializes the msal-node library with the app ID and password for the app.</span></span>

1. <span data-ttu-id="cee48-112">Erstellen Sie eine neue Datei im Verzeichnis **./routes** mit **auth.js,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="cee48-112">Create a new file in the **./routes** directory named **auth.js** and add the following code.</span></span>

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

    <span data-ttu-id="cee48-113">Dadurch wird ein Router mit drei Routen definiert: `signin` `callback` , und `signout` .</span><span class="sxs-lookup"><span data-stu-id="cee48-113">This defines a router with three routes: `signin`, `callback`, and `signout`.</span></span>

    <span data-ttu-id="cee48-114">Die `signin` Route ruft die Funktion `getAuthCodeUrl` auf, um die Anmelde-URL zu generieren, und leitet dann den Browser an diese URL um.</span><span class="sxs-lookup"><span data-stu-id="cee48-114">The `signin` route calls the `getAuthCodeUrl` function to generate the login URL, then redirects the browser to that URL.</span></span>

    <span data-ttu-id="cee48-115">Die `callback` Route leitet Azure um, nachdem die Anmeldung abgeschlossen ist.</span><span class="sxs-lookup"><span data-stu-id="cee48-115">The `callback` route is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="cee48-116">Der Code ruft die `acquireTokenByCode` Funktion auf, um den Autorisierungscode gegen ein Zugriffstoken zu tauschen.</span><span class="sxs-lookup"><span data-stu-id="cee48-116">The code calls the `acquireTokenByCode` function to exchange the authorization code for an access token.</span></span> <span data-ttu-id="cee48-117">Nachdem das Token erhalten wurde, wird es mit dem Zugriffstoken im temporären Fehlerwert zurück zur Startseite umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="cee48-117">Once the token is obtained, it redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="cee48-118">Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktioniert, bevor wir weiter arbeiten.</span><span class="sxs-lookup"><span data-stu-id="cee48-118">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="cee48-119">Vor dem Testen müssen wir die Express-App für die Verwendung des neuen Routers aus **./routes/auth.js**.</span><span class="sxs-lookup"><span data-stu-id="cee48-119">Before we test, we need to configure the Express app to use the new router from **./routes/auth.js**.</span></span>

    <span data-ttu-id="cee48-120">Die `signout` Methode meldet den Benutzer ab und zerstört die Sitzung.</span><span class="sxs-lookup"><span data-stu-id="cee48-120">The `signout` method logs the user out and destroys the session.</span></span>

1. <span data-ttu-id="cee48-121">Öffnen **Sie ./app.js,** und fügen Sie den folgenden Code **vor der** Zeile `var app = express();` ein.</span><span class="sxs-lookup"><span data-stu-id="cee48-121">Open **./app.js** and insert the following code **before** the `var app = express();` line.</span></span>

    ```javascript
    var authRouter = require('./routes/auth');
    ```

1. <span data-ttu-id="cee48-122">Fügen Sie den folgenden Code **nach der** `app.use('/', indexRouter);` Zeile ein.</span><span class="sxs-lookup"><span data-stu-id="cee48-122">Insert the following code **after** the `app.use('/', indexRouter);` line.</span></span>

    ```javascript
    app.use('/auth', authRouter);
    ```

<span data-ttu-id="cee48-123">Starten Sie den Server, und navigieren Sie zu `https://localhost:3000` .</span><span class="sxs-lookup"><span data-stu-id="cee48-123">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="cee48-124">Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden.</span><span class="sxs-lookup"><span data-stu-id="cee48-124">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="cee48-125">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="cee48-125">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="cee48-126">Der Browser leitet zur App um, in der Sie das Token sehen.</span><span class="sxs-lookup"><span data-stu-id="cee48-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="cee48-127">Benutzerdetails abrufen</span><span class="sxs-lookup"><span data-stu-id="cee48-127">Get user details</span></span>

1. <span data-ttu-id="cee48-128">Erstellen Sie eine neue Datei im Stammverzeichnis des Projekts namens **graph.js,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="cee48-128">Create a new file in the root of the project named **graph.js** and add the following code.</span></span>

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

    <span data-ttu-id="cee48-129">Dadurch wird die Funktion exportiert, die das Microsoft Graph SDK zum Aufrufen des Endpunkts und `getUserDetails` `/me` zum Zurückgeben des Ergebnisses verwendet.</span><span class="sxs-lookup"><span data-stu-id="cee48-129">This exports the `getUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="cee48-130">Öffnen **Sie ./routes/auth.js,** und fügen Sie die folgenden Anweisungen am Oberen Rand `require` der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="cee48-130">Open **./routes/auth.js** and add the following `require` statements to the top of the file.</span></span>

    ```javascript
    var graph = require('../graph');
    ```

1. <span data-ttu-id="cee48-131">Ersetzen Sie die vorhandene Rückrufroute durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="cee48-131">Replace the existing callback route with the following code.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/auth.js" id="CallbackSnippet" highlight="13-23":::

    <span data-ttu-id="cee48-132">Der neue Code speichert die Konto-ID des Benutzers in der Sitzung, ruft die Details des Benutzers aus Microsoft Graph ab und speichert sie im Benutzerspeicher der App.</span><span class="sxs-lookup"><span data-stu-id="cee48-132">The new code saves the user's account ID in the session, gets the user's details from Microsoft Graph, and saves it in the app's user storage.</span></span>

1. <span data-ttu-id="cee48-133">Starten Sie den Server neu, und gehen Sie den Anmeldevorgang durch.</span><span class="sxs-lookup"><span data-stu-id="cee48-133">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="cee48-134">Sie sollten wieder auf der Startseite landen, aber die Benutzeroberfläche sollte geändert werden, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="cee48-134">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. <span data-ttu-id="cee48-136">Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den Link **"Abmelden" zu** klicken.</span><span class="sxs-lookup"><span data-stu-id="cee48-136">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="cee48-137">Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="cee48-137">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="cee48-139">Speichern und Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="cee48-139">Storing and refreshing tokens</span></span>

<span data-ttu-id="cee48-140">An diesem Punkt verfügt Ihre Anwendung über ein Zugriffstoken, das im `Authorization` Header von API-Aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="cee48-140">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="cee48-141">Dies ist das Token, mit dem die App im Namen des Benutzers auf Microsoft Graph zugreifen kann.</span><span class="sxs-lookup"><span data-stu-id="cee48-141">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="cee48-142">Dieses Token ist jedoch nur kurzzeitig verfügbar.</span><span class="sxs-lookup"><span data-stu-id="cee48-142">However, this token is short-lived.</span></span> <span data-ttu-id="cee48-143">Das Token läuft eine Stunde nach seiner Entsprechung ab.</span><span class="sxs-lookup"><span data-stu-id="cee48-143">The token expires an hour after it is issued.</span></span> <span data-ttu-id="cee48-144">An dieser Stelle kommt das Aktualisierungstoken ins Spiel.</span><span class="sxs-lookup"><span data-stu-id="cee48-144">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="cee48-145">Die OAuth-Spezifikation führt ein Aktualisierungstoken ein, mit dem die App ein neues Zugriffstoken anfordern kann, ohne dass sich der Benutzer erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="cee48-145">The OAuth specification introduces a refresh token, which allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="cee48-146">Da die App das msal-node-Paket verwendet, müssen Sie keine Tokenspeicher- oder Aktualisierungslogik implementieren.</span><span class="sxs-lookup"><span data-stu-id="cee48-146">Because the app is using the msal-node package, you do not need to implement any token storage or refresh logic.</span></span> <span data-ttu-id="cee48-147">Die App verwendet den standardmäßigen Speichertokencache "msal-node", der für eine Beispielanwendung ausreicht.</span><span class="sxs-lookup"><span data-stu-id="cee48-147">The app uses the default msal-node in-memory token cache, which is sufficient for a sample application.</span></span> <span data-ttu-id="cee48-148">Produktionsanwendungen sollten ihr eigenes Plug-In [zum](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md) Zwischenspeichern bereitstellen, um den Tokencache auf einem sicheren, zuverlässigen Speichermedium zu serialisieren.</span><span class="sxs-lookup"><span data-stu-id="cee48-148">Production applications should provide their own [caching plugin](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md) to serialize the token cache in a secure, reliable storage medium.</span></span>
