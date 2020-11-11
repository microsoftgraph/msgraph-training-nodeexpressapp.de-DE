<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="386a7-101">In dieser Übung werden Sie Microsoft Graph in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="386a7-101">In this exercise you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="386a7-102">Für diese Anwendung verwenden Sie die [Microsoft-Graph-Client-](https://github.com/microsoftgraph/msgraph-sdk-javascript) Bibliothek, um Anrufe an Microsoft Graph zu tätigen.</span><span class="sxs-lookup"><span data-stu-id="386a7-102">For this application, you will use the [microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="386a7-103">Abrufen von Kalenderereignissen von Outlook</span><span class="sxs-lookup"><span data-stu-id="386a7-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="386a7-104">Öffnen Sie **./graph.js** , und fügen Sie die folgende Funktion in hinzu `module.exports` .</span><span class="sxs-lookup"><span data-stu-id="386a7-104">Open **./graph.js** and add the following function inside `module.exports`.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/graph.js" id="GetCalendarViewSnippet":::

    <span data-ttu-id="386a7-105">Überlegen Sie sich, was dieser Code macht.</span><span class="sxs-lookup"><span data-stu-id="386a7-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="386a7-106">Die URL, die aufgerufen wird, lautet `/me/events`.</span><span class="sxs-lookup"><span data-stu-id="386a7-106">The URL that will be called is `/me/events`.</span></span>
    - <span data-ttu-id="386a7-107">Die `select` -Methode schränkt die für die einzelnen Ereignisse zurückgegebenen Felder auf genau diejenigen ein, die die Ansicht tatsächlich verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="386a7-107">The `select` method limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="386a7-108">Die `orderby` Methode sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu der Sie erstellt wurden, wobei das letzte Element zuerst angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="386a7-108">The `orderby` method sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="386a7-109">Erstellen Sie eine neue Datei im **./routes** -Verzeichnis mit dem Namen **calendar.js** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="386a7-109">Create a new file in the **./routes** directory named **calendar.js** , and add the following code.</span></span>

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

1. <span data-ttu-id="386a7-110">Update **./app.js** , um diese neue Route zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="386a7-110">Update **./app.js** to use this new route.</span></span> <span data-ttu-id="386a7-111">Fügen Sie die folgende **vor** der `var app = express();` -Reihe hinzu.</span><span class="sxs-lookup"><span data-stu-id="386a7-111">Add the following line **before** the `var app = express();` line.</span></span>

    ```javascript
    var calendarRouter = require('./routes/calendar');
    ```

1. <span data-ttu-id="386a7-112">Fügen Sie die folgende Folge **nach** der `app.use('/auth', authRouter);` -Verbindung hinzu.</span><span class="sxs-lookup"><span data-stu-id="386a7-112">Add the following line **after** the `app.use('/auth', authRouter);` line.</span></span>

    ```javascript
    app.use('/calendar', calendarRouter);
    ```

1. <span data-ttu-id="386a7-113">Starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="386a7-113">Restart the server.</span></span> <span data-ttu-id="386a7-114">Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** .</span><span class="sxs-lookup"><span data-stu-id="386a7-114">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="386a7-115">Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="386a7-115">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="386a7-116">Anzeigen der Ergebnisse</span><span class="sxs-lookup"><span data-stu-id="386a7-116">Display the results</span></span>

<span data-ttu-id="386a7-117">Jetzt können Sie eine Ansicht hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="386a7-117">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="386a7-118">Fügen Sie den folgenden Code in **./app.js nach der-** `app.set('view engine', 'hbs');` Verbindung hinzu.</span><span class="sxs-lookup"><span data-stu-id="386a7-118">Add the following code in **./app.js after** the `app.set('view engine', 'hbs');` line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="FormatDateSnippet":::

    <span data-ttu-id="386a7-119">Dadurch wird ein [Lenker-Helfer](http://handlebarsjs.com/#helpers) implementiert, um das von Microsoft Graph zurückgegebene ISO 8601-Datum in etwas menschlicheres zu formatieren.</span><span class="sxs-lookup"><span data-stu-id="386a7-119">This implements a [Handlebars helper](http://handlebarsjs.com/#helpers) to format the ISO 8601 date returned by Microsoft Graph into something more human-friendly.</span></span>

1. <span data-ttu-id="386a7-120">Erstellen Sie eine neue Datei im Verzeichnis **./views** mit dem Namen **Calendar. HB** , und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="386a7-120">Create a new file in the **./views** directory named **calendar.hbs** and add the following code.</span></span>

    :::code language="html" source="../demo/graph-tutorial/views/calendar.hbs" id="LayoutSnippet":::

    <span data-ttu-id="386a7-121">Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="386a7-121">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="386a7-122">Aktualisieren Sie nun die Route in **./routes/calendar.js** , um diese Ansicht zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="386a7-122">Now update the route in **./routes/calendar.js** to use this view.</span></span> <span data-ttu-id="386a7-123">Ersetzen Sie die vorhandene Route durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="386a7-123">Replace the existing route with the following code.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/calendar.js" id="GetRouteSnippet" highlight="33-36,49,51-54,61":::

1. <span data-ttu-id="386a7-124">Speichern Sie Ihre Änderungen, starten Sie den Server neu, und melden Sie sich bei der APP an.</span><span class="sxs-lookup"><span data-stu-id="386a7-124">Save your changes, restart the server, and sign in to the app.</span></span> <span data-ttu-id="386a7-125">Klicken Sie auf den Link **Kalender** , und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.</span><span class="sxs-lookup"><span data-stu-id="386a7-125">Click on the **Calendar** link and the app should now render a table of events.</span></span>

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
