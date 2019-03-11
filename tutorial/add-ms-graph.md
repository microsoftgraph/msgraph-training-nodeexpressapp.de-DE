<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f368d-101">In dieser Übung integrieren Sie Microsoft Graph in die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="f368d-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="f368d-102">Für diese Anwendung verwenden Sie die [Microsoft-Graph-Client-](https://github.com/microsoftgraph/msgraph-sdk-javascript) Bibliothek, um Aufrufe von Microsoft Graph zu tätigen.</span><span class="sxs-lookup"><span data-stu-id="f368d-102">For this application, you will use the [microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="f368d-103">Abrufen von Kalenderereignissen aus Outlook</span><span class="sxs-lookup"><span data-stu-id="f368d-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="f368d-104">Beginnen Sie mit dem Hinzufügen einer neuen `./graph.js` Methode zur Datei, um die Ereignisse aus dem Kalender abzurufen.</span><span class="sxs-lookup"><span data-stu-id="f368d-104">Start by adding a new method to the `./graph.js` file to get the events from the calendar.</span></span> <span data-ttu-id="f368d-105">Fügen Sie die folgende Funktion innerhalb `module.exports` der `./graph.js`in hinzu.</span><span class="sxs-lookup"><span data-stu-id="f368d-105">Add the following function inside the `module.exports` in `./graph.js`.</span></span>

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

<span data-ttu-id="f368d-106">Überlegen Sie sich, was dieser Code tut.</span><span class="sxs-lookup"><span data-stu-id="f368d-106">Consider what this code is doing.</span></span>

- <span data-ttu-id="f368d-107">Die URL, die aufgerufen wird, `/me/events`lautet.</span><span class="sxs-lookup"><span data-stu-id="f368d-107">The URL that will be called is `/me/events`.</span></span>
- <span data-ttu-id="f368d-108">Die `select` -Methode schränkt die für jedes Ereignis zurückgegebenen Felder auf nur die ein, die die Ansicht tatsächlich verwendet.</span><span class="sxs-lookup"><span data-stu-id="f368d-108">The `select` method limits the fields returned for each events to just those the view will actually use.</span></span>
- <span data-ttu-id="f368d-109">Die `orderby` -Methode sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu denen Sie erstellt wurden, wobei das neueste Element zuerst angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="f368d-109">The `orderby` method sorts the results by the date and time they were created, with the most recent item being first.</span></span>

<span data-ttu-id="f368d-110">Erstellen Sie eine neue Datei im `./routes` Verzeichnis mit `calendar.js`dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f368d-110">Create a new file in the `./routes` directory named `calendar.js`, and add the following code.</span></span>

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

<span data-ttu-id="f368d-111">Aktualisieren `./app.js` Sie, um diese neue Route zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="f368d-111">Update `./app.js` to use this new route.</span></span> <span data-ttu-id="f368d-112">Fügen Sie die folgende \*\*\*\* Codezeile vor `var app = express();` der Leitung hinzu.</span><span class="sxs-lookup"><span data-stu-id="f368d-112">Add the following line **before** the `var app = express();` line.</span></span>

```js
var calendarRouter = require('./routes/calendar');
```

<span data-ttu-id="f368d-113">Fügen Sie dann die folgende \*\*\*\* Codezeile nach `app.use('/auth', authRouter);` der Leitung hinzu.</span><span class="sxs-lookup"><span data-stu-id="f368d-113">Then add the following line **after** the `app.use('/auth', authRouter);` line.</span></span>

```js
app.use('/calendar', calendarRouter);
```

<span data-ttu-id="f368d-114">Jetzt können Sie dies testen.</span><span class="sxs-lookup"><span data-stu-id="f368d-114">Now you can test this.</span></span> <span data-ttu-id="f368d-115">Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** .</span><span class="sxs-lookup"><span data-stu-id="f368d-115">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="f368d-116">Wenn alles funktioniert, sollte ein JSON-Dump von Ereignissen im Kalender des Benutzers angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="f368d-116">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="f368d-117">Anzeigen der Ergebnisse</span><span class="sxs-lookup"><span data-stu-id="f368d-117">Display the results</span></span>

<span data-ttu-id="f368d-118">Jetzt können Sie eine Ansicht hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="f368d-118">Now you can add a view to display the results in a more user-friendly manner.</span></span> <span data-ttu-id="f368d-119">Fügen Sie zunächst den folgenden Code in `./app.js` **nach** der `app.set('view engine', 'hbs');` -Leitung hinzu.</span><span class="sxs-lookup"><span data-stu-id="f368d-119">First, add the following code in `./app.js` **after** the `app.set('view engine', 'hbs');` line.</span></span>

```js
var hbs = require('hbs');
var moment = require('moment');
// Helper to format date/time sent by Graph
hbs.registerHelper('eventDateTime', function(dateTime){
  return moment(dateTime).format('M/D/YY h:mm A');
});
```

<span data-ttu-id="f368d-120">Dies implementiert einen [Lenker-Helfer](http://handlebarsjs.com/#helpers) , um das von Microsoft Graph zurückgegebene ISO 8601-Datum in etwas menschlich freundlicheres zu formatieren.</span><span class="sxs-lookup"><span data-stu-id="f368d-120">This implements a [Handlebars helper](http://handlebarsjs.com/#helpers) to format the ISO 8601 date returned by Microsoft Graph into something more human-friendly.</span></span>

<span data-ttu-id="f368d-121">Erstellen Sie eine neue Datei im `./views` Verzeichnis mit `calendar.hbs` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="f368d-121">Create a new file in the `./views` directory named `calendar.hbs` and add the following code.</span></span>

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

<span data-ttu-id="f368d-122">, Der eine Auflistung von Ereignissen durchläuft und jeweils eine Tabellenzeile hinzufügt.</span><span class="sxs-lookup"><span data-stu-id="f368d-122">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="f368d-123">Aktualisieren Sie nun die Route `./routes/calendar.js` in, um diese Ansicht zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="f368d-123">Now update the route in `./routes/calendar.js` to use this view.</span></span> <span data-ttu-id="f368d-124">Ersetzen Sie die vorhandene Route durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="f368d-124">Replace the existing route with the following code.</span></span>

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

<span data-ttu-id="f368d-125">Speichern Sie Ihre Änderungen, starten Sie den Server neu, und melden Sie sich bei der APP an.</span><span class="sxs-lookup"><span data-stu-id="f368d-125">Save your changes, restart the server, and sign in to the app.</span></span> <span data-ttu-id="f368d-126">Klicken Sie auf den Link **Kalender** , und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.</span><span class="sxs-lookup"><span data-stu-id="f368d-126">Click on the **Calendar** link and the app should now render a table of events.</span></span>

![Screenshot der Ereignistabelle](./images/add-msgraph-01.png)