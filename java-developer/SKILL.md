---
name: java-developer
description: "Java-Entwicklungsrichtlinien und Best Practices. Laden wenn in Java-Projekten entwickelt wird: neue Features, Bugfixes, Refactoring, Tests oder Architekturentscheidungen."
---

## When to Use

Immer laden wenn in Java-Projekten entwickelt wird – egal ob neue Features, Bugfixes, Code-Reviews, Refactoring, Tests oder Architekturentscheidungen.

## Procedure

1. **Projektstruktur prüfen:** Maven (`pom.xml`) oder Gradle (`build.gradle`) erkennen und entsprechende Befehle verwenden (`mvn` vs `./gradlew`).
2. **Java-Version feststellen:** `java -version` oder in pom.xml/build.gradle nachsehen. API-Nutzung an die Zielversion anpassen (Records, Sealed Classes, Pattern Matching erst ab Java 16+/17+/21+).
3. **Code-Stil & Namenskonventionen strikt einhalten:**
   - Methoden und Felder: camelCase, beginnen mit **kleinem** Buchstaben (`betreuungHinzufuegen`, nicht `BetreuungHinzufuegen`)
   - Klassen und Interfaces: PascalCase (`FamilienService`)
   - Konstanten (`static final`): SCREAMING_SNAKE_CASE (`MAX_RETRY_COUNT`)
   - Paketnamen: komplett kleingeschrieben, keine Unterstriche
   - 4 Spaces Einrückung, Javadoc für public API
4. **Moderne Java-Features bevorzugen:** Records statt manueller POJOs, `var` für lokale Typinferenz, Stream API, Optional (nicht als Methodenparameter), Text Blocks für mehrzeilige Strings.
5. **Null-Safety:** `Optional` zurückgeben statt null, `@NonNull`/`@Nullable` Annotationen verwenden, defensive Checks mit `Objects.requireNonNull()`.
6. **Fehlerbehandlung:** Checked Exceptions sparsam einsetzen, eigene Exception-Hierarchien für Domänen-Fehler, nie leere catch-Blöcke, immer loggen oder rethrowen.
7. **Immutability bevorzugen:** `final` Felder, unmodifiable Collections (`List.of()`, `Map.of()`), Records für Value Objects.
8. **Tests:** JUnit 5 + Mockito als Standard. Tests nach AAA-Pattern (Arrange, Act, Assert). Testklassen mit `Test`-Suffix, Methodennamen beschreibend (`should_returnEmpty_when_listIsEmpty`).
9. **Dependency Injection:** Spring-Projekte: Constructor Injection bevorzugen (nicht `@Autowired` auf Felder). Beans so klein und fokussiert wie möglich halten.
10. **Build & Run:** `mvn clean verify` oder `./gradlew build` vor Commits ausführen. Checkstyle/SpotBugs-Warnungen ernst nehmen.
11. **Logging:** SLF4J als Abstraktion, Logback oder Log4j2 als Implementierung. Nie `System.out.println` in Produktionscode. Log-Level korrekt wählen (DEBUG, INFO, WARN, ERROR).
12. **Performance-Fallen vermeiden:** String-Konkatenation in Schleifen → StringBuilder; `HashMap` statt `Hashtable`; `ArrayList` statt `LinkedList` für Random Access.

## equals und hashCode

| Typ | equals/hashCode | Basis |
|---|---|---|
| **Value Object** | immer | alle Felder — besser: `record` verwenden, dann automatisch |
| **DDD-Entity / Aggregate** | immer | nur die ID — zwei Objekte mit gleicher ID sind semantisch identisch |
| **JPA-Entity (`@Entity`)** | immer | nur die ID — aber `hashCode()` muss eine Konstante zurückgeben (`return getClass().hashCode()`), weil die ID beim ersten `save()` noch `null` sein kann und ein wechselnder Hash ein Objekt im Set unauffindbar macht |
| **DTO / Service-Objekt** | nur wenn nötig | alle Felder — nur wenn explizit in Sets gesteckt oder mit `assertEquals` verglichen |

**Faustregel:** Hat das Objekt eine fachliche Identität (ID) → `equals`/`hashCode` auf Basis der ID. Hat es keine Identität (reine Daten) → auf Basis aller Felder, oder `record` verwenden.

## Pitfalls

- Nicht `@Autowired` auf private Felder verwenden – erschwert Testbarkeit; stattdessen Constructor Injection.
- Keine rohen Typen (`List` statt `List<String>`) – Generics immer explizit angeben.
- Keine `java.util.Date` oder `Calendar` mehr verwenden – stattdessen `java.time.*` (LocalDate, LocalDateTime, ZonedDateTime).
- `Optional.get()` ohne `isPresent()`-Check ist ein Anti-Pattern – `map()`, `orElse()`, `orElseThrow()` bevorzugen.
- Nie `Exception` oder `Throwable` pauschal catchen, außer an der äußersten Grenze (z.B. global error handler).
- Static-Imports sparsam einsetzen – nur wenn der Name eindeutig und bekannt ist (z.B. `Assertions.*` in Tests).
- Keine `Thread.sleep()` in Tests – stattdessen Awaitility oder Mocking verwenden.
- Gradle Wrapper (`./gradlew`) statt globalem `gradle` verwenden, um Build-Reproduzierbarkeit sicherzustellen.
- **Enum-Felder in DAOs immer mit `@Enumerated(EnumType.STRING)` annotieren:** Enum-Felder in JPA-Entities niemals als `String` speichern und manuell per `.name()`/`.valueOf()` konvertieren — stattdessen den Feldtyp direkt als Enum deklarieren und `@Enumerated(EnumType.STRING)` setzen, damit JPA die Konvertierung übernimmt. Manuelles `.valueOf()` ist nur an Systemgrenzen erlaubt (z.B. Request-DTOs aus User-Input).
- **Repository-Adapter nie als eigene Klasse implementieren:** Wenn ein Domain-Port-Interface (`*Repository`) und ein Spring Data JPA Interface (`Spring*Repository extends JpaRepository`) parallel existieren und durch eine manuelle `*RepositoryImpl`-Klasse verbunden werden, ist das Boilerplate — stattdessen ein einziges `*JpaRepository extends JpaRepository<DAO, UUID>, *Repository` erstellen, das den Domain-Port via `default`-Methoden direkt implementiert; Spring Data generiert den Rest automatisch.
- Kein intermediäres `Set`/`List` nur für Membership-Check: Wenn eine Collection ausschließlich für `.contains()` gesammelt wird, stattdessen direkt `stream().anyMatch(...)` auf der Quelle verwenden — kein unnötiger Overhead, klarere Intention.
- **DAOs mit vielen Feldern immer mit Builder Pattern implementieren:** Einen inneren `Builder`-Static-Class mit fluent Setter-Methoden erstellen und `fromAggregate`/`fromDomain` darüber aufbauen — kein Konstruktor mit vielen Parametern, keine post-construction Feldzuweisungen (`dao.field = ...`). Getter/Setter nur hinzufügen wenn sie tatsächlich von außen verwendet werden.
- Transaktionale Methoden, die ausschließlich Lesezugriffe durchführen, mit `@Transactional(readOnly = true)` annotieren — spart dem Datenbankoptimizer unnötige Schreibsperren.
- Smarte JDK-Utility-Methoden nutzen statt manueller Null-Checks: `Objects.requireNonNullElse()`, `Objects.requireNonNullElseGet()`, `Objects.toString(obj, fallback)` u.ä.
- Standard-JavaBean-Getter verwenden (`getName()`, `getEmail()`) — keine record-style Accessors (`name()`, `email()`) in normalen Klassen.
- **Keine Magic Strings:** Literal-Strings, die fachliche Werte repräsentieren (z.B. Rollennamen wie `"ErkranktesKind"`), niemals direkt in Vergleichen oder Logik verwenden — stattdessen als `public static final String`-Konstante in der Klasse definieren, die den Typ trägt (z.B. im zugehörigen Record oder DTO), und ausschließlich über diese Konstante referenzieren. Wenn ein passender Enum existiert, diesen bevorzugen; ist der Typ an dieser Stelle jedoch zwingend `String` (z.B. in einem modul-übergreifenden API-Record), gehört die Konstante in genau diesen Record.
- **Domain-Services immer als Bean injizieren, nie mit `new` instanziieren:** Domain-Service-Implementierungen (`*ServiceImpl`) werden ausschließlich in der `@Configuration`-Klasse des Moduls als `@Bean` erzeugt und per Konstruktor-Injection eingebracht — niemals `new BetreuungenServiceImpl(...)` o.ä. direkt im Application-Service-Konstruktor aufrufen.
- `@Repository` auf Spring Data JPA-Interfaces (die `JpaRepository` erweitern) ist redundant — Spring Data JPA erkennt und registriert diese automatisch via `@EnableJpaRepositories`. Die Annotation nur auf eigenen Implementierungsklassen verwenden, die nicht automatisch gescannt werden.
- `@Service` und `@RestController` nie auf derselben Klasse – Service gibt Domain-Typen zurück, Controller wrapped in `ResponseEntity`.
- Wenn MapStruct im Projekt verwendet wird (`mapstruct` in pom.xml/build.gradle), Objekt-Mapping immer mit MapStruct umsetzen statt manuell.

## Verification

- `mvn clean verify` oder `./gradlew build` läuft ohne Fehler durch.
- Alle neuen public Klassen/Methoden haben Javadoc-Kommentare.
- Neue Logik ist durch Unit-Tests abgedeckt.
- Keine `System.out.println`-Aufrufe im Produktionscode.
- Keine ungenutzten Imports oder Variablen.
