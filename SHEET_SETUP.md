# Meeting Inventory Tool — Setup Guide

This guide walks you through connecting the tool to Google Sheets so quantities stay in sync.

---

## Step 1 — Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet.
2. Name it something like **Meeting Inventory**.
3. You need **two sheets (tabs)** inside the file:

### Tab 1: `Literature`

Name this tab exactly: **Literature**

| Column A (name) | Column B (qty) |
|---|---|
| Basic Text | 5 |
| Just for Today | 5 |
| Living Clean | 3 |
| It Works: How and Why | 3 |
| Step Working Guides | 3 |
| Guiding Principles (Traditions) | 2 |
| In Times of Illness | 2 |
| For the Newcomer | 10 |
| White Booklet | 10 |
| NA Way of Life | 2 |
| PR Booklet | 2 |
| Today a New Beginning | 2 |

> **Column A** = item name, **Column B** = agreed quantity. No header row.

### Tab 2: `Chips`

Name this tab exactly: **Chips**

| Column A (name) | Column B (qty) | Column C (description) | Column D (category) |
|---|---|---|---|
| Welcome Keytag | 10 | white — surrender | Keytags |
| 24 Hour Keytag | 10 | orange | Keytags |
| 30 Day Keytag | 5 | glow — 1 month | Keytags |
| 60 Day Keytag | 5 | bronze — 2 months | Keytags |
| 90 Day Keytag | 5 | red — 3 months | Keytags |
| 6 Month Keytag | 3 | dark blue — 6 months | Keytags |
| 9 Month Keytag | 3 | green — 9 months | Keytags |
| I Year Coin | 3 | 1 year | Year Coins |
| II Year Coin | 2 | 2 years | Year Coins |
| III Year Coin | 2 | 3 years | Year Coins |
| ... | ... | ... | ... |
| L Year Coin | 1 | 50 years | Year Coins |

> Columns C and D are optional but recommended for the chips tab — they add the description and grouping.

---

## Step 2 — Publish the Sheet for Reading

For the tool to **read** quantities automatically:

1. In your spreadsheet, click **File → Share → Publish to web**
2. Under "Link", choose **Entire Document** and **Web page**
3. Click **Publish** and confirm
4. Close the dialog

> The sheet still needs to be **shared** too: Click **Share** in the top right → change to **Anyone with the link can view**.

---

## Step 3 — Get Your Sheet ID

From your sheet's URL:

```
https://docs.google.com/spreadsheets/d/[THIS_IS_YOUR_SHEET_ID]/edit
```

Copy everything between `/d/` and `/edit`. Paste it into the **Sheet ID** field in the tool's Settings tab.

---

## Step 4 — Set Up the Apps Script (for Saving Back to Sheet)

This step lets the tool **write** your updated quantities back to the sheet when you click Save.

1. In your spreadsheet, click **Extensions → Apps Script**
2. Delete the default code and paste this:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheetName = data.sheet;
    const items = data.data;

    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(sheetName);
    if (!sheet) throw new Error('Sheet not found: ' + sheetName);

    sheet.clearContents();

    const rows = items.map(item => {
      const row = [item.name, item.qty];
      if (item.sub) row.push(item.sub);
      if (item.category) row.push(item.category);
      return row;
    });

    sheet.getRange(1, 1, rows.length, rows[0].length).setValues(rows);

    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'error', message: err.message }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  return ContentService.createTextOutput('Meeting Inventory Script is running.');
}
```

3. Click **Save** (name the project anything, like "Meeting Inventory")
4. Click **Deploy → New deployment**
5. Click the gear icon next to "Select type" and choose **Web app**
6. Set:
   - **Execute as**: Me
   - **Who has access**: Anyone
7. Click **Deploy**
8. Authorize the permissions when prompted
9. Copy the **Web App URL** that appears

Paste that URL into the **Apps Script Web App URL** field in the tool's Settings tab.

---

## Step 5 — Host on GitHub Pages

1. Create a new GitHub repository (public)
2. Upload `meeting-tool.html` and `SHEET_SETUP.md`
3. Go to **Settings → Pages**
4. Under "Source" choose **main branch**, folder **/ (root)**
5. Click **Save**
6. Your tool will be live at: `https://[yourusername].github.io/[repo-name]/meeting-tool.html`

---

## How It Works (Summary)

| Action | What Happens |
|---|---|
| Open the tool | Reads quantities from Google Sheet (published CSV) |
| Enter counts on Literature/Chip tab | Stored in memory only — not saved |
| Click "Generate Order" | Computes what needs ordering and shows printable text |
| Click "Copy" or "Print" | Copies to clipboard or opens print dialog |
| Change qty in Settings → Save | Writes new quantities back to Google Sheet via Apps Script |

---

## Troubleshooting

**Quantities aren't loading** — Make sure your sheet is published (Step 2) and the Sheet ID is correct. Tab names must be exactly `Literature` and `Chips`.

**Save isn't syncing to sheet** — The Apps Script URL may not be set, or authorization may be needed. Check Step 4.

**CORS error in console** — Your sheet must be published publicly (Step 2).
