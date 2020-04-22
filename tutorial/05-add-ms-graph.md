<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="09f99-101">In dieser Übung werden Sie Microsoft Graph in die Anwendung integrieren.</span><span class="sxs-lookup"><span data-stu-id="09f99-101">In this exercise you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="09f99-102">Für diese Anwendung verwenden Sie die [Microsoft-Graph-Client-](https://github.com/microsoftgraph/msgraph-sdk-javascript) Bibliothek, um Anrufe an Microsoft Graph zu tätigen.</span><span class="sxs-lookup"><span data-stu-id="09f99-102">For this application, you will use the [microsoft-graph-client](https://github.com/microsoftgraph/msgraph-sdk-javascript) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="09f99-103">Abrufen von Kalenderereignissen von Outlook</span><span class="sxs-lookup"><span data-stu-id="09f99-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="09f99-104">Öffnen `./graph.js` Sie und fügen Sie die folgende `module.exports`Funktion in ein.</span><span class="sxs-lookup"><span data-stu-id="09f99-104">Open `./graph.js` and add the following function inside `module.exports`.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/graph.js" id="GetEventsSnippet":::

    <span data-ttu-id="09f99-105">Überlegen Sie sich, was dieser Code macht.</span><span class="sxs-lookup"><span data-stu-id="09f99-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="09f99-106">Die URL, die aufgerufen wird, lautet `/me/events`.</span><span class="sxs-lookup"><span data-stu-id="09f99-106">The URL that will be called is `/me/events`.</span></span>
    - <span data-ttu-id="09f99-107">Die `select` -Methode schränkt die für die einzelnen Ereignisse zurückgegebenen Felder auf genau diejenigen ein, die die Ansicht tatsächlich verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="09f99-107">The `select` method limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="09f99-108">Die `orderby` Methode sortiert die Ergebnisse nach dem Datum und der Uhrzeit, zu der Sie erstellt wurden, wobei das letzte Element zuerst angezeigt wird.</span><span class="sxs-lookup"><span data-stu-id="09f99-108">The `orderby` method sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="09f99-109">Erstellen Sie eine neue Datei im `./routes` Verzeichnis mit `calendar.js`dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="09f99-109">Create a new file in the `./routes` directory named `calendar.js`, and add the following code.</span></span>

    ```javascript
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
          else {
            req.flash('error_msg', 'Could not get an access token');
          }
        }
      }
    );

    module.exports = router;
    ```

1. <span data-ttu-id="09f99-110">Aktualisieren `./app.js` Sie diese neue Route.</span><span class="sxs-lookup"><span data-stu-id="09f99-110">Update `./app.js` to use this new route.</span></span> <span data-ttu-id="09f99-111">Fügen Sie die folgende **vor** der `var app = express();` -Reihe hinzu.</span><span class="sxs-lookup"><span data-stu-id="09f99-111">Add the following line **before** the `var app = express();` line.</span></span>

    ```javascript
    var calendarRouter = require('./routes/calendar');
    ```

1. <span data-ttu-id="09f99-112">Fügen Sie die folgende **after** Folge nach `app.use('/auth', authRouter);` der-Verbindung hinzu.</span><span class="sxs-lookup"><span data-stu-id="09f99-112">Add the following line **after** the `app.use('/auth', authRouter);` line.</span></span>

    ```javascript
    app.use('/calendar', calendarRouter);
    ```

1. <span data-ttu-id="09f99-113">Starten Sie den Server neu.</span><span class="sxs-lookup"><span data-stu-id="09f99-113">Restart the server.</span></span> <span data-ttu-id="09f99-114">Melden Sie sich an, und klicken Sie in der Navigationsleiste auf den Link **Kalender** .</span><span class="sxs-lookup"><span data-stu-id="09f99-114">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="09f99-115">Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="09f99-115">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="09f99-116">Anzeigen der Ergebnisse</span><span class="sxs-lookup"><span data-stu-id="09f99-116">Display the results</span></span>

<span data-ttu-id="09f99-117">Jetzt können Sie eine Ansicht hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="09f99-117">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="09f99-118">Fügen Sie den folgenden Code `./app.js` **nach** der `app.set('view engine', 'hbs');` -Codezeile hinzu.</span><span class="sxs-lookup"><span data-stu-id="09f99-118">Add the following code in `./app.js` **after** the `app.set('view engine', 'hbs');` line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/app.js" id="FormatDateSnippet":::

    <span data-ttu-id="09f99-119">Dadurch wird ein [Lenker-Helfer](http://handlebarsjs.com/#helpers) implementiert, um das von Microsoft Graph zurückgegebene ISO 8601-Datum in etwas menschlicheres zu formatieren.</span><span class="sxs-lookup"><span data-stu-id="09f99-119">This implements a [Handlebars helper](http://handlebarsjs.com/#helpers) to format the ISO 8601 date returned by Microsoft Graph into something more human-friendly.</span></span>

1. <span data-ttu-id="09f99-120">Erstellen Sie eine neue Datei im `./views` Verzeichnis mit `calendar.hbs` dem Namen, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="09f99-120">Create a new file in the `./views` directory named `calendar.hbs` and add the following code.</span></span>

    :::code language="html" source="../demo/graph-tutorial/views/calendar.hbs" id="LayoutSnippet":::

    <span data-ttu-id="09f99-121">Dadurch wird eine Ereignissammlung durchlaufen und jedem Ereignis wird jeweils eine Tabellenzeile hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="09f99-121">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="09f99-122">Aktualisieren Sie nun die Route `./routes/calendar.js` in, um diese Ansicht zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="09f99-122">Now update the route in `./routes/calendar.js` to use this view.</span></span> <span data-ttu-id="09f99-123">Ersetzen Sie die vorhandene Route durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="09f99-123">Replace the existing route with the following code.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/calendar.js" id="GetRouteSnippet" highlight="16-19,26,28-31,37":::

1. <span data-ttu-id="09f99-124">Speichern Sie Ihre Änderungen, starten Sie den Server neu, und melden Sie sich bei der APP an.</span><span class="sxs-lookup"><span data-stu-id="09f99-124">Save your changes, restart the server, and sign in to the app.</span></span> <span data-ttu-id="09f99-125">Klicken Sie auf den Link **Kalender** , und die APP sollte jetzt eine Tabelle mit Ereignissen rendern.</span><span class="sxs-lookup"><span data-stu-id="09f99-125">Click on the **Calendar** link and the app should now render a table of events.</span></span>

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
