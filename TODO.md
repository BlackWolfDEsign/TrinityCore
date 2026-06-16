# TODO.md — WotLK-Classic Custom Server

Phasen durch Gates getrennt. **Phase 0** entscheidet, ob der Core tragfähig ist. **Phase 1** (Content-Backport) ist der größte Block und liefert den eigentlichen Spielwert. Erst nach P0-Gate parallelisieren.

## Phase 0 — Bare-Core-Boot (Go/No-Go) · Agent A
- [ ] `wotlk_classic`-Branch klonen, Abhängigkeiten, sauberer Build
- [ ] **Branch-Build-String verifizieren** (welchen exakten 3.4.x-Build erwartet `wotlk_classic`?) und gegen vorhandenen **3.4.3**-Client abgleichen; bei Mismatch passenden Build besorgen
- [ ] MySQL/MariaDB: `auth`/bnet, `characters`, `world` aufsetzen, Branch-TDB importieren
- [ ] Client-Daten aus 3.4.3 mit **Branch-eigenen** Extraktoren ziehen (maps/vmaps/mmaps/DB2)
- [ ] bnetserver + worldserver starten, Account anlegen
- [ ] Arctium-Connect, Login, Charakter erstellen, in der Welt bewegen
- [ ] **Gate:** Core stabil genug zum Draufbauen? Nein → Stop/Strategiewechsel.

## Phase 1 — Content-Audit & Backport 3.3.5 → wotlk_classic · Team B
> Gleiche WotLK-Inhaltsmenge auf beiden Branches → forward-porten, nicht neu bauen.

### 1.0 Mapping-Fundament (Tor vor Bulk) · B1
- [ ] Schema-Diff World-DB: TDB 335.x ↔ wotlk_classic-TDB (Tabellen/Spalten)
- [ ] **ID-Alignment-Validierung** (Spell/Item/Quest/Map/Creature): fluchten 3.3.5- und 3.4-IDs? Abweichungen aus Re-Release-Reindexierung katalogisieren
- [ ] Migrations-/Mapping-Tooling aufsetzen (wiederholbar, reversibel)

### 1.1 Datengetriebener Bulk-Backport · B1
- [ ] Quests (`quest_template` & Verwandte) migrieren + validieren
- [ ] Creature/GameObject-Spawns + Templates
- [ ] Loot-Tabellen
- [ ] Gossip / Menüs
- [ ] **SmartAI** (`smart_scripts`) — Schema-Anpassung + Migration
- [ ] Stichproben-Verifikation pro Charge in-game

### 1.2 C++-Script-Portierung · B2
- [ ] Inventur: welche Boss-/Instanz-/Creature-Scripts auf 3.3.5 vorhanden vs. wotlk_classic
- [ ] Scripts gegen master-Linien-Script-API neu schreiben (priorisiert: Leveling-Pfad → Dungeons → Raids)
- [ ] Pro portiertem Script in-game testen

### 1.3 Spells/Talente · B3
- [ ] WotLK-Talente: lernen/zurücksetzen funktioniert nativ
- [ ] Spell-Effekt-Korrektheit gegen 3.3.5 abgleichen (Funktion für Funktion), Lücken/Veraltetes fixen

## Phase 2 — Transmog „like retail" (komplett custom auf 3.4.3) · Agent C
- [ ] Server-Handler + Persistenz (`characters`)
- [ ] Appearance-Collection serverseitig
- [ ] **Client-UI ergänzen** (3.4.3 hat kein natives Transmog-UI; 3.4.4 verworfen)
- [ ] Test mit nativen 3.4-Item-Appearances

## Phase 3 — Retail-Teilübernahme (portierbarer Zusatz) · Agent D
- [ ] Item-Skins: Pipeline 12.x-DB2 → 3.4-Schema + M2-Modelle + Feature-Gate-Check; Pilot-Charge
- [ ] Statische Retail-Zone: ADT/WMO/M2 konvertieren, Map/VMap/MMap regenerieren, Spawns
- [ ] Instanz/Dungeon: Server-Encounter-Scripts in C++ von Grund auf (höchstes Risiko, v1+)

---

## Offene Punkte · Lücken · Risiken · Inkonsistenzen
- **R0 (existenziell):** `wotlk_classic` unsupported/unfertig. Baseline unbestätigt → P0-Gate ist echtes Stop-Kriterium.
- **R1:** Kein Eluna auf 3.4 → alle Logik in C++. Langsamere Iteration.
- **R2:** Kein reifes Ökosystem (M+, Bots, KI-NPCs) → v1 out of scope; später volle Eigenentwicklung. (Früherer Wunsch M+/AI-NPCs bewusst zurückgestellt.)
- **R3:** Engine gepinnt — Post-3.4-Freeze-Retail-Features rendern nicht. Pro Port-Asset Gate-Check.
- **R4:** Jedes Retail-Port-Asset = manuelle 12.x→3.4-Konvertierung. Aufwand skaliert mit Menge.
- **R5 (GELÖST):** Transmog-Client entschieden — vorhandener **3.4.3 + Transmog komplett custom** (Server-Handler + Client-UI). 3.4.4-China verworfen (China-exklusiv, abgelöst durch Titan Reforged 3.80.x, regionsgesperrt, zhCN).
- **R6:** Legit-Client + Blizzard-IP. Privat-/Eigennutzung.
- **R7 (Tor P1):** ID-Fluchtung 3.3.5↔3.4 darf nicht angenommen werden — Re-Release reindexierte stellenweise. Validierung vor Bulk-Migration zwingend.
- **R8 (Hauptaufwand P1):** C++-Script-Portierung (B2) skaliert mit Encounter-Zahl und ist nicht kompilier-kompatibel — realistisch der zeitintensivste Posten.
- **R9 (Client, weitgehend entschärft):** 3.4.3 in-hand, Locale deDE/enUS korrekt → Beschaffung kein Problem. Residual: **exakter Build-Match gegen `wotlk_classic`** (siehe P0).
- **KORREKTUR:** Frühere Annahme „WotLK-Content/Talente auf 3.4 nativ/gratis" war falsch. Struktur/UI nativ, aber Content- und Spell-Reife liegen auf 3.3.5 → daher Phase 1.
- **GELÖST:** „Talent-Co-Existenz" und „Pseudo-Bäume DH/Evoker" (master-basiert) gestrichen — auf 3.4 Talente nativ, kein DH/Evoker. Nicht wieder aufnehmen.

## Nächste Aktion
1. Agent A startet P0, zuerst Build-Match (3.4.3 ↔ Branch-Zielbuild). 2. Team B beginnt mit 1.0 (Schema-Diff + ID-Validierung), sobald P0-Gate grün.
