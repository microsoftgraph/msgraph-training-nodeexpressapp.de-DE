<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung integrieren Sie Microsoft Graph in die Anwendung. Für diese Anwendung verwenden Sie die [Microsoft-Graph-Client-](https://github.com/microsoftgraph/msgraph-sdk-javascript) Bibliothek, um Aufrufe von Microsoft Graph zu tätigen.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen aus Outlook

Beginnen Sie mit dem Hinzufügen einer neuen `./graph.js` Methode zur Datei, um die Ereignisse aus dem Kalender abzurufen. Fügen Sie die folgende Funktion innerhalb `module.exports` der `./graph.js`in hinzu.

```js
getEvents: async function(accessToken) {
  const client = getAuthenticatedClient(accessToken);

  const events = await client
    .api('/me/events')
    .select('subject,organizer,start,end')
    .orderby('createdDateTime DESC')
    .get();

  return events;
}
```

Überlegen Sie sich, was dieser Code tut.

- Die URL, die aufgerufen wird, `/me/events`lautet.
- Die `select` -Methode schränkt die für jedes Ereignis zurückgegebenen Felder auf nur die ein, die die Ansicht tatsächlich verwendet.
- Die `orderby` -Methode sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu denen Sie erstellt wurden, wobei das neueste Element zuerst angezeigt wird.

Erstellen Sie eine neue Datei im `./routes` Verzeichnis mit `calendar.js`dem Namen, und fügen Sie den folgenden Code hinzu.

```js
var express = require('express');
var router = express.Router();
var tokens = require('../tokens.js');
var graph = require('../graph.js');

/* GET /calendar */
router.get('/',
  async function(req, res) {
    if (!req.isAuthenticated()) {
      // Redirect unauthenticated requests to home page
      res.redirect('/')
    } else {
      let params = {
        active: { calendar: true }
      };

      // Get the access token
      var accessToken;
      try {
        accessToken = await tokens.getAccessToken(req);
      } catch (err) {
        res.json(err);
      }

      if (accessToken && accessToken.length > 0) {
        try {
          // Get the events
          var events = await graph.getEvents(accessToken);

          res.json(events.value);
        } catch (err) {
          res.json(err);
        }
      }
    }
  }
);

module.exports = router;
```

Aktualisieren `./app.js` Sie, um diese neue Route zu verwenden. Fügen Sie die folgende **** Codezeile vor `var app = express();` der Leitung hinzu.

```js
var calendarRouter = require('./routes/calendar');
```

Fügen Sie dann die folgende **** Codezeile nach `app.use('/auth', authRouter);` der Leitung hinzu.

```js
app.use('/calendar', calendarRouter);
```

Jetzt können Sie dies testen. Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** . Wenn alles funktioniert, sollte ein JSON-Dump von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Jetzt können Sie eine Ansicht hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen. Fügen Sie zunächst den folgenden Code in `./app.js` **nach** der `app.set('view engine', 'hbs');` -Leitung hinzu.

```js
var hbs = require('hbs');
var moment = require('moment');
// Helper to format date/time sent by Graph
hbs.registerHelper('eventDateTime', function(dateTime){
  return moment(dateTime).format('M/D/YY h:mm A');
});
```

Dies implementiert einen [Lenker-Helfer](http://handlebarsjs.com/#helpers) , um das von Microsoft Graph zurückgegebene ISO 8601-Datum in etwas menschlich freundlicheres zu formatieren.

Erstellen Sie eine neue Datei im `./views` Verzeichnis mit `calendar.hbs` dem Namen, und fügen Sie den folgenden Code hinzu.

```html
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    {{#each events}}
      <tr>
        <td>{{this.organizer.emailAddress.name}}</td>
        <td>{{this.subject}}</td>
        <td>{{eventDateTime this.start.dateTime}}</td>
        <td>{{eventDateTime this.end.dateTime}}</td>
      </tr>
    {{/each}}
  </tbody>
</table>
```

, Der eine Auflistung von Ereignissen durchläuft und jeweils eine Tabellenzeile hinzufügt. Aktualisieren Sie nun die Route `./routes/calendar.js` in, um diese Ansicht zu verwenden. Ersetzen Sie die vorhandene Route durch den folgenden Code.

```js
router.get('/',
  async function(req, res) {
    if (!req.isAuthenticated()) {
      // Redirect unauthenticated requests to home page
      res.redirect('/')
    } else {
      let params = {
        active: { calendar: true }
      };

      // Get the access token
      var accessToken;
      try {
        accessToken = await tokens.getAccessToken(req);
      } catch (err) {
        req.flash('error_msg', {
          message: 'Could not get access token. Try signing out and signing in again.',
          debug: JSON.stringify(err)
        });
      }

      if (accessToken && accessToken.length > 0) {
        try {
          // Get the events
          var events = await graph.getEvents(accessToken);
          params.events = events.value;
        } catch (err) {
          req.flash('error_msg', {
            message: 'Could not fetch events',
            debug: JSON.stringify(err)
          });
        }
      }

      res.render('calendar', params);
    }
  }
);
```

Speichern Sie Ihre Änderungen, starten Sie den Server neu, und melden Sie sich bei der APP an. Klicken Sie auf den Link **Kalender** , und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.

![Screenshot der Ereignistabelle](./images/add-msgraph-01.png)