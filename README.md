# RC Setup Helper

Mobile-first setup helper for the Tekno NT48 2.2 / ET48 2.2 truggies and the NB48 2.2 / EB48 2.2 buggies. It's a single-file app (`index.html`) that works offline. Setups and the log are kept in localStorage; Photo Setup pictures are kept in IndexedDB on the phone (DB `rcsetup-photos`, store `shots`, key `<truck>_<shotId>`; the JPEG is stored as `blob`, or — on browsers that refuse Blobs in IndexedDB — as `buf` (ArrayBuffer) + `type`).

My Setup opens straight on the active setup sheet (created from Tekno Box Stock if you have none). Every section starts with its diagram and every row has a mini picture of its value; tap it for the big view. When a new version is deployed the service worker updates itself and the app shows **Update available · tap to refresh**.

Since v10 the My Setup sections, row thumbnails, overview cards and Compare are drawn on **Tekno RC’s own fillable setup sheets** (NT48 2.2, ET48 2.2, EB48 2.2 v3, and the newer NB48 2.2 v3 rev 260520 sheet): 600-dpi grayscale crops in `sheet/<vehicle>/` (lazy-loaded, pre-cached for offline once per version), with tap targets on the PDF form-field circles. Blue = your setting, dashed ring = Tekno box stock, orange = changed. Paper/Dark toggle (`rcsetup.sheetTheme`). Fields the sheet doesn't draw (MIP valving, radio, insert type…) keep the original SVG diagrams; the Learn/Advisor animations are unchanged. **Setup-sheet drawings © Tekno RC (teknorc.com), used for personal reference only — this repo is personal.**

Live: https://aspinwelldrilling-commits.github.io/rc-setup-app/

* `tekno-data.json`: every setup-sheet field, its real options and the Tekno box-stock values with manual page sources, plus a `mip` section (MIP Bypass1 pistons/valves, from miponline.com only).
* `example-setup-import.json`: a ready-to-import example.

## Photo Setup

**Photo Setup** (camera icon) can be opened three ways: the **Photos** tab in the bottom bar, the orange **Photo Setup** button at the bottom right of My Setup, or the **Photo Setup** button in the setup sheet's action row. It walks through a shot list per vehicle. It stores each photo locally as a ≤1600 px JPEG and shares them as files named `nt48_<shot>.jpg` / `et48_<shot>.jpg` / `nb48_<shot>.jpg` / `eb48_<shot>.jpg` (Web Share API, or a .zip download with `<truck>_shotlist.txt`).
Send the photos to RC Racing in chat. It replies with a setup JSON file, which you import from **Photo Setup → Import setup** (file or paste) or from **My Setup → Import**.

Shot ids: `front_straight, rear_straight, front_tower, rear_tower, front_camber_link, rear_camber_link, front_spindle, front_arm_lower, rear_arm_lower, blocks_ab, blocks_cd, rear_hub, sway_front, sway_rear, shocks, steering, clutch_spur` (NT48) or `pinion_spur` (ET48), `tires`, `notes_card` (optional).

## Vehicles and sources

| truck id | model | kit | Box Stock sheet |
|---|---|---|---|
| `NT48` | Tekno NT48 2.2 nitro 1/8 truggy | TKR9505 | manual p34 |
| `ET48` | Tekno ET48 2.2 electric 1/8 truggy | TKR9605 | manual p30 |
| `NB48` | Tekno NB48 2.2 nitro 1/8 buggy | TKR9305 | manual p35 |
| `EB48` | Tekno EB48 2.2 electric 1/8 buggy | TKR9005 | manual p30 |

Only official Tekno manuals / setup sheets / teknorc.com kit pages were used (no pro sheets). Values Tekno doesn't publish are left blank; see `gaps` and `discrepancies` in tekno-data.json.

## Setup import JSON schema

```
{
  "app": "rc-setup-helper",        // optional
  "kind": "setups",                // optional
  "version": 2,                    // optional
  "setups": [ Setup, ... ]         // required (a bare array of Setup, or a single Setup object, is also accepted)
}

Setup = {
  "truck":  "NT48" | "ET48" | "NB48" | "EB48",   // REQUIRED ("nb48", "NB48 2.2" also accepted)
  "vals":   { fieldId: value },    // REQUIRED, keys from the field table below
  "name":   string,                // default "Imported setup"
  "date":   "YYYY-MM-DD",          // default today
  "track":  string,                // default "Speed Lot (Ramona, CA)"
  "cond":   one of ["Dry / dusty", "Loose", "Medium bite", "High bite / blue groove", "Damp / tacky", "Wet / muddy"],
  "temp":   number (°F) | null,
  "base":   "stock" | "blank",     // "stock" = fill every field not in vals with the Tekno box-stock value (recommended when building from photos)
  "active": true | false,          // make it the active setup the Advisor reads (it is also made active if the truck has none)
  "notes":  string,                // optional, stored in the Notes field
  "source": string,                // optional, e.g. "photos"
  "id":     string                 // optional; an existing id REPLACES that setup, omit it to add a new one
}
```

Value rules (the importer checks each value and reports warnings):
* **select** fields: use the exact option value from the table (numbers stay numbers, e.g. `"fOil": 550`, `"fDiffOil": 7000`). Case-insensitive matches against the value or label are accepted, and so are numbers written as text (`"600 cst"` → 600). Anything else is kept as a *custom* value with a warning.
* **number** fields: a JSON number (strings like `"121 mm"` are parsed).
* **text** fields: any string.
* Unknown field ids, and fields that don't apply to that truck (e.g. `clutchBell` on the ET48/EB48, `cBlock` on the truggies, `braceNB` on anything but the NB48), are skipped with a warning.
* Leave out anything you can't read from the photos. With `"base": "stock"` those fields get the Tekno stock value. Without it they stay blank.
* `fPisType`/`rPisType` default to `"tekno"`, or to the type of the MIP piston given in `fMipPiston`/`rMipPiston`.

Exports from the app (`Export / share`) use the same format, with extra keys (`id`, `created`, `updated`, `parent`, `version`). They import unchanged.

### Example

```json
{
  "app": "rc-setup-helper",
  "kind": "setups",
  "version": 2,
  "setups": [
    {
      "truck": "NT48",
      "name": "Speed Lot – from photos",
      "date": "2026-10-17",
      "track": "Speed Lot (Ramona, CA)",
      "cond": "Dry / dusty",
      "temp": 78,
      "base": "stock",
      "active": true,
      "source": "photos",
      "vals": {
        "fUpper": "Hole 3",
        "fStand": "+2mm",
        "fInCam": "4",
        "fOutCam": "B",
        "fLower": "Outside",
        "rUpper": "Hole 2",
        "rInCam": "2",
        "rOutCam": "5",
        "rAxle": "-2mm",
        "aPill": "Center",
        "bPill": "Top-center",
        "fBar": "2.3",
        "rBar": "2.5",
        "fSpring": "TKR6091",
        "rSpring": "TKR6090",
        "fShockLen": 121,
        "rShockLen": 134,
        "fOil": 550,
        "rOil": 650,
        "fDiffOil": 7000,
        "cDiffOil": 7000,
        "rDiffOil": 5000,
        "rPisType": "b1",
        "rMipPiston": "19021",
        "rMipValve": "B",
        "clutchBell": 13,
        "spurNT": 48,
        "fCam": -2,
        "tBrand": "(tire name from sidewall)",
        "notes": "Built from photos; diff oils from notes card."
      }
    }
  ]
}
```

### Field reference (generated from tekno-data.json)

| id | label | vehicles | type | allowed values | Tekno stock |
|---|---|---|---|---|---|
| `name` | Setup name | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |
| `date` | Date | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |
| `event` | Event / track | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |
| `temp` | Ambient temp (°F) | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |
| `track` | Track | all | string | "Indoor", "Outdoor" | NT48: null; ET48: null; NB48: null; EB48: null |
| `size` | Size | all | string | "Small", "Medium", "Large" | NT48: null; ET48: null; NB48: null; EB48: null |
| `traction` | Traction | all | string | "Low", "Medium", "High" | NT48: null; ET48: null; NB48: null; EB48: null |
| `surface` | Surface | all | string | "Smooth", "Bumpy", "Rutted" | NT48: null; ET48: null; NB48: null; EB48: null |
| `type` | Type | all | string | "Loose / loamy", "Hard packed", "Blue groove", "Clay", "Astro" | NT48: null; ET48: null; NB48: null; EB48: null |
| `condition` | Condition | all | string | "Dusty", "Dry", "Wet", "Muddy" | NT48: null; ET48: null; NB48: null; EB48: null |
| `ackHole` | Ackermann plate (bellcrank) hole | all | string | "Hole 1 (upper on sheet)", "Hole 2 (lower on sheet)" | NT48: "Hole 1 (upper on sheet)"; ET48: "Hole 1 (upper on sheet)"; NB48: "Hole 1 (upper on sheet)"; EB48: "Hole 1 (upper on sheet)" |
| `ackWash` | Ackermann washers | all | string | free text | NT48: 4; ET48: 4; NB48: 4; EB48: 4 |
| `ackPN` | Ackermann plate part # | NB48 | string | free text | NB48: null |
| `spPlate` | Spindle (steering arm) plate | all | string | "A", "B", "C", "E", "F", "G", "H" | NT48: "F"; ET48: "F"; NB48: "F"; EB48: "F" |
| `spWash` | Bump-steer washers (spindle ball stud) | all | string | free text | NT48: 4; ET48: 4; NB48: 1; EB48: 1 |
| `spHeight` | Spindle height (kingpin sleeves) | all | string | "L", "H" | NT48: "H"; ET48: "H"; NB48: "H"; EB48: "H" |
| `spPart` | Spindle part | all | string | NT48, ET48, EB48: "TKR9141", "TKR9141B"; NB48: "TKR9141", "TKR9141B", "TKR9141C" | NT48: "TKR9141B"; ET48: "TKR9141B"; NB48: "TKR9141B"; EB48: "TKR9141B" |
| `caster` | Caster block (spindle carrier) (NB48: “Spindle carrier (caster block)”) | all | string | "15", "18", "21" | NT48: "18"; ET48: "18"; NB48: "21"; EB48: "21" |
| `saver` | Servo saver (full turns from fully tight) | all | string | free text | NT48: 1; ET48: 1; NB48: 2; EB48: 2 |
| `fBar` | Front sway bar | all | string | NT48, ET48: "none", "1.6", "1.7", "1.8", "1.9", "2.0", "2.1", "2.2", "2.3", "2.4", "2.5", "2.6", "2.7", "2.8", "2.9", "3.0"; NB48, EB48: "none", "2.0", "2.1", "2.2", "2.3", "2.4", "2.5", "2.6", "2.7", "2.8" | NT48: "2.3"; ET48: "2.3"; NB48: "2.3"; EB48: "2.3" |
| `rBar` | Rear sway bar | all | string | NT48, ET48: "none", "1.6", "1.7", "1.8", "1.9", "2.0", "2.1", "2.2", "2.3", "2.4", "2.5", "2.6", "2.7", "2.8", "2.9", "3.0"; NB48, EB48: "none", "2.2", "2.3", "2.4", "2.5", "2.6", "2.7", "2.8", "2.9", "3.0" | NT48: "2.5"; ET48: "2.5"; NB48: "2.5"; EB48: "2.5" |
| `fBush` | Front bar dead-band bushing (TKR9290) | all | string | "2.4", "2.7", "3.0", "3.3" | NT48: "2.7"; ET48: "2.7"; NB48: "2.7"; EB48: "2.7" |
| `rBush` | Rear bar dead-band bushing (TKR9290) | all | string | "2.4", "2.7", "3.0", "3.3" | NT48: "3.0"; ET48: "3.0"; NB48: "3.0"; EB48: "3.0" |
| `fTower` | Front shock tower | all | string | free text | NT48: "TKR9428B"; ET48: "TKR9428B"; NB48: "TKR9268E"; EB48: "TKR9268E" |
| `fUpper` | Front upper shock hole (tower) | all | string | NT48, ET48: "Hole 1 (outermost)", "Hole 2", "Hole 3", "Hole 4 (innermost)"; NB48, EB48: "Hole 1 (outermost)", "Hole 2", "Hole 3 (innermost)" | NT48: "Hole 2"; ET48: "Hole 2"; NB48: "Hole 2"; EB48: "Hole 2" |
| `fStand` | Front shock standoff | all | string | NT48, ET48, EB48: "0mm", "+2mm", "+4mm"; NB48: "0mm", "+2mm" | NT48: "+2mm"; ET48: "+2mm"; NB48: "+2mm"; EB48: "+2mm" |
| `fLower` | Front lower shock hole (arm) | all | string | NT48, ET48: "Inside", "Middle", "Outside"; NB48, EB48: "Inside", "Outside" | NT48: "Outside"; ET48: "Outside"; NB48: "Inside"; EB48: "Inside" |
| `fInCam` | Front inner camber link (tower, 1–6) (NB48/EB48: “Front inner camber link (tower, 1–3)”) | all | string | NT48, ET48: "1", "2", "3", "4", "5", "6"; NB48, EB48: "1", "2", "3" | NT48: "3"; ET48: "3"; NB48: "3"; EB48: "3" |
| `fOutCam` | Front outer camber link (spindle A/B/C) | all | string | "A", "B", "C" | NT48: "B"; ET48: "B"; NB48: "B"; EB48: "B" |
| `fCvd` | Front driveshaft | NT48, ET48, EB48 | string | NT48, ET48: "U - TKR9487", "C - TKR9472"; EB48: "U - TKR9087", "C - TKR8072" | NT48: "U - TKR9487"; ET48: "U - TKR9487"; EB48: "U - TKR9087" |
| `fHub` | Front wheel hub offset | all | string | NT48, ET48: "2mm", "3mm", "4mm"; NB48, EB48: "0mm", "1mm", "2mm" | NT48: "4mm"; ET48: "4mm"; NB48: "1mm"; EB48: "1mm" |
| `aBlock` | "A" block (front of front arms) | all | string | NT48, ET48: "TKR9461", "TKR9461B"; NB48, EB48: "TKR9161", "TKR9161B" | NT48: "TKR9461B"; ET48: "TKR9461B"; NB48: "TKR9161B"; EB48: "TKR9161B" |
| `aPill` | "A" block insert position | all | string | "Top-left", "Top-center", "Top-right", "Middle-left", "Center", "Middle-right", "Bottom-left", "Bottom-center", "Bottom-right" | NT48: "Center"; ET48: "Center"; NB48: "Center"; EB48: "Center" |
| `bBlock` | "B" block (rear of front arms) | all | string | NT48, ET48: "TKR9462", "TKR9462B"; NB48, EB48: "TKR9162", "TKR9162B" | NT48: "TKR9462B"; ET48: "TKR9462B"; NB48: "TKR9162B"; EB48: "TKR9162B" |
| `bPill` | "B" block insert position | all | string | "Top-left", "Top-center", "Top-right", "Middle-left", "Center", "Middle-right", "Bottom-left", "Bottom-center", "Bottom-right" | NT48: "Center"; ET48: "Center"; NB48: "Center"; EB48: "Center" |
| `insertType` | Hinge-pin insert system | all | string | "TKR5165 / TKR9055C inserts (kit)", "TKR31565 inserts (5mm pins TKR9620)" | NT48: "TKR5165 / TKR9055C inserts (kit)"; ET48: "TKR5165 / TKR9055C inserts (kit)"; NB48: "TKR5165 / TKR9055C inserts (kit)"; EB48: "TKR5165 / TKR9055C inserts (kit)" |
| `fRH` | Front ride height | all | string | free text | NT48: 34; ET48: 34; NB48: 24; EB48: 23 |
| `fCam` | Front camber | all | string | free text | NT48: -2; ET48: -2; NB48: -2; EB48: -2 |
| `fCaster` | Caster (from block) (NB48: “Spindle carrier (caster)”) | all | string | free text | NT48: 18; ET48: 18; NB48: 21; EB48: 21 |
| `fSweep` | Arm sweep | all | string | free text | NT48: 0; ET48: 0; NB48: 0; EB48: 0 |
| `fKick` | Kick-up | all | string | free text | NT48: 10; ET48: 10; NB48: 10; EB48: 10 |
| `fToe` | Front toe (out, per side) | all | string | free text | NT48: 2; ET48: 2; NB48: 2; EB48: 2 |
| `fShockLen` | Front shock length (NB48: “Front body / shaft (shock length)”) | all | string | free text | NT48: 121; ET48: 121; NB48: 115.5; EB48: 115.5 |
| `fUpTravel` | Front up-travel limiters | all | string | free text | NT48: 0; ET48: 0; NB48: 0; EB48: 0 |
| `fDroop` | Front droop (block gauge) | all | string | free text | NT48: 0; ET48: 0; NB48: 4; EB48: 4 |
| `fOil` | Front shock oil | all | number | 250, 300, 350, 400, 450, 500, 550, 600, 650, 700, 750, 800, 850, 900, 950, 1000, 1050 | NT48: 550; ET48: 550; NB48: 500; EB48: 500 |
| `fPiston` | Front pistons | all | string | NT48, ET48: "TKR6136", "TKR6137", "TKR6130", "TKR6131", "TKR6160", "TKR6163"; NB48, EB48: "TKR6138", "TKR6139", "TKR6130", "TKR6131", "TKR6160", "TKR6163" | NT48: "TKR6136"; ET48: "TKR6136"; NB48: "TKR6138"; EB48: "TKR6138" |
| `fPisType` | Front piston type | all | string | "tekno", "b1", "hf", "thf" | NT48: "tekno"; ET48: "tekno"; NB48: "tekno"; EB48: "tekno" |
| `fMipPiston` | Front MIP Bypass1 piston | all | string | "19021", "19011", "23401", "23402", "23403", "23404", "24401", "24402", "24403", "24404", "24406" | NT48: null; ET48: null; NB48: null; EB48: null |
| `fMipValve` | Front MIP rebound valve | all | string | "G", "TG", "B", "C", "N" | NT48: null; ET48: null; NB48: null; EB48: null |
| `fSpring` | Front springs | all | string | NT48, ET48: "TKR6090", "TKR6091", "TKR6092", "TKR6093"; NB48, EB48: "TKR6030B", "TKR6030C", "TKR6030D", "TKR6031B", "TKR6031C", "TKR6031D", "TKR6032B", "TKR6032C", "TKR6032D", "TKR6033B", "TKR6033C", "TKR6033D", "TKR6034B", "TKR6034C", "TKR6034D", "TKR6035", "TKR6036", "TKR6037", "TKR6038", "TKR6039", "TKR6040", "TKR6045" | NT48: "TKR6090"; ET48: "TKR6090"; NB48: "TKR6032B"; EB48: "TKR6032B" |
| `fRebound` | Front rebound | all | string | free text | NT48: 0; ET48: 0; NB48: 0; EB48: 0 |
| `fBuild` | Front shock build | all | string | "Emulsion", "Bladder" | NT48: "Emulsion"; ET48: "Emulsion"; NB48: "Emulsion"; EB48: "Emulsion" |
| `fBoot` | Front shock boots | NB48 | string | free text | NB48: null |
| `fRodEnd` | Front shock rod ends | NB48 | string | free text | NB48: null |
| `rTower` | Rear shock tower | all | string | free text | NT48: "TKR9429B"; ET48: "TKR9429B"; NB48: "TKR9269E"; EB48: "TKR9269E" |
| `rUpper` | Rear upper shock hole (tower) | all | string | NT48, ET48: "Hole 1 (outermost)", "Hole 2", "Hole 3", "Hole 4 (innermost)"; NB48, EB48: "Hole 1 (outermost)", "Hole 2", "Hole 3", "Hole 4", "Hole 5 (innermost)" | NT48: "Hole 2"; ET48: "Hole 3"; NB48: "Hole 3"; EB48: "Hole 3" |
| `rStand` | Rear shock standoff | all | string | NT48, ET48, EB48: "0mm", "+2mm", "+4mm"; NB48: "0mm", "+2mm" | NT48: "0mm"; ET48: "0mm"; NB48: "0mm"; EB48: "0mm" |
| `rLower` | Rear lower shock hole (arm) | all | string | NT48, ET48: "Inside", "Middle", "Outside"; NB48, EB48: "Inside", "Outside" | NT48: "Inside"; ET48: "Inside"; NB48: "Inside"; EB48: "Inside" |
| `rInCam` | Rear inner camber link (tower, 1–8) (NB48/EB48: “Rear inner camber link (tower, 1–6)”) | all | string | NT48, ET48: "1", "2", "3", "4", "5", "6", "7", "8"; NB48, EB48: "1", "2", "3", "4", "5", "6" | NT48: "2"; ET48: "2"; NB48: "2"; EB48: "2" |
| `rOutCam` | Rear outer camber link (hub hole 1–15) | all | string | "1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15" | NT48: "5"; ET48: "5"; NB48: "5"; EB48: "5" |
| `rCvd` | Rear driveshaft (NB48: “Rear driveshaft (UNI)”) | all | string | NT48, ET48: "U - TKR9487", "U - TKR9687", "C - TKR9472"; NB48: "U - TKR9087", "U - TKR9287", "U - TKR9287B"; EB48: "U - TKR9087", "U - TKR9287", "U - TKR9287B", "C - TKR8072" | NT48: "U - TKR9687"; ET48: "U - TKR9687"; NB48: "U - TKR9287"; EB48: "U - TKR9287" |
| `rHub` | Rear wheel hub offset | all | string | NT48, ET48: "2mm", "3mm", "4mm"; NB48, EB48: "0mm", "1mm", "2mm" | NT48: "2mm"; ET48: "2mm"; NB48: "1mm"; EB48: "1mm" |
| `rAxle` | Rear hub axle height | all | string | "-2mm", "0mm", "+2mm" | NT48: "-2mm"; ET48: "-2mm"; NB48: "-2mm"; EB48: "-2mm" |
| `cPill` | "C" block insert position | all | string | "Top-left", "Top-center", "Top-right", "Middle-left", "Center", "Middle-right", "Bottom-left", "Bottom-center", "Bottom-right" | NT48: "Center"; ET48: "Center"; NB48: "Center"; EB48: "Center" |
| `cBlock` | “C” block (rear arm, front) | NB48, EB48 | string | NB48: "TKR9163C", "TKR9163B", "TKR9163BW", "TKR9163CW"; EB48: "TKR9163B", "TKR9163C" | NB48: "TKR9163B"; EB48: "TKR9163B" |
| `cScrew` | "C" block screw (TKR1324) | all | string | "In", "Out" | NT48: "In"; ET48: "In"; NB48: "In"; EB48: "In" |
| `dPill` | "D" block insert position | all | string | "Top-left", "Top-center", "Top-right", "Middle-left", "Center", "Middle-right", "Bottom-left", "Bottom-center", "Bottom-right" | NT48: "Center"; ET48: "Center"; NB48: "Center"; EB48: "Center" |
| `dBlock` | “D” block (rear arm, rear) | NB48, EB48 | string | NB48: "TKR9164B", "TKR9164C", "TKR9164CW"; EB48: "TKR9164B", "TKR9164C" | NB48: "TKR9164C"; EB48: "TKR9164C" |
| `rRH` | Rear ride height | all | string | free text | NT48: 38; ET48: 38; NB48: 26; EB48: 25 |
| `rCam` | Rear camber | all | string | free text | NT48: -2; ET48: -2; NB48: -2; EB48: -2 |
| `rAS` | Anti-squat | all | string | free text | NT48: 3; ET48: 3; NB48: 2; EB48: 2 |
| `rToe` | Rear toe-in | all | string | free text | NT48: 2.5; ET48: 2.5; NB48: 2.5; EB48: 2.5 |
| `rShockLen` | Rear shock length (NB48: “Rear body / shaft (shock length)”) | all | string | free text | NT48: 134; ET48: 134; NB48: 125; EB48: 125 |
| `rUpTravel` | Rear up-travel limiters | all | string | free text | NT48: 0; ET48: 0; NB48: 0; EB48: 0 |
| `rDroop` | Rear droop (block gauge) | all | string | free text | NT48: -8; ET48: -8; NB48: -4; EB48: -4 |
| `rOil` | Rear shock oil | all | number | 250, 300, 350, 400, 450, 500, 550, 600, 650, 700, 750, 800, 850, 900, 950, 1000, 1050 | NT48: 650; ET48: 650; NB48: 600; EB48: 600 |
| `rPiston` | Rear pistons | all | string | NT48, ET48: "TKR6136", "TKR6137", "TKR6130", "TKR6131", "TKR6160", "TKR6163"; NB48, EB48: "TKR6138", "TKR6139", "TKR6130", "TKR6131", "TKR6160", "TKR6163" | NT48: "TKR6137"; ET48: "TKR6137"; NB48: "TKR6139"; EB48: "TKR6139" |
| `rPisType` | Rear piston type | all | string | "tekno", "b1", "hf", "thf" | NT48: "tekno"; ET48: "tekno"; NB48: "tekno"; EB48: "tekno" |
| `rMipPiston` | Rear MIP Bypass1 piston | all | string | "19021", "19011", "23401", "23402", "23403", "23404", "24401", "24402", "24403", "24404", "24406" | NT48: null; ET48: null; NB48: null; EB48: null |
| `rMipValve` | Rear MIP rebound valve | all | string | "G", "TG", "B", "C", "N" | NT48: null; ET48: null; NB48: null; EB48: null |
| `rSpring` | Rear springs | all | string | NT48, ET48: "TKR6080", "TKR6081", "TKR6082", "TKR6083", "TKR6090", "TKR6091", "TKR6092", "TKR6093"; NB48, EB48: "TKR6030B", "TKR6030C", "TKR6030D", "TKR6031B", "TKR6031C", "TKR6031D", "TKR6032B", "TKR6032C", "TKR6032D", "TKR6033B", "TKR6033C", "TKR6033D", "TKR6034B", "TKR6034C", "TKR6034D" | NT48: "TKR6090"; ET48: "TKR6090"; NB48: "TKR6032B"; EB48: "TKR6032B" |
| `rRebound` | Rear rebound | all | string | free text | NT48: 0; ET48: 0; NB48: 0; EB48: 0 |
| `rBuild` | Rear shock build | all | string | "Emulsion", "Bladder" | NT48: "Emulsion"; ET48: "Emulsion"; NB48: "Emulsion"; EB48: "Emulsion" |
| `rBoot` | Rear shock boots | NB48 | string | free text | NB48: null |
| `rRodEnd` | Rear shock rod ends | NB48 | string | free text | NB48: null |
| `fDiffOil` | Front diff oil | all | number | 1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000, 9000, 10000, 12500, 15000, 20000, 30000 | NT48: 10000; ET48: 10000; NB48: 7000; EB48: 7000 |
| `cDiffOil` | Center diff oil | all | number | 1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000, 9000, 10000, 12500, 15000, 20000, 30000 | NT48: 10000; ET48: 10000; NB48: 5000; EB48: 5000 |
| `rDiffOil` | Rear diff oil | all | number | 1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000, 9000, 10000, 12500, 15000, 20000, 30000 | NT48: 5000; ET48: 5000; NB48: 5000; EB48: 5000 |
| `fOring` | Front diff o-rings | all | string | "TKR5144B", "TKR5144" | NT48: "TKR5144B"; ET48: "TKR5144B"; NB48: "TKR5144B"; EB48: "TKR5144B" |
| `fShim` | Front diff gear shims | all | string | "TKR5145B", "TKR9145" | NT48: "TKR5145B"; ET48: "TKR5145B"; NB48: "TKR5145B"; EB48: "TKR5145B" |
| `fOutdrive` | Front outdrives | all | string | "Standard (TKR9114/TKR9112)", "Lightened (TKR5114XB/TKR5112x)" | NT48: "Standard (TKR9114/TKR9112)"; ET48: "Standard (TKR9114/TKR9112)"; NB48: "Standard (TKR9114/TKR9112)"; EB48: "Standard (TKR9114/TKR9112)" |
| `fGears` | Front diff gears | all | string | NT48, ET48, EB48: "TKR9150", "TKR9150B"; NB48: "TKR9150", "TKR9150B", "TKR9150X" | NT48: "TKR9150B"; ET48: "TKR9150B"; NB48: "TKR9150B"; EB48: "TKR9150B" |
| `fCase` | Front diff case | all | string | "TKR9115C", "TKR9115B" | NT48: "TKR9115C"; ET48: "TKR9115B"; NB48: "TKR9115C"; EB48: "TKR9115C" |
| `fPin` | Front cross pins | all | string | "TKR9149", "TKR9149A", "TKR9149B" | NT48: "TKR9149"; ET48: "TKR9149"; NB48: "TKR9149"; EB48: "TKR9149" |
| `cOring` | Center diff o-rings | all | string | "TKR5144B", "TKR5144" | NT48: "TKR5144B"; ET48: "TKR5144B"; NB48: "TKR5144B"; EB48: "TKR5144B" |
| `cShim` | Center diff gear shims | all | string | "TKR5145B", "TKR9145" | NT48: "TKR5145B"; ET48: "TKR5145B"; NB48: "TKR5145B"; EB48: "TKR5145B" |
| `cOutdrive` | Center outdrives | all | string | "Standard (TKR9114/TKR9112)", "Lightened (TKR5114XB/TKR5112x)" | NT48: "Standard (TKR9114/TKR9112)"; ET48: "Standard (TKR9114/TKR9112)"; NB48: "Standard (TKR9114/TKR9112)"; EB48: "Standard (TKR9114/TKR9112)" |
| `cGears` | Center diff gears | all | string | NT48, ET48, EB48: "TKR9150", "TKR9150B"; NB48: "TKR9150", "TKR9150B", "TKR9150X" | NT48: "TKR9150B"; ET48: "TKR9150B"; NB48: "TKR9150B"; EB48: "TKR9150B" |
| `cCase` | Center diff case | all | string | "TKR9115C", "TKR9115B" | NT48: "TKR9115C"; ET48: "TKR9115B"; NB48: "TKR9115C"; EB48: "TKR9115C" |
| `cPin` | Center cross pins | all | string | "TKR9149", "TKR9149A", "TKR9149B" | NT48: "TKR9149"; ET48: "TKR9149"; NB48: "TKR9149"; EB48: "TKR9149" |
| `rOring` | Rear diff o-rings | all | string | "TKR5144B", "TKR5144" | NT48: "TKR5144B"; ET48: "TKR5144B"; NB48: "TKR5144B"; EB48: "TKR5144B" |
| `rShim` | Rear diff gear shims | all | string | "TKR5145B", "TKR9145" | NT48: "TKR5145B"; ET48: "TKR5145B"; NB48: "TKR5145B"; EB48: "TKR5145B" |
| `rOutdrive` | Rear outdrives | all | string | "Standard (TKR9114/TKR9112)", "Lightened (TKR5114XB/TKR5112x)" | NT48: "Standard (TKR9114/TKR9112)"; ET48: "Standard (TKR9114/TKR9112)"; NB48: "Standard (TKR9114/TKR9112)"; EB48: "Standard (TKR9114/TKR9112)" |
| `rGears` | Rear diff gears | all | string | NT48, ET48, EB48: "TKR9150", "TKR9150B"; NB48: "TKR9150", "TKR9150B", "TKR9150X" | NT48: "TKR9150B"; ET48: "TKR9150B"; NB48: "TKR9150B"; EB48: "TKR9150B" |
| `rCase` | Rear diff case | all | string | "TKR9115C", "TKR9115B" | NT48: "TKR9115C"; ET48: "TKR9115B"; NB48: "TKR9115C"; EB48: "TKR9115C" |
| `rPin` | Rear cross pins | all | string | "TKR9149", "TKR9149A", "TKR9149B" | NT48: "TKR9149"; ET48: "TKR9149"; NB48: "TKR9149"; EB48: "TKR9149" |
| `fBulk` | Front bulkhead | all | string | NT48, ET48, EB48: "TKR9416", "TKR9416B"; NB48: "TKR9012", "TKR9412", "TKR9416B" | NT48: "TKR9416B"; ET48: "TKR9416B"; NB48: "TKR9416B"; EB48: "TKR9416B" |
| `rBulk` | Rear bulkhead | all | string | NT48, ET48, EB48: "TKR9416", "TKR9416B"; NB48: "TKR9012", "TKR9412", "TKR9416B" | NT48: "TKR9416B"; ET48: "TKR9416B"; NB48: "TKR9416B"; EB48: "TKR9416B" |
| `braceMid` | Middle chassis brace | NT48, ET48, EB48 | string | "Installed", "Removed" | NT48: "Installed"; ET48: "Installed"; EB48: "Installed" |
| `braceRear` | Rear chassis brace | NT48, ET48, EB48 | string | "Installed", "Removed" | NT48: "Installed"; ET48: "Installed"; EB48: "Installed" |
| `braceNB` | Rear chassis brace | NB48 | string | "Rear-short", "Rear-long" | NB48: "Rear-long" |
| `wbRear` | Rear-arm wheelbase shims – rear (TKR5165) | all | string | free text | NT48: 3; ET48: 3; NB48: 1; EB48: 1 |
| `wbFront` | Rear-arm wheelbase shims – front (TKR5165) | all | string | free text | NT48: 2; ET48: 2; NB48: 2; EB48: 2 |
| `tBrand` | Tire brand / tread | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |
| `tComp` | Compound | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |
| `tInsert` | Insert | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |
| `tWheel` | Wheel | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |
| `body` | Body | all | string | free text | NT48: "Stock (TKR9445A)"; ET48: "Stock (TKR9645A)"; NB48: "Stock (TKR9345B)"; EB48: "Stock (TKR9045)" |
| `wing` | Wing | all | string | free text | NT48: "Stock (TKR8292W)"; ET48: "Stock (TKR8292W)"; NB48: "Stock (TKR8292W)"; EB48: "Stock (TKR8292W)" |
| `wingPos` | Wing mount position | all | string | NT48, ET48: "0mm", "+7mm", "+12mm"; NB48, EB48: "0mm", "+7mm" | NT48: "+12mm"; ET48: "+12mm"; NB48: "+7mm"; EB48: "+7mm" |
| `clutchBell` | Clutch bell | NT48, NB48 | string | free text | NT48: 13; NB48: 13 |
| `spurNT` | Spur gear | NT48, NB48 | string | free text | NT48: 48; NB48: 48 |
| `shoes` | Clutch shoes | NT48, NB48 | string | free text | NT48: "Aluminum (TKR5351)"; NB48: "Aluminum (TKR5351)" |
| `clutchSprings` | Clutch springs | NT48, NB48 | string | free text | NT48: "2× red, 2× gold (sheet)"; NB48: "2× green, 2× gold (sheet)" |
| `brakeF` | Brake bias front | NT48, NB48 | string | free text | NT48: 60; NB48: 60 |
| `brakeR` | Brake bias rear | NT48, NB48 | string | free text | NT48: 40; NB48: 40 |
| `pinion` | Pinion | ET48, EB48 | string | "13", "14", "15", "16", "17", "18" | ET48: "15"; EB48: "16" |
| `spurET` | Spur gear (44t) | ET48, EB48 | string | "44t steel TKR9117", "44t composite TKR9237" | ET48: "44t composite TKR9237"; EB48: "44t composite TKR9237" |
| `engine` | Engine / pipe (NB48: “Engine”) | NT48, NB48 | string | free text | NT48: null; NB48: null |
| `pipe` | Pipe | NB48 | string | free text | NB48: null |
| `fuel` | Fuel / plug (NB48: “Fuel”) | NT48, NB48 | string | free text | NT48: null; NB48: null |
| `plug` | Glow plug | NB48 | string | free text | NB48: null |
| `esc` | ESC | ET48, EB48 | string | free text | ET48: null; EB48: null |
| `motor` | Motor (kV) | ET48, EB48 | string | free text | ET48: null; EB48: null |
| `battery` | Battery | ET48, EB48 | string | free text | ET48: null; EB48: null |
| `sServo` | Steering servo | all | string | free text | NT48: "300 oz min"; ET48: "300 oz min"; NB48: "300 oz min"; EB48: "300 oz min" |
| `tServo` | Throttle / brake servo (NB48: “Throttle / brake servo”) | NT48, NB48 | string | free text | NT48: "300 oz min"; NB48: "300 oz min" |
| `expo` | Steering expo | all | string | free text | NT48: -10; ET48: -10; NB48: -10; EB48: -10 |
| `servoSpd` | Steering servo speed | all | string | free text | NT48: -10; ET48: -10; NB48: -10; EB48: -10 |
| `brakeDR` | Brake dual rate | all | string | free text | NT48: 80; ET48: 80; NB48: 80; EB48: 80 |
| `notes` | Notes | all | string | free text | NT48: null; ET48: null; NB48: null; EB48: null |

## v11 – livery & icon
- Vehicle art (advisor scenes, before/after, Tires & body, empty states) now uses Travis's own truck livery: silver body, orange/yellow flames, black lower sides, black wing. NT48 has the nitro head/antenna, two windows and gold springs; ET48 has a closed body, one window and silver springs. Buggies use the same livery family on a darker body.
- The app icon and header logo are original vector art rendered from the in-app NT48 art. Tekno/TZR marks are suggested with text-free shapes only.
- My Setup has an optional "My truck photo" per vehicle. It is stored only on the device (IndexedDB) and is not included in Photo Setup counts, zip or share.

## v12 – photo-based vehicle art
- **Side views (all advisor scenes, before/after, empty states):** cut-outs of Travis's own trucks. The NT48 is from the May 3 2025 photo, with the 1st-place tag removed; there's an option in My Setup to show it again. The ET48 is from the Feb 15 photo, mirrored with its decals turned back to read correctly. Body and wheels are separate layers, so the existing transform animations still move the chassis on its suspension. Each layer has a soft contact shadow.
- **Buggies (NB48/EB48):** photo composites built from the same cut-outs, with buggy wheelbase and wheel size and a darker body.
- **Top / rear views:** photo-textured shaded renders. These are the previous art with photo-style pin-tread tyres, grain and softening.
- **Icon, header logo and splash:** the NT48 photo cut-out on carbon fibre with an orange accent.
- **Dirt ground:** CC0 texture "Ground103" from ambientCG (ambientcg.com).
- All sprites are in `img/` (WebP) and are cached by the service worker for offline use. The vector art is still the fallback if an image can't load.
