<!-- markdownlint-disable MD002 MD041 -->

In diesem Lernprogramm erfahren Sie, wie Sie eine Node.js Express-Webanwendung erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.

> [!TIP]
> Wenn Sie das fertige Lernprogramm einfach herunterladen möchten, können Sie es auf zwei Arten herunterladen.
>
> - Laden Sie den [Node.js Schnellstart](https://developer.microsoft.com/graph/quick-start?platform=option-node) herunter, um in wenigen Minuten Arbeitscode zu erhalten.
> - Laden Sie das [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp)herunter, oder Klonen Sie es.

## <a name="prerequisites"></a>Voraussetzungen

Bevor Sie mit dieser Demo beginnen, sollten Sie [Node.js](https://nodejs.org) auf Ihrem Entwicklungscomputer installiert haben. Wenn Sie nicht über Node.js verfügen, besuchen Sie den vorherigen Link für Downloadoptionen.

> [!NOTE]
> Windows-Benutzer müssen möglicherweise python-und Visual Studio-Erstellungs Tools installieren, um NPM-Module zu unterstützen, die aus C/C++ kompiliert werden müssen. Das Node.js-Installationsprogramm unter Windows bietet die Möglichkeit, diese Tools automatisch zu installieren. Alternativ können Sie den Anweisungen unter folgen [https://github.com/nodejs/node-gyp#on-windows](https://github.com/nodejs/node-gyp#on-windows) .

Sie sollten auch über ein persönliches Microsoft-Konto mit einem Postfach auf Outlook.com oder ein Microsoft-Arbeits-oder Schulkonto verfügen. Wenn Sie kein Microsoft-Konto haben, gibt es mehrere Optionen, um ein kostenloses Konto zu erhalten:

- Sie können [sich für ein neues persönliches Microsoft-Konto registrieren](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Sie können sich [für das Office 365 Entwicklerprogramm registrieren](https://developer.microsoft.com/office/dev-program) , um ein kostenloses Office 365-Abonnement zu erhalten.

> [!NOTE]
> Dieses Lernprogramm wurde mit Node Version 12.18.4 geschrieben. Die Schritte in diesem Leitfaden funktionieren möglicherweise mit anderen Versionen, jedoch nicht getestet.

## <a name="feedback"></a>Feedback

Geben Sie Feedback zu diesem Lernprogramm im [GitHub-Repository](https://github.com/microsoftgraph/msgraph-training-nodeexpressapp)an.
