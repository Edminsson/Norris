# Norris – 3D-modell av hemmet

Webbapp som körs i webbläsaren där man kan **vandra runt i huset i 3D**, **prova olika möbleringar** och **prova olika renoveringar** (flytta/ta bort väggar, byta golv, färg, kök m.m.). Repo: https://github.com/Edminsson/Norris (publikt).

## Huset (underlag)

Underlag (planskisser + foton) finns i `../bilder/`, **utanför repot**. Repot är publikt – bilderna får aldrig committas eller kopieras in i repot. Läs dem innan du ändrar husmodellen.

- **Typ:** enplans kedje-/atriumhus i tegel med platt tak, plus **källare**. Uterum i glas mot trädgården. Trädgården omges av tegelmur och häckar.
- **Entréplan** (`../bilder/SkissOvanvåning.jpeg`): entré (pil, framsida nertill), vardagsrum, kök (spis/kyl/frys), tvätt (TM), WC (V), badrum med badkar, 3 sovrum, garderober (G/L), soprum (Sop), uterum med trädäck, trappa ut mot framsidan.
- **Källare** (`../bilder/SkissKällare.jpeg`): hall, sovrum/allrum, 2 sovrum, förråd, matkällare (Matk), dusch, bastu, trappa upp till markplan.
- **Spiraltrappa** förbinder entréplan och källare (samma position i båda planen).
- Foton per rum: `Kök`, `Matsal`, `Vardagsrum`, `VardagsrumOchMatsal`, `VardagsrumOchTrappa`, `TrappaOchMatsal`, `TrappaOchKällare`, `SovrumM`, `SovrumS`, `TrädgårdenIgen`, `Ovanifrån`, `Ovanifrån2` – används för material/färger och proportioner.
- **Mått:** planskisserna saknar mått. Exakta mått ska läggas in i husdatan (se nedan) när de finns – gissa aldrig tyst; markera uppskattade värden med `estimated: true`.

## Teknikstack

- **Angular** (senaste versionen), standalone components, **signals**, zoneless, `OnPush` överallt.
- **Three.js** för 3D-renderingen (TypeScript-typer via `@types/three`).
- **Vitest** för enhetstester.
- Utvecklas i **VS Code**. Node LTS, npm.
- Ingen backend i första versionen – allt körs i klienten. Scenarier sparas i `localStorage` och kan exporteras/importeras som JSON.
- Deploy: **GitHub Pages** via GitHub Actions (`ng build` med rätt `--base-href /norris/`).

## Arkitektur

Separera tydligt **data → 3D-generering → UI**.

```
src/app/
  model/        # Rena TS-typer + husdata (ingen Three.js här)
    house.types.ts       # Level, Wall, Opening (dörr/fönster), Room, Stair, Material
    house.data.ts        # Huset "som det är idag" i meter
    furniture.catalog.ts # Möbeltyper med mått och modell-referens
    scenario.types.ts    # Scenario = bas-hus + renoveringar + möblering
  engine/       # Three.js – bygger scen från modellen, ingen Angular-logik
    scene-builder.ts     # Model → Three.js-meshar (väggar extruderas, öppningar skärs ut)
    walk-controls.ts     # Förstapersonsläge: PointerLock + WASD, ögonhöjd ~1.65 m, kollision mot väggar
    orbit-view.ts        # Översiktsläge / planvy ovanifrån med "avtaget" tak
    picking.ts           # Raycast för att välja/flytta möbler
  state/        # Signal-baserade stores (aktivt scenario, valt objekt, vy-läge)
  ui/           # Angular-komponenter: verktygsfält, möbelkatalog, renoveringspanel, scenariolista
public/models/  # glTF/GLB-möbler (endast fria licenser, t.ex. CC0)
```

### Principer
- **Husdatan är källan till sanning.** 3D-geometrin genereras alltid från datan – modellera aldrig huset för hand i Three.js.
- Enheter: **meter**, högerhänt koordinatsystem, **Y uppåt**. Golv på entréplan = `y = 0`, källare negativ.
- **Renoveringar är diffar** mot bashuset (`removeWall`, `addWall`, `moveOpening`, `setMaterial` …), inte kopior. Då kan flera scenarier jämföras och bashuset uppdateras utan att scenarier går sönder.
- **Möblering** = lista av `{ catalogId, position, rotationY, levelId }` per scenario.
- Scenario-JSON ska ha `schemaVersion` och migreras vid formatändringar.
- Render-loopen körs utanför Angulars change detection; UI pratar med motorn via signals/tydligt service-API.
- Kassera Three.js-resurser (`geometry.dispose()`, `material.dispose()`, texturer) när scenen byggs om.
- Prestanda: sikta på 60 fps på en vanlig laptop. Instancing för upprepade objekt, låg polygonnivå, inga onödiga skuggor.

## Funktioner (prioritetsordning)

1. Entréplan i 3D med väggar, dörr- och fönsteröppningar, golv. Orbit-vy.
2. Förstapersonsläge (gå runt) med kollision.
3. Källare + spiraltrappa, byte mellan plan.
4. Möbelkatalog: placera, flytta, rotera, ta bort (enkla lådor först, glTF senare).
5. Renoveringar: ta bort/lägg till vägg, byt golv/väggfärg.
6. Spara/ladda/jämföra scenarier, export/import JSON.
7. Uterum, trädgård, mur och omgivning.
8. Mätverktyg (avstånd/yta).

## Konventioner

- Kod, identifierare och commit-meddelanden på **engelska**; UI-texter på **svenska** (`lang="sv"`).
- Strikt TypeScript (`strict: true`), inga `any`.
- Tester med Vitest för modell-/geometrilogik (t.ex. att väggöppningar hamnar rätt, kollision, scenario-diffar). Three.js-rendering testas inte pixelvis.
- Små, fokuserade commits. Kör `npm test` och `npm run build` innan push.
- Arbeta på feature-grenar och skapa pull requests mot `main` – pusha inte direkt till `main`.

## Kommandon

```bash
npm install
npm start          # ng serve – http://localhost:4200
npm test           # vitest
npm run build      # produktionsbygge
```

## Öppna frågor

- Exakta mått (rummens bredd/djup, takhöjd entréplan och källare, väggtjocklek, fönster- och dörrmått).
