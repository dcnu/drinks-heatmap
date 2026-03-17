# Drinks Heatmap

A GitHub-style heatmap for alcoholic drinks logged in Apple Health. An iOS Shortcut queries HealthKit, encodes the data into a URL fragment, and opens a static page that renders the chart client-side.

No backend. No database. No tracking. All data stays on-device and in the URL fragment (which is never sent to the server).

## iOS Shortcut Setup

The Shortcut queries Apple Health for alcoholic drink samples, aggregates them by date, and opens the heatmap page with the data encoded in the URL.

### Steps

- **Find Health Samples**
	- Type: Alcoholic Drinks
	- Filter: Start Date is on or after January 1 of the current year
	- Sort: Start Date, Oldest First

- **Dictionary** (empty, 0 items)

- **Set Variable** `totals` to the Dictionary output

- **Repeat with Each** item in Find Health Samples result:
	- **Format Date** on Repeat Item (select the Date property)
		- Date Format: Custom
		- Format String: `yyyy-MM-dd`
	- **Get Dictionary Value** for Formatted Date in `totals`
	- **If** Dictionary Value has any value:
		- **Calculate**: Dictionary Value + Repeat Item
	- **Otherwise**:
		- **Number**: Repeat Item
	- **End If**
	- **Set Dictionary Value**: set Formatted Date to If Result in `totals`
	- **Set Variable** `totals` to the Set Dictionary Value output
- **End Repeat**

- **Get Dictionary Value**: All Keys in `totals`

- **Repeat with Each** item in All Keys:
	- **Get Dictionary Value** for Repeat Item in `totals`
	- **Text**: `[Repeat Item],[Dictionary Value]`
- **End Repeat**

- **Combine Text**: Repeat Results with New Lines
- **URL Encode**: Combined Text
- **Text**: `https://your-domain.com/#[URL Encoded Text]`
- **Open URL**: the Text output

### Notes

- The Set Variable `totals` step after Set Dictionary Value inside the loop is required. Without it, dictionary mutations are discarded between iterations.
- If the Shortcut is slow (hundreds of samples), you can skip the aggregation loop entirely: output each sample as `date,1` and let the page sum duplicates.
- URL length is safe under ~2,000 characters. Sparse data (drinks on fewer than 150 days) stays well under this limit.

## Hosting

The site is a single `index.html` file with no dependencies. Deploy to any static host (Vercel, Cloudflare Pages, Netlify, etc.).

## Usage

### Via Shortcut

Run the Shortcut. It opens the page in broswer with the heatmap pre-rendered.

### Manual

Open the page directly. Paste CSV data (`date,count` per line) into the text area and click Render.

### Copy as Image

Click "Copy Image" below the heatmap to copy a PNG to the clipboard. Falls back to a file download if clipboard access is unavailable.

## CSV Format

```
2026-01-01,2
2026-01-03,1
2026-01-10,3
```

- One row per date
- Dates not present default to 0
- Duplicate dates are summed
- Only the current calendar year (Jan 1 through today) is rendered

## Privacy

- No data is sent to any server
- The URL fragment (`#...`) is excluded from HTTP requests per RFC 3986
- No analytics, cookies, or third-party scripts