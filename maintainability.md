# Maintainability

Maintainability beschrijft hoe gemakkelijk software in de tijd begrepen, aangepast en uitgebreid kan worden zonder onbedoelde gevolgen. Hieronder de kerneigenschappen — bewust beschreven op conceptueel niveau, los van specifieke technieken of frameworks.

## 1. Begrijpelijkheid (Understandability)
- Een lezer kan in korte tijd vatten wat een stuk code doet en waarom.
- De intentie van de code is duidelijk, niet alleen de werking.
- Nieuwe teamleden kunnen binnen redelijke tijd productief worden.

## 2. Leesbaarheid (Readability)
- Namen, structuur en opbouw vertellen het verhaal van de code.
- De lezer hoeft niet te raden of te reverse-engineeren om betekenis af te leiden.
- Visuele en logische hiërarchie sluiten op elkaar aan.

## 3. Voorspelbaarheid (Predictability)
- Vergelijkbare situaties zijn op vergelijkbare manieren opgelost.
- Het gedrag van een component komt overeen met wat de naam en context suggereren.
- Er zijn geen verrassende neveneffecten.

## 4. Wijzigbaarheid (Modifiability)
- Een aanpassing op één plek vereist geen kettingreactie van wijzigingen elders.
- De impact van een wijziging is af te bakenen en in te schatten.
- Code kan veilig worden uitgebreid zonder bestaand gedrag te breken.

## 5. Testbaarheid (Testability)
- Gedrag is te isoleren en te verifiëren.
- Het is duidelijk wat de "juiste" uitkomst is van een stuk code.
- Tests geven vertrouwen bij wijzigingen.

## 6. Vindbaarheid (Discoverability)
- Logica woont op een logische, te verwachten plek.
- Conventies maken het mogelijk om snel te vinden wat je zoekt.
- Er is geen verborgen kennis nodig om de structuur te begrijpen.

## 7. Consistentie
- Patronen en conventies worden door de hele codebase op dezelfde manier toegepast.
- Afwijkingen zijn bewust en uitlegbaar, niet toevallig.
- Stijl en structuur voelen aan als één hand.

## 8. Eenvoud (Simplicity)
- Een oplossing is niet complexer dan het probleem vereist.
- Abstracties bestaan omdat ze waarde toevoegen, niet voor de zekerheid.
- Er is geen ongebruikte of speculatieve functionaliteit.

## 9. Scheiding van verantwoordelijkheden (Separation of Concerns)
- Elk onderdeel heeft een duidelijke, afgebakende rol.
- Verschillende soorten beslissingen leven niet door elkaar heen.
- Het is duidelijk wáár iets thuishoort.

## 10. Lage koppeling, hoge cohesie
- Onderdelen weten zo min mogelijk van elkaars binnenkant.
- Wat bij elkaar hoort, staat bij elkaar.
- Een wijziging in het ene onderdeel hoeft een ander onderdeel niet te raken.

## 11. Robuustheid bij verandering
- De code is bestand tegen voorzienbare verandering (nieuwe eisen, nieuwe data, nieuwe schermen).
- Beslissingen zijn omkeerbaar; we hebben ons niet onnodig vastgelegd.
- Falen gebeurt op een herkenbare en hanteerbare manier.

## 12. Documentatie & context
- Niet-triviale beslissingen, aannames en beperkingen zijn vastgelegd waar ze relevant zijn.
- Documentatie beschrijft het *waarom*, de code beschrijft het *wat*.
- Verouderde documentatie wordt actief opgeruimd.

## 13. Debugbaarheid
- Bij een probleem is de oorzaak terug te leiden zonder de halve codebase te hoeven lezen.
- Het systeem geeft betekenisvolle signalen wanneer iets misgaat.
- De weg van symptoom naar oorzaak is kort.

## 14. Onboarding & overdraagbaarheid
- De codebase is niet afhankelijk van één persoon om begrepen te worden.
- Kennis zit in de code en de documentatie, niet alleen in hoofden.
- Een nieuw teamlid kan zelfstandig bijdragen zonder constante begeleiding.

## 15. Levensduur
- De code is geschreven met het besef dat hij vaker gelezen dan geschreven wordt.
- Korte-termijnoplossingen worden herkend als zodanig en niet stilzwijgend permanent.
- Technische schuld is zichtbaar, niet onzichtbaar.

---

**Werkdefinitie voor de skill:**
> Een stuk code of solution is *maintainable* als een ontwikkelaar die het niet zelf geschreven heeft, het binnen redelijke tijd kan begrijpen, veilig kan aanpassen en met vertrouwen kan uitbreiden — zonder afhankelijk te zijn van de oorspronkelijke auteur.
