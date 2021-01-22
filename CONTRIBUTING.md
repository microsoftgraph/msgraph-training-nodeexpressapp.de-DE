# <a name="contributing-to-microsoft-graph-training-repositories"></a>Beitrag zu Microsoft Graph-Schulungsrepositorys

Vielen Dank, dass Sie zu diesem Projekt beigetragen haben! Bevor Sie Ihre Pullanforderung übermitteln, sollten Sie Folgendes berücksichtigen.

## <a name="overview"></a>Übersicht

Der Code in diesem Repository dient drei Zwecken:

- Die Markdown-Dateien [im](/tutorial) Lernprogrammordner werden als Lernprogramm auf der [Microsoft Graph-Lernprogrammseite](https://docs.microsoft.com/graph/tutorials) veröffentlicht.
- Das Beispielprojekt im [Demoordner](/demo) ist die Quelle für einen [Microsoft Graph-Schnellstart.*](https://developer.microsoft.com/graph/quick-start) *\** _
- Das Beispielprojekt im Demoordner kann auch direkt von GitHub heruntergeladen werden und sollte nach einer einfachen Konfiguration wie folgt ausgeführt werden.

> _*\**_ Nicht alle Schulungsrepositorys sind (noch) als Schnellstart verfügbar.

Dies ist wichtig zu beachten, da Änderungen an einem Ort _may* Änderungen an einem anderen erfordern, um die Dinge synchron zu halten. Wenn möglich, verweisen die #A0 direkt (mithilfe einer benutzerdefinierten Syntax) auf die Quellcodedateien, sodass beim Aktualisieren von Code in der Quelle der Code `:::code` in Markdown automatisch aktualisiert wird.

## <a name="updating-code"></a>Aktualisieren von Code

Die `:::code` in Markdown verwendete Syntax hängt von bestimmten Kommentaren in der Quellcodedatei ab. Diese Kommentare sehen wie folgt aus:

```csharp
// <MySnippet>
Console.WriteLine("Hello World!");
// </MySnippet>
```

Wenn Sie Code zwischen diesen "Marker"-Kommentaren aktualisieren, erhalten die Markdown-Dateien diese Änderungen automatisch, wenn sie auf der Microsoft Graph-Dokumentationswebsite veröffentlicht werden. Wenn Sie Code außerhalb dieser Kommentare aktualisieren, ist es sehr wahrscheinlich, dass Sie den entsprechenden Markdown aktualisieren müssen.

## <a name="adding-features"></a>Hinzufügen von Features

Die Begeisterung wird zwar geschätzt, aber senden Sie keine Pullanforderungen, um dem Beispiel neue Features hinzuzufügen. Da es sich bei diesem Repository in erster Linie um ein Lernprogramm zum Erstellen Ihrer ersten App handelt, ist der Featuresatz entwurfsweise eingeschränkt.

## <a name="submitting-pull-requests"></a>Senden von Pullanforderungen

Senden Sie alle Pullanforderungen an die `master` Verzweigung.

## <a name="when-do-changes-get-published"></a>Wann werden Änderungen veröffentlicht?

Die Veröffentlichung von Updates auf der [Microsoft Graph-Lernprogrammwebsite](https://docs.microsoft.com/graph/tutorials) erfolgt nicht automatisch. Änderungen müssen zuerst in die Verzweigung heraufgestuft werden, dann muss ein `live` Build von den Websiteadministratoren ausgelöst werden. Dies erfolgt in der Regel "nach Bedarf".

## <a name="code-of-conduct"></a>Verhaltensregeln

In diesem Projekt wurden die [Microsoft Open Source-Verhaltensregeln](https://opensource.microsoft.com/codeofconduct/) übernommen. Weitere Informationen finden Sie unter [Häufig gestellte Fragen zu Verhaltensregeln](https://opensource.microsoft.com/codeofconduct/faq/), oder richten Sie Ihre Fragen oder Kommentare an [opencode@microsoft.com](mailto:opencode@microsoft.com).
