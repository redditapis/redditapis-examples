# redditapis-examples

> Official code samples for [RedditAPI](https://redditapis.com), the pay-per-call Reddit data API. Reads from $0.002, no subscription, no Reddit developer app.

[![Docs](https://img.shields.io/badge/docs-redditapis.com-FF4500)](https://docs.redditapis.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-FF4500)](LICENSE)
[![Price](https://img.shields.io/badge/from-%240.002%2Fcall-FF4500)](https://redditapis.com/pricing)
[![Free credit](https://img.shields.io/badge/free%20credit-%240.50-FF4500)](https://redditapis.com)

Code samples for every supported language: curl, Python, Node.js, Go, Rust, PHP, Ruby, Java.

## Quick start

1. Sign up at [redditapis.com](https://redditapis.com) (gets $0.50 free credit, no card required).
2. Copy your API key from the dashboard.
3. Set as env var: `export REDDITAPIS_KEY=...`
4. Run any example below.

Base URL: `https://api.redditapis.com`
Authentication: `Authorization: Bearer ${REDDITAPIS_KEY}` header on every request.

---

## curl

```bash
# Fetch hot posts from a subreddit
curl -H "Authorization: Bearer $REDDITAPIS_KEY" \
  "https://api.redditapis.com/r/programming/hot?limit=25"

# Get a single post by ID with its top comments
curl -H "Authorization: Bearer $REDDITAPIS_KEY" \
  "https://api.redditapis.com/post/1abc234/comments?sort=top&limit=50"

# Search Reddit for a query, limited to a subreddit
curl -H "Authorization: Bearer $REDDITAPIS_KEY" \
  "https://api.redditapis.com/search?q=postgres+sharding&subreddit=programming&sort=top&t=year"

# User profile + recent submissions
curl -H "Authorization: Bearer $REDDITAPIS_KEY" \
  "https://api.redditapis.com/user/spez/about"
curl -H "Authorization: Bearer $REDDITAPIS_KEY" \
  "https://api.redditapis.com/user/spez/submitted?limit=10"

# Vote up a post (account-bound, requires write scope)
curl -X POST -H "Authorization: Bearer $REDDITAPIS_KEY" \
  "https://api.redditapis.com/vote/up?id=t3_1abc234"
```

---

## Python

Install: `pip install requests`

```python
import os
import requests

API_KEY = os.environ["REDDITAPIS_KEY"]
BASE = "https://api.redditapis.com"
HEADERS = {"Authorization": f"Bearer {API_KEY}"}


def get_subreddit_hot(subreddit: str, limit: int = 25) -> dict:
    r = requests.get(f"{BASE}/r/{subreddit}/hot", params={"limit": limit}, headers=HEADERS)
    r.raise_for_status()
    return r.json()


def get_post_comments(post_id: str, sort: str = "top", limit: int = 50) -> dict:
    r = requests.get(
        f"{BASE}/post/{post_id}/comments",
        params={"sort": sort, "limit": limit},
        headers=HEADERS,
    )
    r.raise_for_status()
    return r.json()


def search(query: str, subreddit: str | None = None, sort: str = "top", t: str = "year") -> dict:
    params = {"q": query, "sort": sort, "t": t}
    if subreddit:
        params["subreddit"] = subreddit
    r = requests.get(f"{BASE}/search", params=params, headers=HEADERS)
    r.raise_for_status()
    return r.json()


def get_user(username: str) -> dict:
    r = requests.get(f"{BASE}/user/{username}/about", headers=HEADERS)
    r.raise_for_status()
    return r.json()


if __name__ == "__main__":
    posts = get_subreddit_hot("programming", limit=10)
    for p in posts["data"]["children"]:
        print(p["data"]["title"])
```

### PRAW replacement pattern

If you are migrating from PRAW:

```python
# Before (PRAW, requires Reddit developer app + OAuth)
import praw
reddit = praw.Reddit(
    client_id="...",
    client_secret="...",
    username="...",
    password="...",
    user_agent="my-app/1.0",
)
for submission in reddit.subreddit("programming").hot(limit=25):
    print(submission.title)

# After (RedditAPI, single bearer token)
import os, requests
r = requests.get(
    "https://api.redditapis.com/r/programming/hot",
    params={"limit": 25},
    headers={"Authorization": f"Bearer {os.environ['REDDITAPIS_KEY']}"},
)
for child in r.json()["data"]["children"]:
    print(child["data"]["title"])
```

---

## Node.js

```javascript
const API_KEY = process.env.REDDITAPIS_KEY;
const BASE = "https://api.redditapis.com";
const headers = { Authorization: `Bearer ${API_KEY}` };

async function getSubredditHot(subreddit, limit = 25) {
  const r = await fetch(`${BASE}/r/${subreddit}/hot?limit=${limit}`, { headers });
  if (!r.ok) throw new Error(`${r.status} ${await r.text()}`);
  return r.json();
}

async function search(query, { subreddit, sort = "top", t = "year" } = {}) {
  const params = new URLSearchParams({ q: query, sort, t });
  if (subreddit) params.set("subreddit", subreddit);
  const r = await fetch(`${BASE}/search?${params}`, { headers });
  if (!r.ok) throw new Error(`${r.status} ${await r.text()}`);
  return r.json();
}

const posts = await getSubredditHot("programming", 10);
for (const c of posts.data.children) console.log(c.data.title);
```

---

## Go

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
)

func main() {
	key := os.Getenv("REDDITAPIS_KEY")
	req, _ := http.NewRequest("GET",
		"https://api.redditapis.com/r/programming/hot?limit=25", nil)
	req.Header.Set("Authorization", "Bearer "+key)
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()
	body, _ := io.ReadAll(resp.Body)
	fmt.Println(string(body))
}
```

---

## Rust

`Cargo.toml`:

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json"] }
tokio = { version = "1", features = ["full"] }
```

`src/main.rs`:

```rust
use std::env;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let key = env::var("REDDITAPIS_KEY")?;
    let resp = reqwest::Client::new()
        .get("https://api.redditapis.com/r/programming/hot?limit=25")
        .bearer_auth(key)
        .send().await?
        .text().await?;
    println!("{resp}");
    Ok(())
}
```

---

## PHP

```php
<?php
$key = getenv("REDDITAPIS_KEY");
$ch = curl_init("https://api.redditapis.com/r/programming/hot?limit=25");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, ["Authorization: Bearer $key"]);
$resp = curl_exec($ch);
curl_close($ch);
echo $resp;
```

---

## Ruby

```ruby
require "net/http"
require "json"

uri = URI("https://api.redditapis.com/r/programming/hot?limit=25")
req = Net::HTTP::Get.new(uri)
req["Authorization"] = "Bearer #{ENV['REDDITAPIS_KEY']}"
resp = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) { |http| http.request(req) }
puts JSON.pretty_generate(JSON.parse(resp.body))
```

---

## Java

```java
import java.net.URI;
import java.net.http.*;

public class HotPosts {
  public static void main(String[] args) throws Exception {
    String key = System.getenv("REDDITAPIS_KEY");
    HttpRequest req = HttpRequest.newBuilder(
        URI.create("https://api.redditapis.com/r/programming/hot?limit=25"))
        .header("Authorization", "Bearer " + key)
        .GET()
        .build();
    HttpResponse<String> r = HttpClient.newHttpClient()
        .send(req, HttpResponse.BodyHandlers.ofString());
    System.out.println(r.body());
  }
}
```

---

## Links

- Documentation: [docs.redditapis.com](https://docs.redditapis.com)
- Pricing: [redditapis.com/pricing](https://redditapis.com/pricing)
- Contact: [emma@redditapis.com](mailto:emma@redditapis.com)

License: MIT.
