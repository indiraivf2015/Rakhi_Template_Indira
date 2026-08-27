# Creative Studio template

A static, in-browser studio that drops a photo and name / designation / location onto official campaign artwork, then downloads a PNG.

This Rakshabandhan build is one instance of that studio. Copy the folder and swap artwork + numbers to reuse it for Diwali, Independence Day, Doctors’ Day, or any other card.

Photos and text stay in the browser. Nothing is uploaded to a server.

Repo: [indiraivf2015/Rakhi_Template_Indira](https://github.com/indiraivf2015/Rakhi_Template_Indira)

## What you get

- Solo mode: upload one photo, type name / designation / city, download PNG
- Optional Excel / ZIP batch (off by default)
- Doctor and Employee layouts for the same festival
- Photo fills the circular placeholder; the printed rakhi / ring stay on top
- Printed placeholders (`Your Name`, etc.) are wiped only when that field is filled

## Run it

Needs Python 3 on the PATH.

```bat
start.bat
```

Or:

```bat
cd C:\Rakshabandhan_Template
python -m http.server 8765
```

Open **http://127.0.0.1:8765/** and hard-refresh after you edit the HTML.

`index.html` redirects to `Indira_Creative_Studio.html`.

## Files to copy for a new campaign

| Path | Role |
|---|---|
| `Indira_Creative_Studio.html` | Whole app (UI + canvas) |
| `index.html` | Redirect into the studio |
| `templates/*.png` | Official artwork (native size, usually 768×1024) |
| `start.bat` | Local server |
| `vercel.json` | Optional host rewrite |
| `.gitignore` | Ignore junk |

Do **not** overwrite official PNGs with inpainted or screenshot copies. Point `BUILTIN` at new files instead.

## Reuse for another application

1. Copy this folder. Rename it, for example `Diwali_Template`.
2. Put the new official PNGs in `templates/`. Keep the originals lossless.
3. In `Indira_Creative_Studio.html`, change the title, header copy, and `VERSION`.
4. Point `BUILTIN` at the new files and add one `PRESETS` entry per layout.
5. Add matching `<option>` rows in `#layoutSelect`.
6. Measure the photo circle and nameplate on the **new** PNG (see below). Do not reuse Rakshabandhan `ellipse` / `textX` / `textCover` values.
7. Set `loc.prefix` if the printed line is `Indira IVF, {city}` (or change it).
8. Run locally, type a short name and a long name, download a PNG, and check both layouts.

### 1. Artwork paths

```js
const BUILTIN = {
  employee: 'templates/employee-YOUR-CAMPAIGN.png',
  doctor:   'templates/doctor-YOUR-CAMPAIGN.png'
};
```

Add or remove keys to match the dropdown.

### 2. Layout dropdown

```html
<select id="layoutSelect">
  <option value="doctor">Doctor · Your campaign</option>
  <option value="employee">Employee · Your campaign</option>
</select>
```

`value` must match a key in `BUILTIN` and `PRESETS`.

### 3. Geometry (`PRESETS`)

Each layout is one object. Coordinates are in **native template pixels** (768×1024 here).

```js
employee: {
  label: 'Employee · Your campaign',
  baseW: 768,
  aspect: 768 / 1024,
  ellipse: [x0, y0, x1, y1],   // photo hole, inner edge of the ring
  holeInset: 0,                // 0 = punch the full ellipse; do not use a large inset
  align: 'center',
  textX: 204,                  // center X of the floral / name (not always the photo center)
  maxW: 230,                   // max name width before the font shrinks
  name: { baseline: 738, size: 20, weight: 700, fill: '#A51C30' },
  desg: { baseline: 765, size: 15, weight: 400, fill: '#4A4A4A' },
  loc:  { baseline: 789, size: 15, weight: 400, fill: '#4A4A4A', prefix: 'Indira IVF, ' },
  paperY: [708, 714],          // clean cream rows ABOVE the name (no letters)
  paperMaxX: 278,              // stop before rakhi / ornaments on the right
  textCover: [
    { box: [x0, y0, x1, y1] }, // name
    { box: [x0, y0, x1, y1] }, // designation
    { box: [x0, y0, x1, y1] }  // location
  ]
}
```

How to measure:

- **ellipse** — inner fill of the grey / cream photo disk, including the part under the rakhi. If the photo stops short, grow `y1` (and the hole) until the disk is full. The punched template keeps the ring and rakhi on top.
- **textX** — horizontal center of the floral divider, not the photo, if they differ.
- **baselines** — Y of the official printed lines.
- **textCover** — boxes that fully cover `Your Name` / `Your Designation` / `Indira IVF, Your City`, including anti-alias. Keep the right edge off the rakhi.
- **paperY** — a few rows of plain paper *above* the name and *below* the floral. Never a row that still has printed text. Covering location from the designation row is what created the ghost line.

When name, designation, and city are all filled, the studio wipes the whole nameplate from `paperY`, then draws your three lines.

### 4. Branding and solo vs Excel

| Knob | Where | What to change |
|---|---|---|
| Page title / header | `<title>`, header HTML | Campaign name |
| Accent colours | `:root` `--accent`, `--cream` | Brand |
| Location prefix | `PRESETS.*.loc.prefix` | `Indira IVF, ` or empty |
| Excel / ZIP | `EXCEL_ENABLE` | `true` for team batch |
| Default layout | `geoKey` / `store.get('tpl', …)` | `'doctor'` or `'employee'` |

### 5. Excel batch (optional)

Set `EXCEL_ENABLE = true`. The sheet needs:

- **Name** (or Employee Name / Doctor Name)
- **Employee ID** and/or **Document Name** (photo file stem)

Optional: designation, location / city / centre.

Photos match when the file name equals Document Name or contains the Employee ID.

## How drawing works

1. Photo is cover-scaled and clipped to `ellipse`.
2. A hole-punched copy of the official PNG is drawn on top (ring + rakhi stay above the photo).
3. Filled fields trigger `coverPlaceholderLines` so printed placeholders disappear without a grainy box.
4. Name (bold maroon), designation, and `prefix + city` are drawn centered on `textX`.
5. Preview scales that native canvas down. Download PNG is native size.

Do not paint random cream, blit a scaled strip from another Y, or overwrite the official PNGs. Those create the grainy / ghost nameplate.

## Download PNG

The button enables when **photo, name, designation, and location** are all filled. You can download again after the first save.

Open the HTML in Chrome (not only an iframe preview) for one-click save. Preview frames can block downloads.

## Deploy

Static host (Vercel, Netlify, any file server). `vercel.json` already rewrites `/` to the studio.

After deploy, hard-refresh so browsers do not keep an old `Indira_Creative_Studio.html`.
