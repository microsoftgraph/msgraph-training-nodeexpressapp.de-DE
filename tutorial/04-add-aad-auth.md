<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="e51ac-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="e51ac-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="e51ac-102">Dies ist erforderlich, um das erforderliche OAuth-Zugriffstoken zum Aufrufen von Microsoft Graph zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="e51ac-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="e51ac-103">In diesem Schritt werden Sie die [msal-Node-](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) Bibliothek in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="e51ac-103">In this step you will integrate the [msal-node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) library into the application.</span></span>

1. <span data-ttu-id="e51ac-104">Erstellen Sie eine neue Datei mit dem Namen " **. env** " im Stammverzeichnis Ihrer Anwendung, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="e51ac-104">Create a new file named **.env** in the root of your application, and add the following code.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/example.env":::

    <span data-ttu-id="e51ac-105">Ersetzen `YOUR_CLIENT_SECRET_HERE` Sie durch die Anwendungs-ID aus dem Anwendungs Registrierungs Portal, und ersetzen `YOUR_APP_SECRET_HERE` Sie durch den von Ihnen generierten Client Schlüssel.</span><span class="sxs-lookup"><span data-stu-id="e51ac-105">Replace `YOUR_CLIENT_SECRET_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the client secret you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="e51ac-106">Wenn Sie die Quellcodeverwaltung wie git verwenden, wäre es jetzt ein guter Zeitpunkt, die **. env** -Datei aus der Quellcodeverwaltung auszuschließen, damit versehentlich Ihre APP-ID und Ihr Kennwort verloren gehen.</span><span class="sxs-lookup"><span data-stu-id="e51ac-106">If you're using source control such as git, now would be a good time to exclude the **.env** file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="e51ac-107">Öffnen Sie **./app.js** , und fügen Sie die folgende Codezeile zum Anfang der Datei hinzu, um die **env** -Datei zu laden.</span><span class="sxs-lookup"><span data-stu-id="e51ac-107">Open **./app.js** and add the following line to the top of the file to load the **.env** file.</span></span>

    ```javascript
    require('dotenv').config();
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="e51ac-108">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="e51ac-108">Implement sign-in</span></span>

1. <span data-ttu-id="e51ac-109">Suchen Sie die-Position `var indexRouter = require('./routes/index');` in **./#b0**.</span><span class="sxs-lookup"><span data-stu-id="e51ac-109">Locate the line `var indexRouter = require('./routes/index');` in **./app.js**.</span></span> <span data-ttu-id="e51ac-110">Fügen Sie den folgenden Code **vor** dieser Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="e51ac-110">Insert the following code **before** that line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="MsalInitSnippet":::

    <span data-ttu-id="e51ac-111">Mit diesem Code wird die msal-Knoten Bibliothek mit der APP-ID und dem Kennwort für die APP initialisiert.</span><span class="sxs-lookup"><span data-stu-id="e51ac-111">This code initializes the msal-node library with the app ID and password for the app.</span></span>

1. <span data-ttu-id="e51ac-112">Erstellen Sie eine neue Datei im **./routes** -Verzeichnis mit dem Namen **auth.js** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="e51ac-112">Create a new file in the **./routes** directory named **auth.js** and add the following code.</span></span>

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

    <span data-ttu-id="e51ac-113">Dadurch wird ein Router mit drei Routen definiert: `signin` , `callback` und `signout` .</span><span class="sxs-lookup"><span data-stu-id="e51ac-113">This defines a router with three routes: `signin`, `callback`, and `signout`.</span></span>

    <span data-ttu-id="e51ac-114">Die `signin` Route Ruft die `getAuthCodeUrl` Funktion auf, um die Anmelde-URL zu generieren, und leitet dann den Browser an diese URL um.</span><span class="sxs-lookup"><span data-stu-id="e51ac-114">The `signin` route calls the `getAuthCodeUrl` function to generate the login URL, then redirects the browser to that URL.</span></span>

    <span data-ttu-id="e51ac-115">Auf der `callback` Route wird nach Abschluss der Anmeldung Azure umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="e51ac-115">The `callback` route is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="e51ac-116">Der Code Ruft die `acquireTokenByCode` Funktion auf, um den Autorisierungscode für ein Zugriffstoken auszutauschen.</span><span class="sxs-lookup"><span data-stu-id="e51ac-116">The code calls the `acquireTokenByCode` function to exchange the authorization code for an access token.</span></span> <span data-ttu-id="e51ac-117">Sobald das Token abgerufen wurde, wird es an die Startseite mit dem Zugriffstoken im temporären Fehlerwert zurückgeleitet.</span><span class="sxs-lookup"><span data-stu-id="e51ac-117">Once the token is obtained, it redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="e51ac-118">Wir verwenden dies, um zu überprüfen, ob unsere Anmeldung funktionsfähig ist, bevor Sie fortfahren.</span><span class="sxs-lookup"><span data-stu-id="e51ac-118">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="e51ac-119">Bevor wir testen, müssen wir die Express-App so konfigurieren, dass Sie den neuen Router aus **./routes/auth.js** verwendet.</span><span class="sxs-lookup"><span data-stu-id="e51ac-119">Before we test, we need to configure the Express app to use the new router from **./routes/auth.js**.</span></span>

    <span data-ttu-id="e51ac-120">Die `signout` -Methode protokolliert den Benutzer und zerstört die Sitzung.</span><span class="sxs-lookup"><span data-stu-id="e51ac-120">The `signout` method logs the user out and destroys the session.</span></span>

1. <span data-ttu-id="e51ac-121">Öffnen Sie **./app.js** , und fügen Sie den folgenden Code **vor** die- `var app = express();` Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="e51ac-121">Open **./app.js** and insert the following code **before** the `var app = express();` line.</span></span>

    ```javascript
    var authRouter = require('./routes/auth');
    ```

1. <span data-ttu-id="e51ac-122">Fügen Sie den folgenden Code **nach** der `app.use('/', indexRouter);` Codezeile ein.</span><span class="sxs-lookup"><span data-stu-id="e51ac-122">Insert the following code **after** the `app.use('/', indexRouter);` line.</span></span>

    ```javascript
    app.use('/auth', authRouter);
    ```

<span data-ttu-id="e51ac-123">Starten Sie den Server, und navigieren Sie zu `https://localhost:3000` .</span><span class="sxs-lookup"><span data-stu-id="e51ac-123">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="e51ac-124">Klicken Sie auf die Schaltfläche zum Anmelden, um zu `https://login.microsoftonline.com`weitergeleitet zu werden.</span><span class="sxs-lookup"><span data-stu-id="e51ac-124">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="e51ac-125">Melden Sie sich mit Ihrem Microsoft-Konto an, und stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="e51ac-125">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="e51ac-126">Der Browser leitet zur App um, in der Sie das Token sehen.</span><span class="sxs-lookup"><span data-stu-id="e51ac-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="e51ac-127">Benutzerdetails abrufen</span><span class="sxs-lookup"><span data-stu-id="e51ac-127">Get user details</span></span>

1. <span data-ttu-id="e51ac-128">Erstellen Sie eine neue Datei im Stamm des Projekts mit dem Namen **graph.js** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="e51ac-128">Create a new file in the root of the project named **graph.js** and add the following code.</span></span>

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

    <span data-ttu-id="e51ac-129">Dadurch wird die `getUserDetails` Funktion exportiert, die das Microsoft Graph-SDK verwendet, um den Endpunkt aufzurufen `/me` und das Ergebnis zurückzugeben.</span><span class="sxs-lookup"><span data-stu-id="e51ac-129">This exports the `getUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="e51ac-130">Öffnen Sie **./routes/auth.js** , und fügen Sie die folgenden `require` Anweisungen am Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="e51ac-130">Open **./routes/auth.js** and add the following `require` statements to the top of the file.</span></span>

    ```javascript
    var graph = require('../graph');
    ```

1. <span data-ttu-id="e51ac-131">Ersetzen Sie die vorhandene Rückruf Route durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="e51ac-131">Replace the existing callback route with the following code.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/auth.js" id="CallbackSnippet" highlight="13-23":::

    <span data-ttu-id="e51ac-132">Der neue Code speichert die Konto-ID des Benutzers in der Sitzung, ruft die Details des Benutzers aus Microsoft Graph ab und speichert ihn im Benutzerspeicher der app.</span><span class="sxs-lookup"><span data-stu-id="e51ac-132">The new code saves the user's account ID in the session, gets the user's details from Microsoft Graph, and saves it in the app's user storage.</span></span>

1. <span data-ttu-id="e51ac-133">Starten Sie den Server neu, und fahren Sie mit dem Anmeldevorgang fort.</span><span class="sxs-lookup"><span data-stu-id="e51ac-133">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="e51ac-134">Sie sollten wieder auf der Startseite enden, aber die Benutzeroberfläche sollte sich ändern, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="e51ac-134">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. <span data-ttu-id="e51ac-136">Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den **Abmelde** Link zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="e51ac-136">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="e51ac-137">Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="e51ac-137">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="e51ac-139">Speichern und Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="e51ac-139">Storing and refreshing tokens</span></span>

<span data-ttu-id="e51ac-140">Zu diesem Zeitpunkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="e51ac-140">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="e51ac-141">Dies ist das Token, das es der App ermöglicht, im Namen des Benutzers auf Microsoft Graph zuzugreifen.</span><span class="sxs-lookup"><span data-stu-id="e51ac-141">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="e51ac-142">Dieses Token ist jedoch nur kurzzeitig verfügbar.</span><span class="sxs-lookup"><span data-stu-id="e51ac-142">However, this token is short-lived.</span></span> <span data-ttu-id="e51ac-143">Das Token läuft eine Stunde nach seiner Ausgabe ab.</span><span class="sxs-lookup"><span data-stu-id="e51ac-143">The token expires an hour after it is issued.</span></span> <span data-ttu-id="e51ac-144">An dieser Stelle kommt das Aktualisierungstoken ins Spiel.</span><span class="sxs-lookup"><span data-stu-id="e51ac-144">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="e51ac-145">Mit der OAuth-Spezifikation wird ein Aktualisierungstoken eingeführt, mit dem die APP ein neues Zugriffstoken anfordern kann, ohne dass sich der Benutzer erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="e51ac-145">The OAuth specification introduces a refresh token, which allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="e51ac-146">Da die APP das msal-Node-Paket verwendet, müssen Sie keine Token-Speicher-oder Aktualisierungslogik implementieren.</span><span class="sxs-lookup"><span data-stu-id="e51ac-146">Because the app is using the msal-node package, you do not need to implement any token storage or refresh logic.</span></span> <span data-ttu-id="e51ac-147">Die APP verwendet den standardmäßigen msal-Token-Cache im Arbeitsspeicher, der für eine Beispielanwendung ausreicht.</span><span class="sxs-lookup"><span data-stu-id="e51ac-147">The app uses the default msal-node in-memory token cache, which is sufficient for a sample application.</span></span> <span data-ttu-id="e51ac-148">Produktionsanwendungen sollten ein eigenes [Cache-Plugin](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md) bereitstellen, um den Token-Cache in einem sicheren, zuverlässigen Speichermedium zu serialisieren.</span><span class="sxs-lookup"><span data-stu-id="e51ac-148">Production applications should provide their own [caching plugin](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md) to serialize the token cache in a secure, reliable storage medium.</span></span>
