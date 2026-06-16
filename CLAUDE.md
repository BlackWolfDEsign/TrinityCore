# CLAUDE.md — Projekt-Charter: WotLK-Classic Custom Server

## Ziel
Privater WoW-Server auf **TrinityCore `wotlk_classic` (Client 3.4.x)** mit:
- moderner Grafik (moderne Engine, eingefrorener 3.4-Build)
- **WotLK-Talentbäumen** (auf 3.4 nativ vorhanden)
- Retail-artigem **Transmog-Interface**

Optional/portierbar (nice-to-have, zuletzt): Transmog-Item-Skins, neue Welten/Instanzen/Dungeons aus Retail.

## Basis-Entscheidung (begründet)
Die drei Must-haves haben genau **einen** gemeinsamen Schnittpunkt:
- „schöne Grafik" → moderne Engine → schließt 3.3.5a aus
- „WotLK-Talentbäume" → auf master nicht vorhanden (Großprojekt), auf 3.4 nativ
- „Transmog like retail" → braucht modernen Client

→ **Basis = `wotlk_classic` / 3.4.x.** master scheitert an Talenten, 3.3.5a an Grafik+UI.

## Kern-Strategie: 3.3.5 ist die Content-Quelle
Der `wotlk_classic`-Branch ist **content-unreif**. Die ausgereiften WotLK-Inhalte (Quests, Skripte, SmartAI, DB-Fixes aus ~15 Jahren) liegen auf **3.3.5**. Beide tragen **dieselbe WotLK-Inhaltsmenge** → 3.3.5 wird systematisch nach `wotlk_classic` forward-portiert. Präzedenz: TC macht genau solche 3.3.5→neuerer-Branch-Cherry-Picks selbst (Issue #15077). Vorteil hier: gleiche Expansion → IDs fluchten weitgehend (statt Neu-Mapping wie bei master→3.4), **aber Fluchtung muss verifiziert werden** (Re-Release-Reindexierung).

Der Backport hat zwei Hälften:
- **Datengetrieben (Bulk):** Quests, Spawns, Loot, Gossip, SmartAI (World-DB). Migrierbar über Schema-Mapping (TDB 335.x → wotlk_classic-TDB) + ID-Validierung. Tabellenweise, skriptbar.
- **C++ (Handarbeit):** Boss-/Instanz-/Creature-Scripts. Alte Script-API → master-Linien-API. Neu schreiben, skaliert mit Encounter-Zahl. Schwerster Teil.

## Architektur (Soll)
- **bnetserver** (moderner Login) + **worldserver** — auf `wotlk_classic` verifizieren
- **Client: vorhandener 3.4.3** (westlich, Locale deDE/enUS, in-hand). Build-gepinnt → exakten `wotlk_classic`-Zielbuild verifizieren und 3.4.3-Build dagegen abgleichen
- DBs: `auth`/bnet, `characters`, `world` — TC-Schema des `wotlk_classic`-Branch
- Client-Daten: Extraktion aus dem 3.4.3-Client (CASC, DB2) **mit Branch-eigenen Extraktoren** — nie mischen
- Client-Connect: **Arctium-Launcher** (umgeht CASC-Signaturprüfung; für moderne Branches Pflicht)

## Harte Constraints
1. **Engine gepinnt.** 3.4 = eingefrorener älterer Build. Post-Freeze-Retail-Features rendern nicht → Feature-Gate-Check pro Port-Asset.
2. **DB2-/Map-/World-DB-Schemata sind versionsspezifisch.** Kein Drag-and-Drop. Schema-Mapping + ID-Validierung nötig.
3. **Core braucht Handler.** Portierter Content ohne passende C++-Logik ist tot. C++-Scripts = Neuentwicklung gegen die neue API.
4. **Kein Ökosystem auf 3.4.** Kein Eluna, keine NPCBots, keine M+/Transmog-Module (alle 3.3.5a). Alles Custom in C++.
5. **Core unsupported.** Baseline-Stabilität unbestätigt → Phase 0 ist Go/No-Go-Gate.
6. **Assets = Blizzard-IP.** Legit-Client (3.4.3 in-hand) nötig; reines Privat-/Eigennutzungs-Projekt. Kein 3.4.4 (China-exklusiv, abgelöst durch Titan Reforged 3.80.x).

## Konventionen
- Code-Sprache: C++ (Core), SQL (DB). Kein Eluna.
- Kommentare in Files/Functions: kurz und prägnant, nur wo nicht selbsterklärend.
- Custom/Port-Änderungen: abgegrenzte, dokumentierte Patches; Core-Eingriffe minimal und markiert.
- Jede Migrations-Charge: Quell-TDB/-Commit + Mapping-Schritt + ID-Validierungsstatus dokumentieren.
- Branch-Hygiene: Custom-Arbeit auf eigenem Branch über `wotlk_classic`, regelmäßig rebasen.
- **DB nie ins Git:** TDB/Dumps bleiben lokal (MySQL + Datei auf Platte); `.gitignore` für `*.sql`-Dumps / TDB-Verzeichnis. Agents greifen über die lokale MySQL-Verbindung zu. TDB-Version pro Branch zum Core-Commit pinnen (`version`-Tabelle), nie cross-branch mischen.

## Agent-Splitting (Empfehlung)
Phase 0 seriell und gatekeepend. Danach parallelisieren; der Backport (P1) ist der dominante Block und wird als Team geführt.
- **Agent A — Core/Infra:** Build, DB-Setup, Build-Match + 3.4.3-Extraktion, Arctium-Connect, Bare-Core-Boot. Liefert Go/No-Go.
- **Team B — Content-Backport (P1, größter Block):**
  - **B1 — Daten/DB:** Schema-Mapping 335.x→wotlk_classic, ID-Alignment-Validierung, Bulk-Migration Quests/Spawns/Loot/Gossip/SmartAI.
  - **B2 — C++-Scripts:** 3.3.5-Boss-/Instanz-/Creature-Scripts gegen neue API neu schreiben.
  - **B3 — Spells/Talente:** Spell-Effekt-Korrektheit gegen 3.3.5 prüfen/angleichen.
- **Agent C — Transmog (P2):** komplett custom auf 3.4.3 — Server-Handler, Appearance-Collection, Client-UI ergänzen.
- **Agent D — Retail-Teilübernahme (P3):** Konvertierungs-Pipeline; Item-Skins → statische Zonen → Dungeons.
- **Human/Coordinator:** Integration + In-Game-Test. Nur ein Mensch am Login validiert Feel/Grafik/Content.

## Definition of Done (v1)
Server stabil, Login via Arctium, Welt auf annähernd 3.3.5-Reifegrad spielbar (Quests/Skripte/Spells), WotLK-Talente korrekt, Transmog-UI im Retail-Stil nutzbar. Retail-Zusatz-Content ist v1 **nicht** Bedingung.
