<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung integrieren Sie Microsoft Graph in die Anwendung. Für diese Anwendung verwenden Sie die [Microsoft-graph-Clientbibliothek,](https://github.com/microsoftgraph/msgraph-sdk-javascript) um Aufrufe an Microsoft Graph zu senden.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen von Outlook

1. Öffnen **Sie ./graph.js,** und fügen Sie die folgende Funktion in `module.exports` hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/graph.js" id="GetCalendarViewSnippet":::

    Überlegen Sie sich, was dieser Code macht.

    - Die URL, die aufgerufen wird, lautet `/me/calendarview`.
    - Die Methode fügt der Anforderung den Header hinzu, wodurch die Start- und Endzeiten in der Zeitzone des `header` `Prefer: outlook.timezone` Benutzers zurückgegeben werden.
    - Die `query` Methode legt die Parameter und die Parameter für die `startDateTime` `endDateTime` Kalenderansicht fest.
    - Die Methode beschränkt die für jedes Ereignis zurückgegebenen Felder auf die Felder, die `select` tatsächlich von der Ansicht verwendet werden.
    - Die `orderby` Methode sortiert die Ergebnisse nach Startzeit.
    - Die `top` Methode beschränkt die Ergebnisse auf 50 Ereignisse.

1. Erstellen Sie eine neue Datei im **Verzeichnis ./routes** namens **calendar.js**, und fügen Sie den folgenden Code hinzu.

    ```javascript
    const router = require('express-promise-router')();
    const graph = require('../graph.js');
    const moment = require('moment-timezone');
    const iana = require('windows-iana');
    const { body, validationResult } = require('express-validator');
    const validator = require('validator');

    /* GET /calendar */
    router.get('/',
      async function(req, res) {
        if (!req.session.userId) {
          // Redirect unauthenticated requests to home page
          res.redirect('/')
        } else {
          const params = {
            active: { calendar: true }
          };

          // Get the user
          const user = req.app.locals.users[req.session.userId];
          // Convert user's Windows time zone ("Pacific Standard Time")
          // to IANA format ("America/Los_Angeles")
          // Moment needs IANA format
          const timeZoneId = iana.findOneIana(user.timeZone);
          console.log(`Time zone: ${timeZoneId.valueOf()}`);

          // Calculate the start and end of the current week
          // Get midnight on the start of the current week in the user's timezone,
          // but in UTC. For example, for Pacific Standard Time, the time value would be
          // 07:00:00Z
          var startOfWeek = moment.tz(timeZoneId.valueOf()).startOf('week').utc();
          var endOfWeek = moment(startOfWeek).add(7, 'day');
          console.log(`Start: ${startOfWeek.format()}`);

          // Get the access token
          var accessToken;
          try {
            accessToken = await getAccessToken(req.session.userId, req.app.locals.msalClient);
          } catch (err) {
            res.send(JSON.stringify(err, Object.getOwnPropertyNames(err)));
            return;
          }

          if (accessToken && accessToken.length > 0) {
            try {
              // Get the events
              const events = await graph.getCalendarView(
                accessToken,
                startOfWeek.format(),
                endOfWeek.format(),
                user.timeZone);

              res.json(events.value);
            } catch (err) {
              res.send(JSON.stringify(err, Object.getOwnPropertyNames(err)));
            }
          }
          else {
            req.flash('error_msg', 'Could not get an access token');
          }
        }
      }
    );

    async function getAccessToken(userId, msalClient) {
      // Look up the user's account in the cache
      try {
        const accounts = await msalClient
          .getTokenCache()
          .getAllAccounts();

        const userAccount = accounts.find(a => a.homeAccountId === userId);

        // Get the token silently
        const response = await msalClient.acquireTokenSilent({
          scopes: process.env.OAUTH_SCOPES.split(','),
          redirectUri: process.env.OAUTH_REDIRECT_URI,
          account: userAccount
        });

        return response.accessToken;
      } catch (err) {
        console.log(JSON.stringify(err, Object.getOwnPropertyNames(err)));
      }
    }

    module.exports = router;
    ```

1. Aktualisieren **Sie ./app.js,** um diese neue Route zu verwenden. Fügen Sie die folgende Zeile **vor der** Zeile `var app = express();` hinzu.

    ```javascript
    var calendarRouter = require('./routes/calendar');
    ```

1. Fügen Sie die folgende **Zeile hinter der** Zeile `app.use('/auth', authRouter);` hinzu.

    ```javascript
    app.use('/calendar', calendarRouter);
    ```

1. Starten Sie den Server neu. Melden Sie sich an, und klicken **Sie** in der Navigationsleiste auf den Kalenderlink. Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Jetzt können Sie eine Ansicht hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen.

1. Fügen Sie den folgenden Code in **"./app.js der** Zeile `app.set('view engine', 'hbs');` hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="FormatDateSnippet":::

    Dadurch wird ein [Handlebars-Hilfsfeld](http://handlebarsjs.com/#helpers) implementiert, um das von Microsoft Graph zurückgegebene ISO 8601-Datum in ein menschenfreundliches Format zu formatieren.

1. Erstellen Sie eine neue Datei im **Verzeichnis "./views"** mit dem Namen **"calendar.barcodes",** und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/graph-tutorial/views/calendar.hbs" id="LayoutSnippet":::

    Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.

1. Aktualisieren Sie jetzt die Route in **./routes/calendar.js,** um diese Ansicht zu verwenden. Ersetzen Sie die vorhandene Route durch den folgenden Code.

    :::code language="javascript" source="../demo/graph-tutorial/routes/calendar.js" id="GetRouteSnippet" highlight="33-36,49,51-54,61":::

1. Speichern Sie Ihre Änderungen, starten Sie den Server neu, und melden Sie sich bei der App an. Klicken Sie auf den **Kalenderlink,** und die App sollte nun eine Tabelle mit Ereignissen rendern.

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
