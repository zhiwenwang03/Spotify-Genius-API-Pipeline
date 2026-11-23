# Spotify-Genius API Pipeline: The Weeknd

This project pulls album and track data for **The Weeknd** from the **Spotify Web API** and song lyrics from the **Genius API**, combines the two datasets, and runs exploratory data analysis (EDA).

**Artist:** The Weeknd

**Albums (6 standard studio albums):** *Kiss Land* (2013), *Beauty Behind The Madness* (2015), *Starboy* (2016), *After Hours* (2020), *Dawn FM* (2022), *Hurry Up Tomorrow* (2025)

## APIs Used
| Part | API | What it provides |
|---|---|---|
| Part 1 | Spotify Web API (Client Credentials flow) | Albums, track names, track numbers, durations, explicit flag, release date, track ID |
| Part 2 | Genius API (via the `lyricsgenius` package) | Lyrics, Genius URL, album release date, annotation count |

## Project Structure
```
Spotify-Genius-API-Pipeline/
├── README.md
├── requirements.txt
├── .env                      # API keys (NOT included in the submission)
├── data/
│   ├── spotify_tracks.csv    # output of Part 1
│   ├── genius_lyrics.csv     # output of Part 2
│   └── combined_tracks.csv   # output of Part 3 (merged dataset)
└── notebooks/
    ├── 01_spotify_pull.ipynb # Part 1: Spotify data pull
    ├── 02_genius_pull.ipynb  # Part 2: Genius lyrics pull
    └── 03_combine_eda.ipynb  # Part 3: merge + EDA
```

## Environment Setup
This project uses a conda environment with Python 3.12.

```bash
conda create -n api_practice python=3.12 -y
conda activate api_practice
pip install -r requirements.txt
```

Then launch Jupyter from the project root:
```bash
jupyter notebook
```

## API Keys
All keys are stored in a `.env` file in the project root (same level as this README). The `.env` file is **not** included in the submission, so create your own:

```
SPOTIFY_CLIENT_ID=your_spotify_client_id
SPOTIFY_CLIENT_SECRET=your_spotify_client_secret
GENIUS_ACCESS_TOKEN=your_genius_access_token
```

No spaces around `=` and no quotes.

**How to get the keys:**
- **Spotify** (requires a Spotify Premium account):
  1. Go to https://developer.spotify.com/dashboard and click **Create app**.
  2. Enter any Redirect URI (e.g. `http://127.0.0.1:8888/callback`) and select **Web API**.
  3. Open the app's **Settings** to copy the Client ID and Client Secret.
- **Genius**:
  1. Go to https://genius.com/api-clients and click **New API Client**.
  2. Enter any App Website URL (e.g. `http://127.0.0.1`).
  3. Click **Generate Access Token** and copy the token.

The notebooks load these values with `python-dotenv`, so no keys appear in the code.

## How to Run
Run the notebooks **in order**, since each one reads the CSV produced by the previous step:

1. **`01_spotify_pull.ipynb`**: gets the artist ID, filters to the 6 studio albums, pulls all tracks → `data/spotify_tracks.csv` (94 tracks)
2. **`02_genius_pull.ipynb`**: searches Genius for each Spotify track and scrapes the lyrics → `data/genius_lyrics.csv` (93 tracks; takes about 5–10 minutes)
3. **`03_combine_eda.ipynb`**: standardizes column names, resolves name mismatches, merges on album + song name → `data/combined_tracks.csv`, then runs the EDA

## Data Notes
- **Name mismatches:** Spotify and Genius format names differently (capitalization, curly vs. straight apostrophes, "(feat. ...)" in titles, album edition names). A naive merge matched only 54 of 94 songs. Each Genius row stores the Spotify title it was searched with, and the Spotify names are used as the standard, which brings the match to 93 of 94.
- **Missing song:** *Until We're Skin & Bones* (*Hurry Up Tomorrow*) was not found on Genius, so it has no lyrics in the combined dataset.
- **Manual fix:** The Genius search for *Earned It* first returned a Marian Hill remix page. It is replaced with the original song in `02_genius_pull.ipynb`.
- **Spotify release date:** Spotify lists *Kiss Land* as 2013-01-01. The actual release date is 2013-09-10, which matches Genius.
- **Spotify token:** The access token expires after 1 hour. If a request returns `401`, re-run the token cell.

## EDA Insights (summary)
1. **Spotify:** Average track length dropped from 5.57 min (*Kiss Land*) to 3.24 min (*Dawn FM*). The share of explicit tracks has no clear time trend: it is highest on *Beauty Behind The Madness* (92.9%) and lowest on *Dawn FM* (25.0%).
2. **Genius:** Average lyric word count fell from 480 (*Kiss Land*) to 288 (*Dawn FM*). Word count and Genius annotation count have a moderate positive correlation (r = 0.44).

See `03_combine_eda.ipynb` for the full tables and plots.