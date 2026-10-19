# YouTube Analytics Tracker

A Python data pipeline that collects YouTube channel and video statistics through the **YouTube Data API v3**. It stores daily snapshots in **SQLite** and visualises channel performance and growth in a **Power BI** dashboard.

```
YouTube Data API v3  →  Python (requests)  →  SQLite  →  CSV (pandas)  →  Power BI
```

## Features

- **Track any channel**: paste a channel link (`/channel/<id>`, `/@handle` or `/c/<name>`)
- **One-command refresh**: update every tracked channel without re-entering links
- **Daily snapshots**: views, likes and comments for each channel's 20 most recent videos, stored by date to show growth over time
- **Channel stats**: subscribers, total views and video count (latest values)
- **Power BI dashboard**: KPI cards, channel filter, views-over-time trend and a video table

## Tech Stack

| Purpose | Tools |
|---------|-------|
| Language | Python |
| Data source | YouTube Data API v3 (`requests`) |
| Storage | SQLite (`sqlite3`) |
| Export | pandas → CSV |
| Visualisation | Power BI Desktop (Power Query, DAX) |
| Config | python-dotenv |

## Project Structure

```
youtube-analytics/
├── init_db.py               # Creates the SQLite tables
├── fetch_youtube_data.py    # Main script: add / refresh channels
├── export_to_csv.py         # Exports tables to CSV for Power BI
├── check_db.py              # Lists tables (debug)
├── check_data.py            # Prints sample rows (debug)
├── youtube_analytics.pbix   # Power BI dashboard
└── requirements.txt
```

## Database Schema

| Table | Columns | Notes |
|-------|---------|-------|
| `channels` | channel_id (PK), channel_name, subscriber_count, total_views, video_count, last_fetched | One row per channel, updated each run |
| `videos` | video_id (PK), channel_id (FK), title, publish_date | One row per tracked video |
| `video_stats` | video_id (FK), fetched_date, views, likes, comments | One row per video per run (the history) |

## How It Works

For each channel, `fetch_youtube_data.py`:

1. Gets channel details and statistics (`channels.list`)
2. Finds the 20 most recent videos (`search.list`)
3. Gets views, likes and comments for those videos in one call (`videos.list`)
4. Saves everything to `database.db`: the channel row is updated, new videos are added, and a dated stats snapshot is appended

If one channel fails (for example, because the quota ran out), the error is printed and the remaining channels still update.

## Getting Started

### Prerequisites

- Python 3.10+
- Power BI Desktop (Windows), for the dashboard
- A YouTube Data API v3 key: in [Google Cloud Console](https://console.cloud.google.com/), create a project, enable **YouTube Data API v3**, then go to **Credentials → Create API key**

### Installation

```bash
git clone <repository-url>
cd youtube-analytics

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Create a `.env` file:

```
YOUTUBE_API_KEY=your_api_key_here
```

### Usage

**1. Create the database (first time only)**

```bash
python init_db.py
```

**2. Add channels or refresh data**

```bash
python fetch_youtube_data.py
```

```
YouTube Analytics Tracker
1. Refresh existing channels
2. Add a new channel
Select an option (1 or 2):
```

- Choose **2** and paste a channel link to start tracking it.
- Choose **1** to update all tracked channels. Run this once a day to build the history.

**3. Export for Power BI**

```bash
python export_to_csv.py
```

This creates `channels.csv`, `videos.csv` and `video_stats.csv`.

**4. View the dashboard**

1. Open `youtube_analytics.pbix` in Power BI Desktop.
2. Point the queries at your CSV files: **Transform data → Data source settings → Change Source**.
3. Click **Refresh**.

## Dashboard

- **KPI cards**: Total Subscribers, Total Views, Total Videos, Total Engagement (likes + comments)
- **Channel slicer**: filter every visual by channel
- **Views over time**: line chart of tracked video views by fetch date
- **Video table**: views and likes per video
- **Detailed Analytics page**: KPI cards with a channel views growth chart

## Notes

- **Run the refresh once per day.** Each run appends a snapshot; running twice on the same day stores duplicate rows for that date.
- **API quota:** every channel refresh uses one `search.list` request, and Google's default quota allows only a limited number of searches per day.
- Video titles and publish dates aren't fetched yet, so videos are identified by their video ID.
- `.env`, `database.db` and the CSV files are gitignored. Never commit your API key.
