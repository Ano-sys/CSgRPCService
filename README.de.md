# CS gRPC Service – RPC mit C# (Client/Server)

Dieses Repository zeigt eine einfache **Remote Procedure Call (RPC)**-Kommunikation mit **gRPC** in C#:

- `csgrpcserver`: Stellt RPC-Methoden bereit
- `csgrpcclient`: Ruft die Methoden über das Netzwerk auf
- `Protos/SimpleCommunication.proto`: Vertrag (Interface) zwischen Client und Server

## Ziel des Projekts

Das Projekt demonstriert, wie man Funktionsaufrufe zwischen zwei Prozessen/Anwendungen so gestaltet, als wären es lokale Methodenaufrufe.

Statt nur Bytes über eine Socket-Verbindung zu schicken, werden bei gRPC:

- Schnittstellen klar definiert,
- Datentypen serialisiert,
- und Aufrufe stark typisiert zwischen Client und Server umgesetzt.

---

## Was ist RPC – einfach erklärt

**RPC (Remote Procedure Call)** bedeutet:

> Eine Funktion wird in einem anderen Prozess (oft auf einem anderen Rechner) aufgerufen, als wäre sie lokal.

Der Aufruf ist technisch ein Netzwerk-Request, aber aus Entwicklersicht ähnelt er einem normalen Methodenaufruf:

1. Client ruft Methode auf (z. B. `GetMessage(...)`)
2. Anfrage wird serialisiert und über Netzwerk gesendet
3. Server führt Methode aus
4. Antwort wird serialisiert zurückgesendet
5. Client erhält das Ergebnis

### Warum gRPC dafür oft gewählt wird

- basiert auf HTTP/2
- nutzt Protocol Buffers (kompakt, schnell)
- stark typisierte Verträge über `.proto`
- automatische Code-Generierung für Client/Server-Stubs

---

## Wozu ist RPC gut?

RPC ist besonders sinnvoll, wenn:

1. **Anwendungslogik getrennt betrieben** werden soll (Client und Service getrennt deploybar)
2. **Mehrere Clients** (Desktop, Web, Backend) denselben Service nutzen sollen
3. **Skalierung** wichtig ist (Server separat horizontal skalierbar)
4. **Klare API-Verträge** benötigt werden (Versionierung über Protos)
5. **Sprachübergreifende Kommunikation** relevant ist (z. B. C#, Go, Java, Python)
6. **Interne Service-Kommunikation** in verteilten Systemen gebraucht wird

Typische Einsatzbereiche:

- Microservices
- zentrale Business-Services
- interne Unternehmens-APIs
- performante Service-zu-Service Kommunikation

---

## Vergleich: RPC vs. reines TCP vs. lokaler Call/Return (DLL oder externer Prozess)

## 1) RPC (z. B. gRPC)

### Vorteile

- **Abstraktion auf Methodenebene** statt Byte-Protokoll von Hand
- **Stark typisiert** durch `.proto`-Verträge
- **Automatische Code-Generierung** reduziert Boilerplate
- **Gute Interoperabilität** zwischen Sprachen/Plattformen
- **Skalierbar und deploybar** als eigener Service
- **Einheitlicher Vertrag** für alle Clients

### Nachteile

- **Netzwerkabhängigkeit** (Latenz, Timeouts, Ausfälle)
- **Komplexeres Betriebsmodell** als reine In-Process-Aufrufe
- **Versionierungsdisziplin nötig** (Breaking Changes vermeiden)
- **Debugging verteilt** oft aufwendiger als lokaler Code

---

## 2) Reines TCP (Sockets, eigenes Protokoll)

### Vorteile

- **Maximale Kontrolle** über Protokoll und Datenfluss
- **Sehr flexibel** für Spezialfälle
- Potenziell **sehr effizient**, wenn sauber implementiert

### Nachteile

- **Hoher Entwicklungsaufwand** (Framing, Serialisierung, Fehlerfälle)
- **Mehr Fehleranfälligkeit** durch eigenes Protokolldesign
- **Schlechte Wartbarkeit**, wenn Spezifikation nicht strikt gepflegt wird
- Kein standardisierter Stub/Contract-Workflow wie bei gRPC

### Wann sinnvoll?

- Wenn ein proprietäres Low-Level-Protokoll zwingend ist
- Wenn extrem spezielle Transportanforderungen bestehen

---

## 3) Normaler Call/Return lokal (DLL einbinden)

Damit ist gemeint: Das C#-Programm bindet eine DLL ein und ruft Funktionen direkt im selben Prozess auf.

### Vorteile

- **Sehr schnell** (kein Netzwerk, keine externe Serialisierung)
- **Einfaches Debugging** im selben Prozess
- **Geringe Laufzeitkomplexität**

### Nachteile

- **Starke Kopplung** zwischen aufrufender App und Bibliothek
- **Deployment-Kopplung** (Versionen müssen zusammenpassen)
- **Schwächere Isolation**: Fehler/Crash in DLL kann den gesamten Prozess betreffen
- **Skalierung begrenzt** auf den lokalen Prozess/Host

### Wann sinnvoll?

- Für lokale, performante Funktionsbibliotheken
- Wenn keine Prozess- oder Rechnertrennung nötig ist

---

## 4) Externe Programme per Call-and-Return (Prozess starten, Ergebnis abholen)

Damit ist gemeint: Das Hauptprogramm startet ein separates Tool/Programm, übergibt Parameter, wartet auf Rückgabe (Exit-Code, Output, Datei).

### Vorteile

- **Starke Isolation** zwischen Hauptprozess und Tool
- Kann vorhandene große Tools wiederverwenden
- Fehler im Tool beeinträchtigen den Hostprozess meist weniger direkt

### Nachteile

- **Startkosten pro Aufruf** (Prozessstart ist teuer)
- **Umständliche Datenübergabe** (Argumente, StdOut, Dateien, Pipes)
- **Begrenzte Interaktivität** für viele kleine Aufrufe
- **Betrieb/Monitoring** komplex bei vielen Tool-Aufrufen

### Wann sinnvoll?

- Bei seltenen, schweren Batch-Aufgaben
- Wenn bestehende CLI-Tools integriert werden müssen

---

## Kurzfazit: Welche Technik wann?

- **gRPC/RPC**: Beste Wahl für stabile, typisierte Kommunikation zwischen getrennten Anwendungen/Services.
- **Reines TCP**: Nur dann, wenn man bewusst ein eigenes Protokoll und volle Low-Level-Kontrolle braucht.
- **DLL lokal**: Beste Wahl für maximale lokale Performance und enge Integration im selben Prozess.
- **Externer Prozessaufruf**: Gut für lose gekoppelte Tool-Integration und isolierte Batch-Schritte.

---

## Entscheidungsleitfaden (praktisch)

Frage dich bei neuen Features:

1. Muss die Funktion **über Netzwerk/zwischen Prozessen** nutzbar sein? → eher **RPC**
2. Muss es **im selben Prozess extrem schnell** laufen? → eher **DLL**
3. Muss ein bestehendes großes Tool nur gelegentlich genutzt werden? → eher **externer Prozess**
4. Braucht ihr ein **eigenes Binärprotokoll mit Spezialanforderungen**? → ggf. **TCP direkt**

---

## Projektstruktur

- `csgrpcclient/CSgRPCClient/Program.cs`: Beispiel-Client
- `csgrpcserver/CSgRPCServer/Program.cs`: Beispiel-Server
- `Protos/SimpleCommunication.proto`: RPC-Vertrag

---

## Praktischer Hinweis zu Fehlerbehandlung bei RPC

Im Gegensatz zu lokalem Funktionsaufruf sollten bei RPC immer mitgedacht werden:

- Timeouts/Deadlines
- Retries (kontrolliert)
- Idempotenz
- saubere Fehlercodes
- Telemetrie/Logging für verteilte Fehleranalyse

Gerade diese Punkte sind ein Kernunterschied zwischen „lokalem Call“ und „Remote Call“.

---

## Zusammenfassung in einem Satz

RPC ist ideal, wenn Funktionen **als Service** bereitgestellt werden sollen; lokale DLL-Aufrufe sind ideal für **in-process Performance**, und externe Prozessaufrufe eignen sich für **lose gekoppelte Tool-Integration** mit höherem Laufzeit-Overhead.
