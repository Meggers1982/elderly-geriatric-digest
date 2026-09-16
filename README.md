# Elderly & Geriatric Research Digest

A GitHub Actions workflow that searches curated clinical geriatrics, dementia and cognitive decline, palliative care, rheumatology, and rehabilitation medicine journals on PubMed, filters out widely covered stories, runs a single Claude pass for journalist-ready summaries and pitch angles, and publishes results to a GitHub Pages dashboard.

## How it works

1. **PubMed search** - Queries journals by ISSN for studies published in the past 7 days
2. **Title screening** - Prioritizes studies with novelty signals and excludes animal-only studies
3. **SERPAPI media filter** - Checks Google News and skips any study with 3+ news results
4. **Abstract fetch** - Retrieves full abstracts for shortlisted studies
5. **Claude pass** - Writes structured JSON: headline, summary, why it matters, caveats, relevance score, and pitch angles per publication type
6. **Artifact upload** - Saves JSON results as a GitHub Actions artifact
7. **Deploy job** - Downloads all job artifacts, merges and deduplicates by PMID, commits `data/results.json`, serves via GitHub Pages
8. **Dashboard publish** - Pushes the merged results to the shared research-digest-dashboard repo

## Dashboard

Features:
- Card view per study with headline, summary, caveats, fact-check notes
- Expandable pitch angles section for publications such as STAT News, The New York Times (Health), NPR Health, KFF Health News, Next Avenue, AARP The Magazine, Health.com, Women's Health Magazine, MedPage Today, Modern Healthcare, McKnight's Senior Living, The Atlantic, and general health outlets
- Filter by category, groundbreaking type, status, date range, and score
- Search across all study text and pitches
- Status tracking (New / Saved / Pitched / Passed) saved to localStorage
- Deduplication across runs by PMID

## Schedule

Runs automatically every morning at 7:00 AM ET. All jobs run in parallel; the deploy job merges results and publishes the dashboard once complete.

Can also be triggered manually via **Actions -> Elderly & Geriatric Research Digest -> Run workflow**.

## Categories

| Category | Journals | Jobs |
|---|---:|---|
| Geriatrics | 108 | 2 (chunks 1-2) |
| Neurology | 384 | 2 (chunks 1-2) |
| Palliative Care | 11 | 2 (chunks 1-2) |
| Rheumatology | 32 | 2 (chunks 1-2) |
| Physical and Rehabilitation Medicine | 62 | 2 (chunks 1-2) |

Large categories are split into chunks to keep run times under 20 minutes.

The category CSVs in `data/` are now hand-maintained and are the source of truth. `scripts/extract_journals.py` generated them from `~/PubMed_Journals_Categorized.xlsx`, which no longer exists, so the script has been deleted. Every row is searched with no topic filter, so a journal's entire weekly output enters the digest.

## Journal list audit (2026-09-14)

Method: OpenAlex's top sources for this digest's subject areas over the prior year were diffed against the CSVs (matched on any ISSN or title), and only titles PubMed indexes with at least 20 articles in the last 12 months were kept. The aging titles `senior-research-digest` added in its own audit the same day were checked too. The audit added 13 journals (584 → 597 rows):

- **Geriatrics** (+8): *npj Parkinson's Disease* (~360 PubMed articles a year), *Dementia & Neuropsychologia*, *Journal of Geriatric Cardiology*, *Geriatric Orthopaedic Surgery & Rehabilitation*, *Immunity & Ageing*, *European Journal of Ageing*, *European Review of Aging and Physical Activity*, *Dementia and Neurocognitive Disorders*
- **Palliative Care** (+1): *Palliative Care and Social Practice*
- **Rheumatology** (+4): *RMD Open*, *Osteoarthritis and Cartilage*, *Rheumatology Advances in Practice*, *BMC Rheumatology*

Left out on purpose:
- **Not in PubMed, or no PubMed articles in the past year**, so they would contribute nothing: *Revue du Rhumatisme*, *Modern Rheumatology Journal*, *Egyptian Rheumatology and Rehabilitation*, *Palliative Medicine in Practice*, *Journal of Population Ageing*, *Educational Gerontology*, *Gerontechnology*, *Ageing International*, *GeroPsych*, *International Journal of Ageing and Later Life*, *Progress in Palliative Care*, and dozens of Indonesian and Ukrainian physical-education journals that OpenAlex files under rehabilitation.
- **Under 20 PubMed articles a year**: *Alzheimer's & Dementia: Behavior & Socioeconomics of Aging* (launched 2025), *Aging Brain*, *Dementia and Geriatric Cognitive Disorders Extra*.
- **Off-beat** titles OpenAlex lumped in: *International Urogynecology Journal*, *Toxicon*, *Mediastinum*, *Advances in Wound Care*, *Cartilage*, neurosurgery titles, and pharmacy practice and education journals (*JAPhA*, *Research in Social and Administrative Pharmacy*, *International Journal of Pharmacy Practice*, *American Journal of Pharmaceutical Education*).
- **Case reports**: *Journal of Neurosurgery Case Lessons*, *Case Reports in Neurology*.
- **Lower priority** rheumatology titles, kept out to hold the list modest: *Lupus Science & Medicine* (skews toward younger patients), *EULAR Rheumatology Open*, *Osteoarthritis and Cartilage Open*, *Rheumatology and Therapy*, *Therapeutic Advances in Musculoskeletal Disease*, and small regional journals.

No category grew by more than a quarter, so the workflow chunking is unchanged.

## Manual Trigger

Go to **Actions -> Elderly & Geriatric Research Digest -> Run workflow**.

- Leave **category** blank to run all jobs
- Enter an exact category name, such as `Geriatrics`, to run just that category

## GitHub Pages Setup

1. Go to **Settings -> Pages**
2. Set source to **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Save; GitHub will serve `index.html` at the dashboard URL

## Required Secrets

Add these in **Settings -> Secrets and variables -> Actions**:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `SERPAPI_KEY` | SerpAPI key for Google News filtering |
| `SUPABASE_URL` | Supabase project URL (enables personalization from dashboard save/delete feedback) |
| `SUPABASE_KEY` | Supabase API key (read-only use; skips personalization if not set) |
| `DASHBOARD_REPO_TOKEN` | Token with push access to `Meggers1982/research-digest-dashboard` |

## Repo Structure

```text
.github/
  workflows/
    elderly-geriatric-digest.yml
scripts/
  elderly_geriatric_digest.py
  merge_results.py
data/
  Geriatrics.csv
  Neurology.csv
  Palliative Care.csv
  Rheumatology.csv
  Physical and Rehabilitation Medicine.csv
  results.json
index.html
requirements.txt
```

## Dashboard Study Card Fields

Each study card shows:

- **Headline** - plain-language present-tense summary
- **Relevance score** - 1-10, weighted for elder care, Alzheimer's disease, nursing and long-term care, and aging policy journalism fit
- **Category & journal** - source metadata
- **Groundbreaking type** - counterintuitive, overturns prior research, first-in-class, or domain-relevant finding
- **Media coverage** - SERPAPI verification status
- **The study** - what was done, who participated, and the key finding
- **Why it matters** - real-world significance for the target audience
- **Caveats** - limitations flagged automatically
- **Fact-check note** - corrections made during the Claude pass
- **Pitch angles** - expandable publication-specific pitch blocks
- **Status** - New / Saved / Pitched / Passed, tracked in your browser
