# RC Setup Helper

Mobile-first setup helper for the Tekno NT48 2.2 / ET48 2.2. It's a single-file app (`index.html`) that works offline. Setups and the log are kept in localStorage; Photo Setup pictures are kept in IndexedDB on the phone.

Live: https://aspinwelldrilling-commits.github.io/rc-setup-app/

* `tekno-data.json`: every setup-sheet field, its real options and the Tekno box-stock values with manual page sources, plus a `mip` section (MIP Bypass1 pistons/valves, from miponline.com only).
* `example-setup-import.json`: a ready-to-import example.

## Photo Setup

My Setup → **Photo Setup** walks through a shot list per truck. It stores each photo locally as a ≤1600 px JPEG and shares them as files named `nt48_<shot>.jpg` / `et48_<shot>.jpg` (Web Share API, or a .zip download with `<truck>_shotlist.txt`).
Send the photos to RC Racing in chat. It replies with a setup JSON file, which you import from **Photo Setup → Import setup** (file or paste) or from **My Setup → Import**.

Shot ids: `front_straight, rear_straight, front_tower, rear_tower, front_camber_link, rear_camber_link, front_spindle, front_arm_lower, rear_arm_lower, blocks_ab, blocks_cd, rear_hub, sway_front, sway_rear, shocks, steering, clutch_spur` (NT48) or `pinion_spur` (ET48), `tires`, `notes_card` (optional).

## Setup import JSON schema

```
{
  "app": "rc-setup-helper",        // optional
  "kind": "setups",                // optional
  "version": 2,                    // optional
  "setups": [ Setup, ... ]         // required (a bare array of Setup, or a single Setup object, is also accepted)
}

Setup = {
  "truck":  "NT48" | "ET48",       // REQUIRED ("nt48", "NT48 2.2" also accepted)
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
* Unknown field ids, and fields that don't apply to that truck (e.g. `clutchBell` on the ET48), are skipped with a warning.
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

| id | label | truck | type | allowed values | Tekno stock |
|---|---|---|---|---|---|
| `track` | Track | both | string | "Indoor", "Outdoor" | — |
| `size` | Size | both | string | "Small", "Medium", "Large" | — |
| `traction` | Traction | both | string | "Low", "Medium", "High" | — |
| `surface` | Surface | both | string | "Smooth", "Bumpy", "Rutted" | — |
| `type` | Type | both | string | "Loose / loamy", "Hard packed", "Blue groove", "Clay", "Astro" | — |
| `condition` | Condition | both | string | "Dusty", "Dry", "Wet", "Muddy" | — |
| `ackHole` | Ackermann plate (bellcrank) hole | both | string | "Hole 1 (upper on sheet)", "Hole 2 (lower on sheet)" | NT48: "Hole 2 (lower on sheet)"; ET48: "Hole 2 (lower on sheet)" |
| `ackWash` | Ackermann washers | both | number | any number | NT48: 4; ET48: 4 |
| `spPlate` | Spindle (steering arm) plate | both | string | "A", "B", "C", "E", "F", "G", "H" | NT48: "F"; ET48: "F" |
| `spWash` | Bump-steer washers (spindle ball stud) | both | number | any number | NT48: 4; ET48: 4 |
| `spHeight` | Spindle height (kingpin sleeves) | both | string | "L", "H" | NT48: "H"; ET48: "H" |
| `spPart` | Spindle part | both | string | "TKR9141", "TKR9141B" | NT48: "TKR9141B"; ET48: "TKR9141B" |
| `caster` | Caster block (spindle carrier) | both | string | "15", "18", "21" | NT48: "18"; ET48: "18" |
| `saver` | Servo saver (full turns from fully tight) | both | number | any number | NT48: 1; ET48: 1 |
| `fBar` | Front sway bar | both | string | "none", "1.6", "1.7", "1.8", "1.9", "2.0", "2.1", "2.2", "2.3", "2.4", "2.5", "2.6", "2.7", "2.8", "2.9", "3.0" | NT48: "2.3"; ET48: "2.3" |
| `rBar` | Rear sway bar | both | string | "none", "1.6", "1.7", "1.8", "1.9", "2.0", "2.1", "2.2", "2.3", "2.4", "2.5", "2.6", "2.7", "2.8", "2.9", "3.0" | NT48: "2.5"; ET48: "2.5" |
| `fBush` | Front bar dead-band bushing (TKR9290) | both | string | "2.4", "2.7", "3.0", "3.3" | NT48: "2.7"; ET48: "2.7" |
| `rBush` | Rear bar dead-band bushing (TKR9290) | both | string | "2.4", "2.7", "3.0", "3.3" | NT48: "3.0"; ET48: "3.0" |
| `fTower` | Front shock tower | both | string | free text | NT48: "TKR9428B"; ET48: "TKR9428B" |
| `fUpper` | Front upper shock hole (tower) | both | string | "Hole 1 (outermost)", "Hole 2", "Hole 3", "Hole 4 (innermost)" | NT48: "Hole 2"; ET48: "Hole 2" |
| `fStand` | Front shock standoff | both | string | "0mm", "+2mm", "+4mm" | NT48: "+2mm"; ET48: "+2mm" |
| `fLower` | Front lower shock hole (arm) | both | string | "Inside", "Middle", "Outside" | NT48: "Outside"; ET48: "Outside" |
| `fInCam` | Front inner camber link (tower, 1–6) | both | string | "1", "2", "3", "4", "5", "6" | NT48: "3"; ET48: "3" |
| `fOutCam` | Front outer camber link (spindle A/B/C) | both | string | "A", "B", "C" | NT48: "B"; ET48: "B" |
| `fCvd` | Front driveshaft | both | string | "U - TKR9487", "C - TKR9472" | NT48: "U - TKR9487"; ET48: "U - TKR9487" |
| `fHub` | Front wheel hub offset | both | string | "2mm", "3mm", "4mm" | NT48: "4mm"; ET48: "4mm" |
| `aBlock` | "A" block (front of front arms) | both | string | "TKR9461", "TKR9461B" | NT48: "TKR9461B"; ET48: "TKR9461B" |
| `aPill` | "A" block insert position | both | string | "Top-left", "Top-center", "Top-right", "Middle-left", "Center", "Middle-right", "Bottom-left", "Bottom-center", "Bottom-right" | NT48: "Center"; ET48: "Center" |
| `bBlock` | "B" block (rear of front arms) | both | string | "TKR9462", "TKR9462B" | NT48: "TKR9462B"; ET48: "TKR9462B" |
| `bPill` | "B" block insert position | both | string | "Top-left", "Top-center", "Top-right", "Middle-left", "Center", "Middle-right", "Bottom-left", "Bottom-center", "Bottom-right" | NT48: "Center"; ET48: "Center" |
| `insertType` | Hinge-pin insert system | both | string | "TKR5165 / TKR9055C inserts (kit)", "TKR31565 inserts (5mm pins TKR9620)" | NT48: "TKR5165 / TKR9055C inserts (kit)"; ET48: "TKR5165 / TKR9055C inserts (kit)" |
| `fRH` | Front ride height | both | number | any number (mm) | NT48: 34; ET48: 34 |
| `fCam` | Front camber | both | number | any number (°) | NT48: -2; ET48: -2 |
| `fCaster` | Caster (from block) | both | number | any number (°) | NT48: 18; ET48: 18 |
| `fSweep` | Arm sweep | both | number | any number (°) | NT48: 0; ET48: 0 |
| `fKick` | Kick-up | both | number | any number (°) | NT48: 10; ET48: 10 |
| `fToe` | Front toe (out, per side) | both | number | any number (° out) | NT48: 2; ET48: 2 |
| `fShockLen` | Front shock length | both | number | any number (mm) | NT48: 121; ET48: 121 |
| `fUpTravel` | Front up-travel limiters | both | number | any number (mm) | NT48: 0; ET48: 0 |
| `fDroop` | Front droop (block gauge) | both | number | any number (mm) | NT48: 0; ET48: 0 |
| `fOil` | Front shock oil | both | number | 250, 300, 350, 400, 450, 500, 550, 600, 650, 700, 750, 800, 850, 900, 950, 1000, 1050 | NT48: 550; ET48: 550 |
| `fPiston` | Front pistons | both | string | "TKR6136", "TKR6137", "TKR6130", "TKR6131", "TKR6160", "TKR6163" | NT48: "TKR6136"; ET48: "TKR6136" |
| `fPisType` | Front piston type | both | string | "tekno", "b1", "hf", "thf" | NT48: "tekno"; ET48: "tekno" |
| `fMipPiston` | Front MIP Bypass1 piston | both | string | "19021", "19011", "23401", "23402", "23403", "23404", "24401", "24402", "24403", "24404", "24406" | — |
| `fMipValve` | Front MIP rebound valve | both | string | "G", "TG", "B", "C", "N" | — |
| `fSpring` | Front springs | both | string | "TKR6090", "TKR6091", "TKR6092", "TKR6093" | NT48: "TKR6090"; ET48: "TKR6090" |
| `fRebound` | Front rebound | both | number | any number (%) | NT48: 0; ET48: 0 |
| `fBuild` | Front shock build | both | string | "Emulsion", "Bladder" | NT48: "Emulsion"; ET48: "Emulsion" |
| `rTower` | Rear shock tower | both | string | free text | NT48: "TKR9429B"; ET48: "TKR9429B" |
| `rUpper` | Rear upper shock hole (tower) | both | string | "Hole 1 (outermost)", "Hole 2", "Hole 3", "Hole 4 (innermost)" | NT48: "Hole 2"; ET48: "Hole 3" |
| `rStand` | Rear shock standoff | both | string | "0mm", "+2mm", "+4mm" | NT48: "0mm"; ET48: "0mm" |
| `rLower` | Rear lower shock hole (arm) | both | string | "Inside", "Middle", "Outside" | NT48: "Inside"; ET48: "Inside" |
| `rInCam` | Rear inner camber link (tower, 1–8) | both | string | "1", "2", "3", "4", "5", "6", "7", "8" | NT48: "2"; ET48: "2" |
| `rOutCam` | Rear outer camber link (hub hole 1–15) | both | string | "1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "12", "13", "14", "15" | NT48: "5"; ET48: "5" |
| `rCvd` | Rear driveshaft | both | string | "U - TKR9487", "U - TKR9687", "C - TKR9472" | NT48: "U - TKR9687"; ET48: "U - TKR9687" |
| `rHub` | Rear wheel hub offset | both | string | "2mm", "3mm", "4mm" | NT48: "2mm"; ET48: "2mm" |
| `rAxle` | Rear hub axle height | both | string | "-2mm", "0mm", "+2mm" | NT48: "-2mm"; ET48: "-2mm" |
| `cPill` | "C" block insert position | both | string | "Top-left", "Top-center", "Top-right", "Middle-left", "Center", "Middle-right", "Bottom-left", "Bottom-center", "Bottom-right" | NT48: "Center"; ET48: "Center" |
| `cScrew` | "C" block screw (TKR1324) | both | string | "In", "Out" | NT48: "In"; ET48: "In" |
| `dPill` | "D" block insert position | both | string | "Top-left", "Top-center", "Top-right", "Middle-left", "Center", "Middle-right", "Bottom-left", "Bottom-center", "Bottom-right" | NT48: "Center"; ET48: "Center" |
| `rRH` | Rear ride height | both | number | any number (mm) | NT48: 38; ET48: 38 |
| `rCam` | Rear camber | both | number | any number (°) | NT48: -2; ET48: -2 |
| `rAS` | Anti-squat | both | number | any number (°) | NT48: 3; ET48: 3 |
| `rToe` | Rear toe-in | both | number | any number (° in) | NT48: 2.5; ET48: 2.5 |
| `rShockLen` | Rear shock length | both | number | any number (mm) | NT48: 134; ET48: 134 |
| `rUpTravel` | Rear up-travel limiters | both | number | any number (mm) | NT48: 0; ET48: 0 |
| `rDroop` | Rear droop (block gauge) | both | number | any number (mm) | NT48: -8; ET48: -8 |
| `rOil` | Rear shock oil | both | number | 250, 300, 350, 400, 450, 500, 550, 600, 650, 700, 750, 800, 850, 900, 950, 1000, 1050 | NT48: 650; ET48: 650 |
| `rPiston` | Rear pistons | both | string | "TKR6136", "TKR6137", "TKR6130", "TKR6131", "TKR6160", "TKR6163" | NT48: "TKR6137"; ET48: "TKR6137" |
| `rPisType` | Rear piston type | both | string | "tekno", "b1", "hf", "thf" | NT48: "tekno"; ET48: "tekno" |
| `rMipPiston` | Rear MIP Bypass1 piston | both | string | "19021", "19011", "23401", "23402", "23403", "23404", "24401", "24402", "24403", "24404", "24406" | — |
| `rMipValve` | Rear MIP rebound valve | both | string | "G", "TG", "B", "C", "N" | — |
| `rSpring` | Rear springs | both | string | "TKR6080", "TKR6081", "TKR6082", "TKR6083", "TKR6090", "TKR6091", "TKR6092", "TKR6093" | NT48: "TKR6090"; ET48: "TKR6090" |
| `rRebound` | Rear rebound | both | number | any number (%) | NT48: 0; ET48: 0 |
| `rBuild` | Rear shock build | both | string | "Emulsion", "Bladder" | NT48: "Emulsion"; ET48: "Emulsion" |
| `fDiffOil` | Front diff oil | both | number | 1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000, 9000, 10000, 12500, 15000, 20000, 30000 | NT48: 10000; ET48: 10000 |
| `cDiffOil` | Center diff oil | both | number | 1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000, 9000, 10000, 12500, 15000, 20000, 30000 | NT48: 10000; ET48: 10000 |
| `rDiffOil` | Rear diff oil | both | number | 1000, 2000, 3000, 4000, 5000, 6000, 7000, 8000, 9000, 10000, 12500, 15000, 20000, 30000 | NT48: 5000; ET48: 5000 |
| `fOring` | Front diff o-rings | both | string | "TKR5144B", "TKR5144" | NT48: "TKR5144B"; ET48: "TKR5144B" |
| `fShim` | Front diff gear shims | both | string | "TKR5145B", "TKR9145" | NT48: "TKR5145B"; ET48: "TKR5145B" |
| `fOutdrive` | Front outdrives | both | string | "Standard (TKR9114/TKR9112)", "Lightened (TKR5114XB/TKR5112x)" | NT48: "Standard (TKR9114/TKR9112)"; ET48: "Standard (TKR9114/TKR9112)" |
| `fGears` | Front diff gears | both | string | "TKR9150", "TKR9150B" | NT48: "TKR9150B"; ET48: "TKR9150B" |
| `fCase` | Front diff case | both | string | "TKR9115C", "TKR9115B" | NT48: "TKR9115C"; ET48: "TKR9115B" |
| `fPin` | Front cross pins | both | string | "TKR9149", "TKR9149A", "TKR9149B" | NT48: "TKR9149"; ET48: "TKR9149" |
| `cOring` | Center diff o-rings | both | string | "TKR5144B", "TKR5144" | NT48: "TKR5144B"; ET48: "TKR5144B" |
| `cShim` | Center diff gear shims | both | string | "TKR5145B", "TKR9145" | NT48: "TKR5145B"; ET48: "TKR5145B" |
| `cOutdrive` | Center outdrives | both | string | "Standard (TKR9114/TKR9112)", "Lightened (TKR5114XB/TKR5112x)" | NT48: "Standard (TKR9114/TKR9112)"; ET48: "Standard (TKR9114/TKR9112)" |
| `cGears` | Center diff gears | both | string | "TKR9150", "TKR9150B" | NT48: "TKR9150B"; ET48: "TKR9150B" |
| `cCase` | Center diff case | both | string | "TKR9115C", "TKR9115B" | NT48: "TKR9115C"; ET48: "TKR9115B" |
| `cPin` | Center cross pins | both | string | "TKR9149", "TKR9149A", "TKR9149B" | NT48: "TKR9149"; ET48: "TKR9149" |
| `rOring` | Rear diff o-rings | both | string | "TKR5144B", "TKR5144" | NT48: "TKR5144B"; ET48: "TKR5144B" |
| `rShim` | Rear diff gear shims | both | string | "TKR5145B", "TKR9145" | NT48: "TKR5145B"; ET48: "TKR5145B" |
| `rOutdrive` | Rear outdrives | both | string | "Standard (TKR9114/TKR9112)", "Lightened (TKR5114XB/TKR5112x)" | NT48: "Standard (TKR9114/TKR9112)"; ET48: "Standard (TKR9114/TKR9112)" |
| `rGears` | Rear diff gears | both | string | "TKR9150", "TKR9150B" | NT48: "TKR9150B"; ET48: "TKR9150B" |
| `rCase` | Rear diff case | both | string | "TKR9115C", "TKR9115B" | NT48: "TKR9115C"; ET48: "TKR9115B" |
| `rPin` | Rear cross pins | both | string | "TKR9149", "TKR9149A", "TKR9149B" | NT48: "TKR9149"; ET48: "TKR9149" |
| `fBulk` | Front bulkhead | both | string | "TKR9416", "TKR9416B" | NT48: "TKR9416B"; ET48: "TKR9416B" |
| `rBulk` | Rear bulkhead | both | string | "TKR9416", "TKR9416B" | NT48: "TKR9416B"; ET48: "TKR9416B" |
| `braceMid` | Middle chassis brace | both | string | "Installed", "Removed" | NT48: "Installed"; ET48: "Installed" |
| `braceRear` | Rear chassis brace | both | string | "Installed", "Removed" | NT48: "Installed"; ET48: "Installed" |
| `wbRear` | Rear-arm wheelbase shims – rear (TKR5165) | both | number | any number (mm) | NT48: 3; ET48: 3 |
| `wbFront` | Rear-arm wheelbase shims – front (TKR5165) | both | number | any number (mm) | NT48: 2; ET48: 2 |
| `tBrand` | Tire brand / tread | both | string | free text | — |
| `tComp` | Compound | both | string | free text | — |
| `tInsert` | Insert | both | string | free text | — |
| `tWheel` | Wheel | both | string | free text | — |
| `body` | Body | both | string | free text | NT48: "Stock (TKR9445A)"; ET48: "Stock (TKR9645A)" |
| `wing` | Wing | both | string | free text | NT48: "Stock (TKR8292W)"; ET48: "Stock (TKR8292W)" |
| `wingPos` | Wing mount position | both | string | "0mm", "+7mm", "+12mm" | NT48: "+12mm"; ET48: "+12mm" |
| `clutchBell` | Clutch bell | NT48 | number | any number (t) | NT48: 13 |
| `spurNT` | Spur gear | NT48 | number | any number (t) | NT48: 48 |
| `shoes` | Clutch shoes | NT48 | string | free text | NT48: "Aluminum (TKR5351)" |
| `clutchSprings` | Clutch springs | NT48 | string | free text | NT48: "2× red, 2× gold (sheet)" |
| `brakeF` | Brake bias front | NT48 | number | any number (%) | NT48: 60 |
| `brakeR` | Brake bias rear | NT48 | number | any number (%) | NT48: 40 |
| `pinion` | Pinion | ET48 | string | "13", "14", "15", "16", "17", "18" | ET48: "15" |
| `spurET` | Spur gear (44t) | ET48 | string | "44t steel TKR9117", "44t composite TKR9237" | ET48: "44t composite TKR9237" |
| `engine` | Engine / pipe | NT48 | string | free text | — |
| `fuel` | Fuel / plug | NT48 | string | free text | — |
| `esc` | ESC | ET48 | string | free text | — |
| `motor` | Motor (kV) | ET48 | string | free text | — |
| `battery` | Battery | ET48 | string | free text | — |
| `sServo` | Steering servo | both | string | free text | NT48: "300 oz min"; ET48: "300 oz min" |
| `tServo` | Throttle / brake servo | NT48 | string | free text | NT48: "300 oz min" |
| `expo` | Steering expo | both | number | any number (%) | NT48: -10; ET48: -10 |
| `servoSpd` | Steering servo speed | both | number | any number | NT48: -10; ET48: -10 |
| `brakeDR` | Brake dual rate | both | number | any number (%) | NT48: 80; ET48: 80 |
| `notes` | Notes | both | string | free text | — |
