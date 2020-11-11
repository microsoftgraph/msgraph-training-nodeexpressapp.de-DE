<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="abb88-101">In diesem Abschnitt können Sie die Möglichkeit zum Erstellen von Ereignissen im Kalender des Benutzers hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="abb88-101">In this section you will add the ability to create events on the user's calendar.</span></span>

## <a name="create-a-new-event-form"></a><span data-ttu-id="abb88-102">Erstellen eines neuen Ereignis Formulars</span><span class="sxs-lookup"><span data-stu-id="abb88-102">Create a new event form</span></span>

1. <span data-ttu-id="abb88-103">Erstellen Sie eine neue Datei im Verzeichnis **./views** namens "New **Event. HB** ", und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="abb88-103">Create a new file in the **./views** directory named **newevent.hbs** and add the following code.</span></span>

    :::code language="html" source="../demo/graph-tutorial/views/newevent.hbs" id="NewEventFormSnippet":::

1. <span data-ttu-id="abb88-104">Fügen Sie den folgenden Code der **/Routes/-calendar.js** Datei vor der- `module.exports = router;` Verbindung hinzu.</span><span class="sxs-lookup"><span data-stu-id="abb88-104">Add the following code to the **./routes/calendar.js** file before the `module.exports = router;` line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/calendar.js" id="GetEventFormSnippet":::

<span data-ttu-id="abb88-105">Dadurch wird ein Formular für die Benutzereingabe implementiert und gerendert.</span><span class="sxs-lookup"><span data-stu-id="abb88-105">This implements a form for user input and renders it.</span></span>

## <a name="create-the-event"></a><span data-ttu-id="abb88-106">Erstellen des Ereignisses</span><span class="sxs-lookup"><span data-stu-id="abb88-106">Create the event</span></span>

1. <span data-ttu-id="abb88-107">Öffnen Sie **./graph.js** , und fügen Sie die folgende Funktion in hinzu `module.exports` .</span><span class="sxs-lookup"><span data-stu-id="abb88-107">Open **./graph.js** and add the following function inside `module.exports`.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/graph.js" id="CreateEventSnippet":::

    <span data-ttu-id="abb88-108">Dieser Code verwendet die Formularfelder zum Erstellen eines Graph-Ereignisobjekts und sendet dann eine Post-Anforderung an den `/me/events` Endpunkt, um das Ereignis im Standardkalender des Benutzers zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="abb88-108">This code uses the form fields to create a Graph event object, then sends a POST request to the `/me/events` endpoint to create the event on the user's default calendar.</span></span>

1. <span data-ttu-id="abb88-109">Fügen Sie den folgenden Code der **/Routes/-calendar.js** Datei vor der- `module.exports = router;` Verbindung hinzu.</span><span class="sxs-lookup"><span data-stu-id="abb88-109">Add the following code to the **./routes/calendar.js** file before the `module.exports = router;` line.</span></span>

    :::code language="javascript" source="../demo/graph-tutorial/routes/calendar.js" id="PostEventFormSnippet":::

    <span data-ttu-id="abb88-110">Mit diesem Code wird die Formulareingabe überprüft und bereinigt, anschließend `graph.createEvent` werden Aufrufe zum Erstellen des Ereignisses durchführen.</span><span class="sxs-lookup"><span data-stu-id="abb88-110">This code validates and sanitized the form input, then calls `graph.createEvent` to create the event.</span></span> <span data-ttu-id="abb88-111">Nachdem der Anruf abgeschlossen wurde, wird er wieder in die Kalenderansicht umgeleitet.</span><span class="sxs-lookup"><span data-stu-id="abb88-111">It redirects back to the calendar view after the call completes.</span></span>

1. <span data-ttu-id="abb88-112">Speichern Sie die Änderungen, und starten Sie die App neu.</span><span class="sxs-lookup"><span data-stu-id="abb88-112">Save your changes and restart the app.</span></span> <span data-ttu-id="abb88-113">Klicken Sie auf das Navigationselement **Kalender** , und klicken Sie dann auf die Schaltfläche **Ereignis erstellen** .</span><span class="sxs-lookup"><span data-stu-id="abb88-113">Click the **Calendar** nav item, then click the **Create event** button.</span></span> <span data-ttu-id="abb88-114">Geben Sie die Werte ein, und klicken Sie auf **Erstellen**.</span><span class="sxs-lookup"><span data-stu-id="abb88-114">Fill in the values and click **Create**.</span></span> <span data-ttu-id="abb88-115">Die APP kehrt zur Kalenderansicht zurück, wenn das neue Ereignis erstellt wird.</span><span class="sxs-lookup"><span data-stu-id="abb88-115">The app returns to the calendar view once the new event is created.</span></span>

    ![Screenshot des neuen Ereignis Formulars](images/create-event-01.png)
