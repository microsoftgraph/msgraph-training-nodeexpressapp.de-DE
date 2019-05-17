<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung verwenden Sie [Express](http://expressjs.com/) , um eine Webanwendung zu erstellen. Wenn Sie den Express-Generator noch nicht installiert haben, können Sie ihn über die Befehlszeilenschnittstelle (CLI) mit dem folgenden Befehl installieren.

```Shell
npm install express-generator -g
```

Öffnen Sie die CLI, navigieren Sie zu einem Verzeichnis, in dem Sie über Berechtigungen zum Erstellen von Dateien verfügen, und führen Sie den folgenden Befehl aus, um eine neue Express-App zu erstellen, die als Rendering-Engine [Lenker](http://handlebarsjs.com/) verwendet.

```Shell
express --hbs graph-tutorial
```

Der Express-Generator erstellt ein neues Verzeichnis `graph-tutorial` mit dem Namen und ein Gerüst für eine Express-App. Navigieren Sie zu diesem neuen Verzeichnis, und geben Sie den folgenden Befehl ein, um Abhängigkeiten zu installieren.

```Shell
npm install
```

Sobald dieser Befehl abgeschlossen ist, verwenden Sie den folgenden Befehl, um einen lokalen Webserver zu starten.

```Shell
npm start
```

Öffnen Sie Ihren Browser und navigieren Sie zu `http://localhost:3000`. Wenn alles funktioniert, wird die Nachricht "Willkommen bei Express" angezeigt. Wenn diese Meldung nicht angezeigt wird, überprüfen Sie das [Express-Handbuch Erste Schritte](http://expressjs.com/starter/generator.html).

Bevor Sie fortfahren, sollten Sie einige zusätzliche Edelsteine installieren, die Sie später verwenden werden:

- [dotenv](https://github.com/motdotla/dotenv) zum Laden von Werten aus einer env-Datei.
- [](https://github.com/moment/moment/) Zeitpunkt für die Formatierung von Datum/Uhrzeit-Werten.
- [Connect-Flash](https://github.com/jaredhanson/connect-flash) zu Flash-Fehlermeldungen in der app.
- [Express-Sitzung](https://github.com/expressjs/session) zum Speichern von Werten in einer serverseitigen Sitzung im Arbeitsspeicher.
- [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad) zur Authentifizierung und zum Abrufen von Zugriffstoken.
- [Simple-oauth2](https://github.com/lelylan/simple-oauth2) für die Tokenverwaltung.
- [Microsoft-Graph-Client](https://github.com/microsoftgraph/msgraph-sdk-javascript) zum tätigen von Anrufen an Microsoft Graph.

Führen Sie den folgenden Befehl in der CLI aus.

```Shell
npm install dotenv@6.2.0 moment@2.24.0 connect-flash@0.1.1 express-session@1.15.6
npm install passport-azure-ad@4.0.0 simple-oauth2@2.2.1 @microsoft/microsoft-graph-client@1.5.2
```

> [!TIP]
> Windows-Benutzer erhalten möglicherweise die folgende Fehlermeldung, wenn Sie versuchen, diese Pakete unter Windows zu installieren.
>
> ```Shell
> gyp ERR! stack Error: Can't find Python executable "python", you can set the PYTHON env variable.
> ```
>
> Um den Fehler zu beheben, führen Sie den folgenden Befehl aus, um die Windows-Erstellungstools mithilfe eines erhöhten Terminalfensters (Administrator) zu installieren, das die vs-Erstellungstools und Python installiert.
>
> ```Shell
> npm install --global --production windows-build-tools
> ```

Aktualisieren Sie nun die Anwendung für die `connect-flash` Verwendung `express-session` der und Middleware. Öffnen Sie `./app.js` die Datei, und fügen `require` Sie die folgende Anweisung am Anfang der Datei hinzu.

```js
var session = require('express-session');
var flash = require('connect-flash');
```

Fügen Sie den folgenden Code unmittelbar nach `var app = express();` der Codezeile hinzu.

```js
// Session middleware
// NOTE: Uses default in-memory session store, which is not
// suitable for production
app.use(session({
  secret: 'your_secret_value_here',
  resave: false,
  saveUninitialized: false,
  unset: 'destroy'
}));

// Flash middleware
app.use(flash());

// Set up local vars for template layout
app.use(function(req, res, next) {
  // Read any flashed errors and save
  // in the response locals
  res.locals.error = req.flash('error_msg');

  // Check for simple error string and
  // convert to layout's expected format
  var errs = req.flash('error');
  for (var i in errs){
    res.locals.error.push({message: 'An error occurred', debug: errs[i]});
  }

  next();
});
```

## <a name="design-the-app"></a>Entwerfen der APP

Erstellen Sie zunächst das globale Layout für die app. Öffnen Sie `./views/layout.hbs` die Datei, und ersetzen Sie den gesamten Inhalt durch den folgenden Code.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Node.js Graph Tutorial</title>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <link rel='stylesheet' href='/stylesheets/style.css' />
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <a href="/" class="navbar-brand">Node.js Graph Tutorial</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <a href="/" class="nav-link{{#if active.home}} active{{/if}}">Home</a>
            </li>
            {{#if user}}
              <li class="nav-item" data-turbolinks="false">
                <a href="/calendar" class="nav-link{{#if active.calendar}} active{{/if}}">Calendar</a>
              </li>
            {{/if}}
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            {{#if user}}
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true"
                  aria-expanded="false">
                  {{#if user.avatar}}
                    <img src="{{ user.avatar }}" class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  {{else}}
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  {{/if}}
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0">{{ user.displayName }}</h5>
                  <p class="dropdown-item-text text-muted mb-0">{{ user.email }}</p>
                  <div class="dropdown-divider"></div>
                  <a href="/auth/signout" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            {{else}}
              <li class="nav-item">
                <a href="/auth/signin" class="nav-link">Sign In</a>
              </li>
            {{/if}}
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      {{#each error}}
        <div class="alert alert-danger" role="alert">
          <p class="mb-3">{{ this.message }}</p>
          {{#if this.debug }}
            <pre class="alert-pre border bg-light p-2"><code>{{ this.debug }}</code></pre>
          {{/if}}
        </div>
      {{/each}}

      {{{body}}}
    </main>
  </body>
</html>
```

Dieser Code fügt [Bootstrap](http://getbootstrap.com/) für einfaches Styling und [Font awesome](https://fontawesome.com/) für einige einfache Symbole hinzu. Außerdem wird ein globales Layout mit einer Navigationsleiste definiert.

Öffnen `./public/stylesheets/style.css` Sie nun und ersetzen Sie den gesamten Inhalt durch Folgendes.

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

Aktualisieren Sie nun die Standardseite. Öffnen Sie `./views/index.hbs` die Datei, und ersetzen Sie den Inhalt durch Folgendes.

```html
<div class="jumbotron">
  <h1>Node.js Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Node.js</p>
  {{#if user}}
    <h4>Welcome {{ user.displayName }}!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  {{else}}
    <a href="/auth/signin" class="btn btn-primary btn-large">Click here to sign in</a>
  {{/if}}
</div>
```

Öffnen Sie `./routes/index.js` die Datei, und ersetzen Sie den vorhandenen Code durch Folgendes.

```js
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  let params = {
    active: { home: true }
  };

  res.render('index', params);
});

module.exports = router;
```

Speichern Sie alle Änderungen, und starten Sie den Server neu. Nun sollte die APP sehr unterschiedlich aussehen.

![Screenshot der neu gestalteten Startseite](./images/create-app-01.png)