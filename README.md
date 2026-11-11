# Weibo Scraper

![Weibo Scraper: posts, comments, profiles and the hot-search board (热搜) without login](https://raw.githubusercontent.com/smoldesk/apify-scrapers/main/assets/weibo/banner.webp)

Scrape Weibo (微博) posts, comments, profiles and the hot-search board (热搜). No login, no cookies, no API key. Runs on [Apify](https://apify.com/smoldesk/weibo-scraper).

## What you get

- **Keyword search** in Chinese or English, or a #hashtag, within a date range. Newest first.
- **Posts**: full text (long posts expanded), likes, comments, reposts, images, video, topics, @mentions, the reposted post, and the author with follower count and verification reason. Each post has a `region` with an English name in `regionEn`, and an `isPromoted` flag from Weibo's own ad marker.
- **Comments and replies**: text, likes, author and region (with `regionEn`). Comments written by Weibo's AI assistants are flagged `isAiGenerated`.
- **Profiles**: followers, following, post count, verification, bio, location, total likes.
- **Hot search (热搜)**: rank, topic, heat value and label. Schedule it hourly to track a topic.
- **Past Weibo's page limit**: Weibo shows about 650 posts per search, and the scraper keeps going back in time on its own until it reaches your post limit, so you can collect thousands.
- **Approximate sentiment** (optional, free): `sentiment` and `sentimentScore` on posts and comments, from a word list and Weibo emoji.
- **Only new posts** mode for daily or hourly keyword monitoring.

## Brand monitoring & China market research

Track what Chinese consumers say about a brand, product or competitor on Weibo, from outside China: no Weibo account, no Chinese phone number, no VPN. Search the brand in Chinese and English, switch on **Only new posts**, add an hourly or daily schedule, and send the results to Google Sheets, Slack or a webhook. Switch on **Add approximate sentiment** for a quick positive, negative or mixed label on every comment, or feed the `text` of posts and comments to your own LLM for better sentiment scores, translation or a daily English summary.

Ready-made examples: [Monitor a brand on Weibo (new posts only)](https://apify.com/smoldesk/weibo-scraper/examples/weibo-brand-monitoring) · [Weibo posts + comments for sentiment research](https://apify.com/smoldesk/weibo-scraper/examples/weibo-posts-comments-sentiment) · [Weibo hot search board](https://apify.com/smoldesk/weibo-scraper/examples/weibo-hot-search-board)

## Run it

Open the [Weibo Scraper on Apify](https://apify.com/smoldesk/weibo-scraper), fill in the form and click Start. Or call it from code with your [Apify token](https://console.apify.com/settings/integrations):

```bash
curl -X POST "https://api.apify.com/v2/acts/smoldesk~weibo-scraper/run-sync-get-dataset-items?token=YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "searchQueries": ["小米", "#新能源汽车#"],
    "postedAfter": "2026-10-01",
    "postedBefore": "2026-10-15",
    "maxPostsPerSearch": 200,
    "includeComments": true,
    "maxCommentsPerPost": 20
  }'
```

Get the hot-search board only: `{"searchQueries": [], "trending": "hotSearch"}`.

Collect more than 650 posts for a keyword: raise `maxPostsPerSearch` (up to 20,000). Large runs take minutes, so raise the run timeout too.

## Sample output

```json
{
  "type": "post",
  "url": "https://weibo.com/7849268899/Rn3IJg938",
  "text": "#金玟庭[超话]#  我可以手把手教，我是iPhone",
  "publishedAt": "2026-10-17T02:44:18+00:00",
  "region": "云南",
  "regionEn": "Yunnan",
  "isPromoted": false,
  "authorName": "Jiiouo_w",
  "likesCount": 1,
  "commentsCount": 0,
  "repostsCount": 0,
  "topics": ["金玟庭[超话]"],
  "images": ["https://wx3.sinaimg.cn/large/008zcJphgy1ii55lkpi94j30dt0u0taf.jpg"],
  "searchQuery": "iPhone",
  "sentiment": null,
  "sentimentScore": null
}
```

```json
{
  "type": "trend",
  "rank": 1,
  "word": "拿打工人血汗钱给女儿铺星路",
  "label": "新",
  "heat": 1124302,
  "url": "https://s.weibo.com/weibo?q=%23%E6%8B%BF%E6%89%93%E5%B7%A5%E4%BA%BA%E8%A1%80%E6%B1%97%E9%92%B1%E7%BB%99%E5%A5%B3%E5%84%BF%E9%93%BA%E6%98%9F%E8%B7%AF%23"
}
```

Export as JSON, CSV or Excel, or pipe it into Google Sheets, Make, Zapier or n8n.

## Pricing

Pay per result. No monthly fee.

| Result | Per 1,000 |
|---|---|
| Post or profile | $8 |
| Comment or reply | $1 |
| Hot-search topic | $1 |

Typical runs: 100 posts for one keyword is about $0.80. 20 posts, 1 profile, 5 hot topics and 8 comments cost $0.18 in a real run. Comments are the cheap lever: 1,000 comments cost what 125 posts cost. Set a maximum cost per run and the scraper stops there.

## Limits

- No user timelines and no follower lists. Weibo needs a login for both. Search the person's name or hashtag instead.
- Weibo shows about 650 posts per search. The scraper continues past that automatically, newest first, until `maxPostsPerSearch` or your `postedAfter` date. Roughly 1,500 posts take 5 minutes.
- **Only new posts** filters keyword searches only. Post links, profiles and hot-search topics are returned on every run.
- `region` and `client` are sometimes `null`: Weibo does not send them for every post.
- Search is newest first only.

## FAQ

**Does Weibo have a public API?**
The official one needs an approved developer app and mostly returns your own account. This scraper reads the public pages any visitor sees and returns clean JSON with English field names.

**How do I scrape Weibo comments?**
Set `includeComments` to true. `maxCommentsPerPost` and `maxRepliesPerComment` control how deep it goes. You can also pass post links in `postUrls` to get comments for specific posts.

**Can I use it from Python?**
Yes. `pip install apify-client`, then call `client.actor("smoldesk/weibo-scraper").call(run_input={...})` and read the dataset. Same input as the curl example above.

**How accurate is the sentiment?**
It is an approximation from a word list and emoji, not a language model. When it gives an answer it was usually right in our own small check on real comments, but it leaves many comments as `null` when there is no clear signal. News-style posts usually come back `null`. Sarcasm and slang are often wrong. For better accuracy, send the `text` to your own LLM.

**Do I need a Weibo account, a Chinese phone number or a VPN?**
No. Nothing to log in to and no proxy to set up.

**Can I track Weibo hot search over time?**
Yes. Schedule a run every hour with `"trending": "hotSearch"` and compare `rank` and `heat` between runs.

## More scrapers

Weibo is one of 20 no-login scrapers by Smoldesk Studios. See the full list: [smoldesk/apify-scrapers](https://github.com/smoldesk/apify-scrapers).
