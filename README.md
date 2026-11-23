# Crm

En liten webapp for kundebehandling (CRM). Dette er startpunktet for databasemodulen (Modul 2). Appen er den samme som løsningsforslaget til DI-oppgaven: kunder og aktiviteter kan lagres i JSON, XML eller i minnet. Oppsettet for Entity Framework Core (EF) mot SQL Server er på plass. Oppgaven står i Canvas.

## Hva den gjør

- **Kunder:** liste med søk og statusfilter, detaljside, opprett, endre og slett.
- **Aktiviteter:** registrer møter, samtaler, e-poster og oppgaver for en kunde, merk dem som ferdige, og se alle åpne aktiviteter i én oversikt. Forfalte aktiviteter er markert.
- **API (JSON):** alle endepunktene under `/api`, som beskrevet under.
- **Dependency injection (DI):** `ICustomerService` og `IActivityService` er registrert, og alle endepunktene får dem som parameter.
- **Database:** oppsettet for EF er på plass, men selve databasen er ikke laget ennå.

## Krav

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- Visual Studio (en nyere versjon, som støtter `.http`-filer), eller en annen editor sammen med kommandolinjeverktøyet `dotnet`
- SQL Server LocalDB, som installeres sammen med Visual Studio (arbeidsmengden **ASP.NET and web development** eller **Data storage and processing**)
- [SQL Server Management Studio (SSMS)](https://learn.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)

Sjekk at installasjonen fungerer:

```
dotnet --version
sqllocaldb info
```

`sqllocaldb info` skal vise `MSSQLLocalDB`.

## Lag din egen kopi

1. Klikk **Use this template** øverst på denne siden, og deretter **Create a new repository**.
2. Klone det nye repoet ditt:

```
git clone <url-to-your-repository>
```

## Kjør appen

**Visual Studio:** åpne `Crm.slnx` (eller `Crm.csproj`) og trykk `F5`. Nettleseren åpner <http://localhost:5080>.

**Kommandolinjen:** i mappen som inneholder `Crm.csproj`:

```
dotnet restore
dotnet run
```

Åpne deretter <http://localhost:5080>.

> **Start alltid appen fra prosjektmappen.** Maler og datafiler leses med relative stier, så sidene feiler hvis appen startes fra en annen mappe.

## API-et

Endepunktene under `/api` returnerer data som JSON, ikke HTML.

### Kunder

| Endepunkt | Beskrivelse | Svar |
|---|---|---|
| `GET /api/customers?search=&status=` | Alle kunder, eventuelt filtrert på tekst og status | 200 |
| `GET /api/customers/{id}` | Én kunde | 200, 404 |
| `POST /api/customers` | Opprett kunde | 201, 400 |
| `PUT /api/customers/{id}` | Oppdater kunde | 200, 404, 400 |
| `DELETE /api/customers/{id}` | Slett kunde og kundens aktiviteter | 204, 404 |

### Aktiviteter

| Endepunkt | Beskrivelse | Svar |
|---|---|---|
| `GET /api/customers/{id}/activities` | Aktivitetene til en kunde, nyeste først | 200, 404 |
| `POST /api/customers/{id}/activities` | Opprett aktivitet for en kunde | 201, 404, 400 |
| `GET /api/activities?open=true` | Alle aktiviteter, eller bare åpne med `open=true` | 200 |
| `PUT /api/activities/{id}/done` | Merk som ferdig eller åpen, med `{ "done": true }` | 204, 404 |
| `DELETE /api/activities/{id}` | Slett aktivitet | 204, 404 |

### Oppslagslister

| Endepunkt | Beskrivelse | Svar |
|---|---|---|
| `GET /api/statuses` | Gyldige kundestatuser | 200 |
| `GET /api/activity-types` | Gyldige aktivitetstyper | 200 |

### Valg i løsningen

- **Validering av kunde:** `name` kan ikke være tom, `email` må inneholde `@`, og `status` må være `Lead`, `Active` eller `Inactive`.
- **Validering av aktivitet:** `title` kan ikke være tom, og `type` må være `Meeting`, `Call`, `Email` eller `Task`. Mangler `date`, brukes nåværende tidspunkt.
- **Feilsvar:** Alle 400-svar har formatet `{ "error": "melding" }`.
- **201 Created:** Svaret fra `POST /api/customers` har en `Location`-header med adressen til den nye kunden. Svaret fra `POST /api/customers/{id}/activities` har ingen, siden API-et ikke har et endepunkt for å hente én aktivitet.
- **Sletting av kunde:** Aktivitetene til kunden slettes sammen med kunden.
- **Sortering:** `GET /api/customers` sorterer på navn.
- **JSON-navn:** Egenskapene i JSON starter med liten bokstav (`name`, `email`).

### Test med `.http`-filer

- `Crm.http` inneholder de tre første kallene fra API-videoen.
- `Assignment.http` inneholder alle kallene som tester løsningen, med forventet statuskode i kommentaren over hvert kall.

Åpne filen i Visual Studio, start appen, og trykk **Send request** over et kall. Kjør kallene ovenfra og ned. Noen kall bruker variabler øverst i filen (`@newCustomerId`, `@activityId` og `@tempCustomerId`). Sett dem til id-en i svaret fra opprett-kallet før du kjører de neste. Hvis appen kjører på en annen port enn 5080, endrer du `@host`.

Alle kallene skal gi de samme statuskodene og svarene uansett hvilken implementasjon som er registrert.

## Databasen i dette repoet

### Oppsett

- **Pakkene** `Microsoft.EntityFrameworkCore.SqlServer` og `Microsoft.EntityFrameworkCore.Tools` (versjon `10.*`) står i `Crm.csproj`.
- **Tilkoblingsstrengen** `Crm` står i `appsettings.json` og peker på databasen `Crm` i LocalDB.
- **`Data/CrmDbContext.cs`** beskriver tabellene `Customers` og `Activities`, med en fremmednøkkel fra `Activities.CustomerId` til `Customers.Id`. Sletter du en kunde, slettes aktivitetene til kunden også.
- **`Program.cs`** registrerer `CrmDbContext` med `AddDbContext`.

Appen bruker fortsatt JSON-filene. `CrmDbContext` registreres, men lages ikke før noe ber om den, så appen kjører også før databasen finnes.

### Bytt implementasjon

Øverst i `Program.cs` står én aktiv linje per interface. Flytt kommentaren `//` for å bytte:

```csharp
builder.Services.AddScoped<ICustomerService, JsonCustomerService>();
//builder.Services.AddScoped<ICustomerService, XmlCustomerService>();
//builder.Services.AddSingleton<ICustomerService, InMemoryCustomerService>();
```

### Koble til med SSMS

- **Server name:** `(localdb)\MSSQLLocalDB`
- **Authentication:** Windows Authentication

Databasen `Crm` vises under **Databases** etter `Update-Database`.

## Prosjektstruktur

```
Program.cs          Endepunkter, hjelpefunksjoner og registrering av tjenester (DI)
appsettings.json    Tilkoblingsstrengen til databasen
Data/               CrmDbContext: tabellene og koblingen mellom dem
Models/             Customer og Activity
Services/           ICustomerService og IActivityService, med JSON-, XML- og minneimplementasjoner
Templates/          HTML-maler med {{placeholders}}
wwwroot/            CSS og JavaScript
customers.json      Eksempelkunder i JSON-format (appen endrer denne filen)
customers.xml       Eksempelkunder i XML-format (appen endrer denne filen)
Crm.http            De tre første kallene fra API-videoen
Assignment.http     Alle kallene som tester API-et. Skal gi samme svar med alle implementasjonene
Log.md              Logg og designnotater som du fører underveis (del av leveransen)
TestResults.md      Resultatene fra kallene i Assignment.http (del av leveransen)
```

Mappen `Migrations` lages når du kjører `Add-Migration`. Den skal committes.

## Data

- `customers.json` og `customers.xml` inneholder fem eksempelkunder. Appen skriver til filen som hører til den registrerte implementasjonen. Slik gjenoppretter du eksempeldataene:

```
git checkout customers.json
git checkout customers.xml
```

- `activities.json` og `activities.xml` opprettes første gang du legger til en aktivitet med JSON- eller XML-versjonen. De følges ikke av Git.
- Versjonene i minnet nullstilles hver gang appen startes på nytt.
- Databasen ligger i LocalDB, utenfor prosjektmappen, og følges ikke av Git. Slik sletter du den og lager den på nytt, tom:

```
Drop-Database
Update-Database
```

Nullstill dataene før du kjører hele `Assignment.http` på nytt. Standardverdiene for variablene øverst i filen stemmer bare med ferske data, med kunde 1–5.

## Logg og testresultater

Repoet inneholder to filer som du fyller ut underveis. Begge er en del av leveransen, sammen med koden og `Assignment.http`.

- **`Log.md`:** før logg mens du jobber med oppgaven: feil du møter, og korte designnotater om valg du tar. Malene ligger i filen. Commit loggen sammen med koden.
- **`TestResults.md`:** før inn statuskodene du faktisk får for hvert kall i `Assignment.http`.

## Feilsøking

| Problem | Hva du bør sjekke |
|---|---|
| Feilen `NETSDK1004` (assets file not found) | Kjør `dotnet restore` i mappen med `Crm.csproj`, og bygg på nytt. |
| Feil eller blank side rett etter oppstart | Appen er startet fra feil mappe. Start den fra mappen med `Crm.csproj`. |
| Port 5080 er opptatt | Endre `applicationUrl` i `Properties/launchSettings.json`, og `@host` i `.http`-filene. |
| NuGet prøver å installere versjon 11 av EF-pakkene | Prosjektet bruker .NET 10. Velg nyeste 10.x, eller bruk `Version="10.*"` i `Crm.csproj`. |
| `Add-Migration` finner ikke `CrmDbContext` | `AddDbContext` mangler i `Program.cs`, eller prosjektet bygger ikke. Bygg og prøv igjen. |
| `Cannot open database "Crm"` | Databasen finnes ikke ennå. Kjør `Update-Database`. |
| Appen feiler med at en tjeneste ikke kan løses | Interfacet er ikke registrert med `builder.Services` i `Program.cs`, eller ingen linje for det interfacet er aktiv. |
| En kunde eller aktivitet forsvinner rett etter at den er opprettet | Versjonen i minnet er registrert med `AddScoped`. Bruk `AddSingleton`. |
| Et kall i `Assignment.http` gir 404 for en id som skulle finnes | Variablene øverst i filen peker på feil id, eller dataene er ikke nullstilt. |