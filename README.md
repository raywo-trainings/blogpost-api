# Blogpost API

API-First-Designprojekt (Teil eines API-Trainings), das eine **Blogpost API**
als OpenAPI-3.0.1-Dokument spezifiziert. Der vollständige Vertrag lebt in
[`blogposts.yaml`](blogposts.yaml) – es gibt keine Implementierung, keinen
Server, kein Build-System und keine Tests. Die Arbeit besteht im Entwurf der
API-Spezifikation selbst.

## Branches – Werdegang der API

`main` beschreibt bewusst nur den **Ausgangspunkt** der API (die reine
Blogpost-Ressource, siehe unten). Die einzelnen Erweiterungen aus dem Training
leben jeweils auf eigenen Branches und werden **absichtlich nicht** nach `main`
gemergt: So bleibt der schrittweise Werdegang der API nachvollziehbar, statt in
einem einzigen Endstand zu verschwinden.

Jeder Feature-Branch zweigt direkt von `main` ab; `all-together` führt die drei
fachlichen Erweiterungen zusammen, und `all-together-with-problemdetails` baut
darauf die Fehlerbehandlung auf.

| Branch                             | Baut auf       | Inhalt                                                                                                                                                                                                          |
|------------------------------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `main`                             | –              | Ausgangspunkt: die reine Blogpost-Ressource (Titel, Inhalt, CRUD).                                                                                                                                              |
| `authors`                          | `main`         | Autoren-Ressource (Benutzername, Name, Motto, Avatar mit versioniertem Cache-Busting); Blogposts erhalten eine verpflichtende `authorId`.                                                                       |
| `hashtags`                         | `main`         | Hashtags als kontrolliertes Vokabular, Zuordnung zu Blogposts und Filterung danach.                                                                                                                             |
| `ratings`                          | `main`         | Bewertungen (1–5 Sterne, optionaler Kommentar) als Subressource eines Blogposts, inkl. Aggregatfelder.                                                                                                          |
| `all-together`                     | die drei o. g. | Zusammenführung von `authors`, `hashtags` und `ratings` zu einer gemeinsamen Spezifikation.                                                                                                                     |
| `all-together-with-problemdetails` | `all-together` | Ergänzt eine einheitliche Fehlerbehandlung nach [RFC-9457](https://datatracker.ietf.org/doc/rfc9457/) (`ProblemDetail`, `application/problem+json`, Trennung von 400 und 422) und demonstriert `allOf`/`anyOf`. |

Wer den vollen Funktionsumfang sehen möchte, checkt den passenden Branch aus, z.
B.:

```sh
git switch all-together-with-problemdetails
```

## Fachlicher Überblick

Ein **Blogpost** besteht aus einem Titel (höchstens 50 Zeichen) und einem
Textinhalt. Zusätzlich führt der Server den Zeitpunkt der letzten Änderung
(`lastModified`) sowie eine eindeutige `id`; beide Felder werden vom Server
vergeben und dürfen beim Anlegen bzw. Ersetzen nicht mitgesendet werden.

## Endpunkte

Alle Pfade sind unter dem Server `https://localhost:8080` angesiedelt (ein
Platzhalter für eine künftige Implementierung, kein laufender Dienst).

| Methode | Pfad              | Beschreibung               | Erfolg | Fehler        |
|---------|-------------------|----------------------------|--------|---------------|
| GET     | `/blogposts`      | Alle Blogposts abrufen     | 200    | 500           |
| POST    | `/blogposts`      | Blogpost anlegen           | 201    | 400, 500      |
| GET     | `/blogposts/{id}` | Einzelnen Blogpost abrufen | 200    | 404, 500      |
| PUT     | `/blogposts/{id}` | Blogpost ersetzen          | 200    | 400, 404, 500 |
| DELETE  | `/blogposts/{id}` | Blogpost löschen           | 204    | 404, 500      |

Beim Anlegen (`POST`) liefert die Antwort einen `Location`-Header mit der URL
des neu angelegten Blogposts. Da sich beim Ersetzen (`PUT`) die Ressource
ändert, wird der aktualisierte Blogpost mit Status 200 zurückgegeben.

## Schemas

Die Schemas sind unter `components/schemas` definiert und werden per `$ref`
referenziert:

- **`BlogpostInput`** – vom Client bereitgestellte, änderbare Daten (`title`,
  `content`). Wird beim Anlegen und Ersetzen verwendet;
  `additionalProperties: false` weist unbekannte Felder ab.
- **`Blogpost`** – ein vollständiger Blogpost inklusive der servergenerierten
  Felder `id` und `lastModified` (beide `readOnly`).
- **`Error`** – standardisierte Fehlerantwort mit `status` und `message`.

### Fehlerbehandlung

Fehler werden über wiederverwendbare Responses unter `components/responses`
modelliert (`BadRequest` → 400, `NotFound` → 404, `InternalServerError` →

500) und geben jeweils das `Error`-Schema als `application/json` zurück.

## Arbeiten mit der Spezifikation

- `blogposts.yaml` ist die **einzige Quelle der Wahrheit** und folgt der
  Struktur von OpenAPI 3.0.1 (`info`, `servers`, `tags`, `paths`,
  `components`).
- Neue Endpunkte werden unter `paths` ergänzt. Wiederverwendbare Schemas,
  Parameter und Responses gehören unter `components` und werden per `$ref`
  eingebunden, statt Strukturen zu duplizieren.
- Das Dokument soll gültiges OpenAPI 3.0.1 bleiben. Vor dem Abschluss einer
  Änderung sollte geprüft werden, dass das YAML parst und der Spezifikation
  entspricht:

  ```sh
  # mit der Redocly CLI
  npx @redocly/cli lint blogposts.yaml

  # oder mit swagger-cli
  npx swagger-cli validate blogposts.yaml
  ```

## Projektstruktur

| Pfad             | Zweck                                                  |
|------------------|--------------------------------------------------------|
| `blogposts.yaml` | Die OpenAPI-3.0.1-Spezifikation (Quelle der Wahrheit). |
| `CLAUDE.md`      | Hinweise für die Arbeit mit diesem Repository.         |
| `.idea/`         | IntelliJ-IDEA-Projektdateien.                          |

Dies ist ein IntelliJ-IDEA-Projekt (`.idea/`, `BlogPost API.iml`); das Modul ist
ein allgemeines Modul ohne konfiguriertes Sprach-SDK.
