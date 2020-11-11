<!-- markdownlint-disable MD002 MD041 -->

In diesem Abschnitt können Sie die Möglichkeit zum Erstellen von Ereignissen im Kalender des Benutzers hinzufügen.

## <a name="create-a-new-event-form"></a>Erstellen eines neuen Ereignis Formulars

1. Erstellen Sie eine neue Datei im Verzeichnis **./views** namens "New **Event. HB** ", und fügen Sie den folgenden Code hinzu.

    :::code language="html" source="../demo/graph-tutorial/views/newevent.hbs" id="NewEventFormSnippet":::

1. Fügen Sie den folgenden Code der **/Routes/-calendar.js** Datei vor der- `module.exports = router;` Verbindung hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/routes/calendar.js" id="GetEventFormSnippet":::

Dadurch wird ein Formular für die Benutzereingabe implementiert und gerendert.

## <a name="create-the-event"></a>Erstellen des Ereignisses

1. Öffnen Sie **./graph.js** , und fügen Sie die folgende Funktion in hinzu `module.exports` .

    :::code language="javascript" source="../demo/graph-tutorial/graph.js" id="CreateEventSnippet":::

    Dieser Code verwendet die Formularfelder zum Erstellen eines Graph-Ereignisobjekts und sendet dann eine Post-Anforderung an den `/me/events` Endpunkt, um das Ereignis im Standardkalender des Benutzers zu erstellen.

1. Fügen Sie den folgenden Code der **/Routes/-calendar.js** Datei vor der- `module.exports = router;` Verbindung hinzu.

    :::code language="javascript" source="../demo/graph-tutorial/routes/calendar.js" id="PostEventFormSnippet":::

    Mit diesem Code wird die Formulareingabe überprüft und bereinigt, anschließend `graph.createEvent` werden Aufrufe zum Erstellen des Ereignisses durchführen. Nachdem der Anruf abgeschlossen wurde, wird er wieder in die Kalenderansicht umgeleitet.

1. Speichern Sie die Änderungen, und starten Sie die App neu. Klicken Sie auf das Navigationselement **Kalender** , und klicken Sie dann auf die Schaltfläche **Ereignis erstellen** . Geben Sie die Werte ein, und klicken Sie auf **Erstellen**. Die APP kehrt zur Kalenderansicht zurück, wenn das neue Ereignis erstellt wird.

    ![Screenshot des neuen Ereignis Formulars](images/create-event-01.png)
