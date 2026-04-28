# Maintainability Skill — Input & Specificaties

> Basis voor het bouwen van een skill (op basis van `skill-manager`) die reeds geschreven code toetst op maintainability.
> Gebaseerd op `input.txt` en `maintainability.md`.

---

## Context Summary

### Idee
Een skill (gebaseerd op `skill-manager`) die reeds geschreven code toetst op maintainability. De skill accepteert een bestand, een folder of de hele solution als input en levert een markdown-rapport met aandachtspunten plus een samenvatting voor de gebruiker. De skill is framework-agnostisch en detecteert zelf de gebruikte technieken bij activatie.

### Probleem
Niet besproken — de gebruiker heeft motivatie expliciet als n.v.t. aangemerkt.

### Doelgroep
Ontwikkelaars die met Claude Code werken.

### Scope
- **In scope**: maintainability zoals gedefinieerd in `maintainability.md`; intern onderzoek naar bestaande conventies in de codebase; extern onderzoek naar best practices van de gedetecteerde technieken (max. 2 jaar oud); analyse van bestanden waar ontwikkelaars in werken.
- **Out of scope**: andere kwaliteitsaspecten (security, performance, bugs/correctheid, accessibility, testdekking); externe dependencies (zoals `node_modules`); vooraf vastleggen van best-practice-kennis in de skill.

### Context & omgeving
- Techniek is vooraf onbekend; de skill onderzoekt zelf welke technieken worden gebruikt in de scope.
- Per run worden twee parallelle onderzoeken gedaan:
  1. **Intern**: wat zijn de bestaande practices/conventies in deze codebase?
  2. **Extern**: wat zijn de huidige web best practices voor de gedetecteerde technieken?
- Bij conflict tussen intern en extern wordt het externe patroon voorgesteld met een duidelijke onderbouwing én impactanalyse op de solution.

### Constraints
- Alleen developer-bestanden worden geanalyseerd (geen externe dependencies).
- Maximale doorlooptijd per run: 1 uur.
- Skill moet portable zijn — daarom geen vastgelegde best-practice-kennis, maar live research per run.

### Dependencies
- `skill-manager` als basis voor de skill.
- Live web-toegang tijdens runtime (voor het externe onderzoek).
- `maintainability.md` als referentie/definitie van wat getoetst wordt.
- `input.txt` als basis voor het skill-contract en de uitgangspunten.

### Succescriteria
- De bevindingen worden door de ontwikkelaar als **plausibel** ervaren.

### Risico's & onzekerheden
- **Datalek**: de skill kan tijdens scannen privégegevens (secrets, env-files, persoonsgegevens) inzien, met als bijkomend risico dat deze via de live web-research naar buiten lekken.

### Open vragen
- Welke concrete tools en subskills heeft de skill nodig? (in `input.txt` nog placeholders `x x x`).
- Hoe wordt het datalek-risico operationeel gemitigeerd?

---

## Functional requirements

| #     | Requirement                                                                                                                            | Prioriteit |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| FR-1  | Skill accepteert `@file`, `@folder` of de hele solution als input.                                                                     | must       |
| FR-2  | Skill detecteert bij activatie zelf welke technieken in de scope worden gebruikt.                                                       | must       |
| FR-3  | Skill onderzoekt de bestaande conventies/practices binnen de scope (intern onderzoek).                                                  | must       |
| FR-4  | Skill voert per run live web-research uit naar best practices van de gedetecteerde technieken (max. 2 jaar oud).                        | must       |
| FR-5  | Skill genereert een markdown-rapport met aandachtspunten — per punt: bestand, regel, reden.                                             | must       |
| FR-6  | Skill toont een samenvatting aan de gebruiker.                                                                                          | must       |
| FR-7  | Bij conflict tussen interne en externe practice stelt de skill het externe patroon voor, met duiding én impactanalyse op de solution.   | must       |
| FR-8  | Skill negeert externe dependencies (zoals `node_modules`) en alle niet-developer bestanden.                                             | must       |
| FR-9  | Skill toetst uitsluitend op maintainability — andere kwaliteitsaspecten worden expliciet genegeerd.                                     | must       |

## Non-functional requirements

| #      | Requirement                                                                              | Categorie     |
| ------ | ---------------------------------------------------------------------------------------- | ------------- |
| NFR-1  | Maximale doorlooptijd per run: 1 uur.                                                    | performance   |
| NFR-2  | Skill is framework-/techniek-agnostisch.                                                  | portability   |
| NFR-3  | Geen vastgelegde best-practice-kennis in de skill; altijd live ophalen per run.          | portability   |
| NFR-4  | Bevindingen moeten als plausibel worden ervaren door de ontwikkelaar.                    | usability     |

## Aannames
- Live web-research levert binnen redelijke tijd voldoende kwalitatieve bronnen op.
- Detectie van de gebruikte techniek(en) werkt ook in mixed-tech / monorepo-situaties.
- Intern onderzoek + extern onderzoek + analyse + rapportage past binnen het tijdsbudget van 1 uur, ook bij grote solutions.
- De definitie in `maintainability.md` is leidend voor wat onder "maintainability" valt.
- De gebruiker accepteert dat externe best practices prevaleren boven interne consistentie bij conflict.

## Dependencies

| #    | Dependency                                            | Type            | Status                  |
| ---- | ----------------------------------------------------- | --------------- | ----------------------- |
| D-1  | `skill-manager` als basis voor de skill                | technisch       | known                   |
| D-2  | Web-toegang tijdens runtime (live research)            | technisch       | known                   |
| D-3  | `maintainability.md` als referentiedefinitie           | organisatorisch | known                   |
| D-4  | `input.txt` als basis voor contract en uitgangspunten | organisatorisch | known                   |
| D-5  | Concrete tools/subskills binnen de skill               | technisch       | assumed (nog onbekend)  |

## Risico's

| #    | Risico                                                                                                  | Kans     | Impact | Mitigatie          |
| ---- | ------------------------------------------------------------------------------------------------------- | -------- | ------ | ------------------ |
| R-1  | Datalek: skill ziet privégegevens (secrets, env-files, persoonsgegevens) en lekt deze via live research. | onbekend | hoog   | nog niet besproken |
