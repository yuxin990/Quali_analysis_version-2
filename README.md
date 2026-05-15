# Qualitative Thematic Analysis Agent — Version 2
### Academic AI Workflow for Politics, International Studies, Geopolitics & AI Governance

An automated three-stage qualitative research workflow built on n8n and Claude AI. Upload a PDF or Word document to Google Drive and receive a full academic thematic analysis — open coding, theme clustering, AI-specific discourse coding, and structured results saved to Google Sheets.

---

## What's New in V2

| Feature | V1 | V2 |
|---|---|---|
| Analysis stages | 2 | **3** |
| Domain specialisation | Generic | **Politics, IR, Geopolitics, AI Governance** |
| Code output | Basic codes | **Codes + definition + quotation + reasoning** |
| Theme output | Basic themes | **Themes + importance reasoning + analytical memo** |
| AI discourse layer | None | **Step 3: classifies by dimension** |
| Dimensions | — | Technological, Ethical, Societal, Geopolitical, Governance, Futures |
| Rate limiting | Manual | **Built-in 30s buffer between chunks** |
| Output schema | Flat JSON | **Nested: step1 codes + step2 clusters + step3 AI codes** |

---

## What the Workflow Does

### Stage 1 — Open Coding
Reads each text chunk and generates granular qualitative codes. Every code includes:
- A descriptive label (`snake_case`)
- A concise definition
- An exact verbatim quotation from the source text
- Analytical reasoning

### Stage 2 — Thematic Clustering
Groups all codes into 3–7 higher-level themes. Each theme includes:
- Theme name and description
- Supporting codes and direct evidence quotes
- Importance reasoning relative to the research question
- Overall finding answering the research question
- Analytical memo noting tensions, gaps, and patterns

### Stage 3 — AI-Specific Discourse Coding
Scans all codes for AI-related content and classifies it by dimension:
- **Technological** — capabilities, systems, infrastructure
- **Ethical** — values, fairness, accountability, transparency
- **Societal** — social impacts, inequality, labour, democracy
- **Geopolitical** — state power, competition, digital sovereignty
- **Governance** — regulation, policy, standards, oversight
- **Futures** — predictions, scenarios, risks, opportunities

---

## Supported Input Formats

- PDF (text-based) — extracted directly via Claude AI
- Word documents (.docx) — extracted via n8n
- Interview transcripts
- Academic articles and reports
- Policy documents

---

## Requirements

- [n8n](https://n8n.io) account (cloud or self-hosted)
- [Anthropic API key](https://console.anthropic.com) (Claude access)
- Google account (Google Drive + Google Sheets)

---

## How to Import into n8n

1. Log in to your n8n instance
2. Go to **Workflows** → click **+** → **Import from file**
3. Select `qualitative-analysis-v2.n8n.json`
4. The workflow opens in the canvas — do not activate yet

---

## Credentials Setup

Set up three credentials in n8n under **Settings → Credentials → Add Credential**.

### 1. Anthropic API Key
- Type: **Header Auth**
- Name: `Anthropic API`
- Header Name: `x-api-key`
- Header Value: your key (`sk-ant-...`)

Get your key at [console.anthropic.com](https://console.anthropic.com) → API Keys.

Apply to nodes: `HTTP: Extract PDF Text`, `HTTP: Step1 Code Generation`, `HTTP: Step2 Code Clustering`, `HTTP: Step3 AI Discourse Coding`

### 2. Google Drive
- Type: **Google Drive OAuth2 API**
- Click **Sign in with Google**

Apply to nodes: `Google Drive Trigger`, `Google Drive: Download File`

### 3. Google Sheets
- Type: **Google Sheets OAuth2 API**
- Click **Sign in with Google**

Apply to node: `Google Sheets: Save V2 Results`

---

## Google Sheets Setup

1. Create a new Google Sheet
2. Rename the bottom tab to exactly: **`V2_Analysis`**
3. Add these column headers in Row 1:

```
source_name | research_question | domain | analysis_method | model_used | total_codes | total_themes | total_ai_codes | overall_finding | analytical_memo | ai_discourse_summary | clusters_json | ai_codes_json | raw_codes_json | completed_at | errors
```

4. Copy the Sheet ID from the URL:
```
https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit
```
5. Paste the Sheet ID into `Google Sheets: Save V2 Results` → Document field

---

## Node-by-Node Configuration

### Node 1 — `Google Drive Trigger`
- Credential: Google Drive OAuth2 API
- Trigger On: `Changes involving a Specific Folder`
- Folder: paste your Google Drive folder ID
- Watch For: `File Created`

Find your folder ID from its URL:
```
https://drive.google.com/drive/folders/YOUR_FOLDER_ID_HERE
```

### Node 2 — `Set: Research Config` ← main settings
Update these fields for your study:

| Field | Description | Example |
|---|---|---|
| `anthropic_api_key` | Your Anthropic API key | `sk-ant-api03-...` |
| `model` | Claude model | `claude-sonnet-4-6` |
| `research_question` | Your research question | `How is AI governance framed in policy discourse?` |
| `analysis_method` | Thematic method | `Braun & Clarke (2006) Thematic Analysis` |
| `domain` | Research domain | `politics, international studies, AI governance` |
| `chunk_size_words` | Words per chunk | `300` |
| `enable_ai_coding` | Run Step 3? | `true` or `false` |

### Nodes 3–9 — File Ingestion & Extraction
No configuration needed. The workflow automatically:
- Downloads the file from Google Drive
- Routes PDFs through Claude extraction
- Routes Word docs through n8n text extraction

### Node 12 — `Wait: Rate Limit Buffer`
- Resume: `After Time Interval`
- Amount: `30`
- Unit: `Seconds`

### Nodes 6, 13, 17, 20 — HTTP Request nodes (Claude API)
Each must be configured identically:
- Authentication: `Generic Credential Type`
- Generic Auth Type: `Header Auth`
- Credential: `Anthropic API`
- Headers: `anthropic-version: 2023-06-01` and `content-type: application/json`
- Specify Body: `JSON`
- Body: `={{ $json.claude_request }}`

### Node 22 — `Google Sheets: Save V2 Results`
- Credential: Google Sheets OAuth2 API
- Document ID: your Sheet ID
- Sheet: `V2_Analysis`
- Map each column using `={{ $json.metadata.field_name }}` expressions

---

## How to Run

1. Activate the workflow (toggle top right → Active)
2. Upload a PDF or Word file to your Google Drive folder
3. The workflow triggers automatically within seconds
4. Watch each node execute in the n8n execution log
5. Results appear as a new row in your `V2_Analysis` sheet

---

## Output Structure

```json
{
  "metadata": {
    "source_name": "article.pdf",
    "research_question": "How is AI governance framed?",
    "domain": "politics, international studies, AI governance",
    "analysis_method": "Braun & Clarke (2006)",
    "model_used": "claude-sonnet-4-6",
    "total_codes": 18,
    "total_themes": 5,
    "total_ai_codes": 4,
    "total_tokens_used": 12400
  },
  "overall_finding": "Direct answer to your research question...",
  "analytical_memo": "Tensions and patterns noted...",
  "step2_clusters": [
    {
      "theme": "Digital Sovereignty as State Strategy",
      "description": "...",
      "codes": ["state_ai_control", "data_localisation"],
      "supporting_quotes": ["exact verbatim quotes..."],
      "importance_reasoning": "...",
      "frequency": 5
    }
  ],
  "step1_codes": [
    {
      "code": "algorithmic_accountability_gap",
      "definition": "...",
      "quotation": "exact verbatim quote",
      "reasoning": "...",
      "chunk_index": 0,
      "source_name": "article.pdf"
    }
  ],
  "step3_ai_codes": [
    {
      "code": "ai_regulatory_fragmentation",
      "description": "...",
      "quotation": "exact verbatim quote",
      "dimension": "governance",
      "reasoning": "..."
    }
  ],
  "ai_discourse_summary": "...",
  "errors": []
}
```

---

## Troubleshooting

**Rate limit errors** — reduce `chunk_size_words` to `200` or switch model to `claude-haiku-4-5-20251001`

**PDF text too short** — large PDFs may be partially extracted. Split into sections before uploading

**Word documents garbled** — convert to PDF before uploading for best results

**Google Sheets not saving** — check the `V2_Analysis` tab exists and column headers match exactly

**Step 3 returns 0 AI codes** — the document may not contain AI-related discourse. Check `ai_discourse_summary` in the output for confirmation

**total_codes: 0** — open the n8n execution log and check `Code: Parse Step1 Codes` for errors. Usually a rate limit or API key issue

---

## Customisation

- **Change domain**: update `domain` in `Set: Research Config` — the prompts adapt automatically
- **Adjust theme count**: edit `Code: Build Step2 Request` prompt (default: 3–7 themes)
- **Add Notion output**: connect a Notion node after `Code: Parse Step3 & Final Output`
- **Skip AI coding**: set `enable_ai_coding: false` in `Set: Research Config`
- **Change model**: update `model` field — use `claude-opus-4-7` for maximum quality

---

## Related

- [Version 1 — General Qualitative Analysis](https://github.com/yuxin990/Quali_analysis_version-1)

---

## Citation

If you use this workflow in your research, please cite:

> Zhou, Y. (2026). *Qualitative Thematic Analysis Agent V2: Academic AI Workflow for Politics, International Studies and AI Governance* [n8n workflow]. GitHub. https://github.com/yuxin990/Quali_analysis_version-2

---

## License

MIT License — free to use, modify, and share with attribution.
