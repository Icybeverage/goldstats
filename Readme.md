# Audius x GoldStats Analytics

This project integrates the **Audius API** with **Google Sheets** to provide real-time analytics for trending tracks, including key metrics like play count, repost count, favorite count, engagement rate, virality score, and monetization potential.

## Features

- **Audius API Integration**: Fetches trending track data from the Audius API.
- **Google Sheets Integration**: The data is updated in a connected Google Sheet.
- **Real-Time Analytics Dashboard**: The data is visualized using Google Looker Studio.

## Live Dashboard

You can view the live dashboard here:
[GoldStats Analytics Dashboard](https://lookerstudio.google.com/u/0/reporting/8876d471-65d1-4ecb-9d3d-c3d3a28bb786/page/ylwDE)

## Script: Fetch Audius Track Analytics

The script uses Google Apps Script to fetch trending tracks and update Google Sheets with the following metrics:
- Track Title
- Artist Name
- Play Count
- Repost Count
- Favorite Count
- Engagement Rate
- Virality Score
- Monetization Potential

```javascript
async function fetchAudiusTrackAnalytics() {
  const spreadsheetId = '1ac5u7JLRSKahQUx4iOoxYppEgy0QMsEOES1ItQBXqgw';
  var spreadsheet = SpreadsheetApp.openById(spreadsheetId);
  var sheet = spreadsheet.getSheetByName('Sheet1');
  
  const responseHost = UrlFetchApp.fetch('https://api.audius.co');
  const hostData = JSON.parse(responseHost.getContentText());
  const host = hostData.data[0];

  const apiUrl = `${host}/v1/tracks/trending?app_name=GoldStats&limit=10`;

  try {
    const response = UrlFetchApp.fetch(apiUrl);
    const data = JSON.parse(response.getContentText());

    sheet.clear();

    sheet.appendRow([
      'Track Title', 'Artist Name', 'Play Count', 'Repost Count', 'Favorite Count',
      'Genre', 'Release Date', 'Duration (sec)', 'Engagement Rate', 'Virality Score',
      'Is Downloadable', 'Has Current User Reposted', 'Monetization Potential'
    ]);

    for (const track of data.data) {
      const releaseDate = new Date(track.release_date);
      const daysSinceRelease = Math.floor((new Date() - releaseDate) / (1000 * 60 * 60 * 24));
      
      const totalEngagements = track.repost_count + track.favorite_count;
      const engagementRate = (totalEngagements / track.play_count * 100).toFixed(2);
      const viralityScore = ((track.repost_count / track.play_count) * 100).toFixed(2);

      const monetizationPotential = calculateMonetizationPotential(track, engagementRate, viralityScore);

      sheet.appendRow([
        track.title,
        track.user.name,
        track.play_count,
        track.repost_count,
        track.favorite_count,
        track.genre,
        track.release_date,
        track.duration,
        engagementRate + '%',
        viralityScore + '%',
        track.downloadable ? 'Yes' : 'No',
        track.has_current_user_reposted ? 'Yes' : 'No',
        monetizationPotential
      ]);
    }

  } catch (error) {
    Logger.log('Error fetching data: ' + error);
  }
}

function calculateMonetizationPotential(track, engagementRate, viralityScore) {
  const playWeight = 0.5;
  const engagementWeight = 0.3;
  const viralityWeight = 0.2;

  const playScore = Math.min(track.play_count / 10000, 1);
  const engagementScore = engagementRate / 100;
  const viralScore = viralityScore / 100;

  const potentialScore = (playScore * playWeight) + (engagementScore * engagementWeight) + (viralScore * viralityWeight);

  if (potentialScore > 0.8) return 'Very High';
  if (potentialScore > 0.6) return 'High';
  if (potentialScore > 0.4) return 'Medium';
  if (potentialScore > 0.2) return 'Low';
  return 'Very Low';
}
