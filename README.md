# CINEVERSE Movies API (static JSON)

168 movies split into small files so you only fetch what you need. Host the repo on GitHub and read the files through raw URLs:

```
https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPOSITORY/main/<file>
```

## Files

| File | Use it for |
|---|---|
| `categories/<name>.json` | A row/page of one category. Every movie card includes the `overview`. |
| `movies.json` | All 168 movies with full details (keywords, all images, `review_count`). Use for search and the movie page. |
| `reviews.json` | `{ "<movie id>": [reviews] }`. Load only when a movie page opens. |
| `scripts/` | Maintenance scripts (not needed by your website). |

Categories: `top_rated`, `action`, `adventure`, `animation`, `comedy`, `crime`, `documentary`, `drama`, `family`, `fantasy`, `science_fiction`.
A movie can be in several categories; every movie lists them in `categories`.

## Fetching

```js
const BASE = "https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPOSITORY/main";
const getJSON = (path) => fetch(`${BASE}/${path}`).then((r) => {
  if (!r.ok) throw new Error(`${path}: ${r.status}`);
  return r.json();
});

const { movies } = await getJSON("categories/action.json");   // one category
const all = await getJSON("movies.json");                      // everything
const movie = all.find((m) => m.slug === "raya-and-the-last-dragon-2021");
const reviews = (await getJSON("reviews.json"))[movie.id] ?? [];
```

## Movie fields

| Field | Example / note |
|---|---|
| `id` | TMDB id |
| `imdb_id`, `imdb_url` | `tt5109280`, link to the IMDb page |
| `slug` | `raya-and-the-last-dragon-2021` (good for URLs) |
| `title`, `original_title`, `tagline` | `tagline` can be `null` |
| `overview` | synopsis |
| `year`, `release_date` | `2021`, `2021-03-03` |
| `runtime`, `runtime_text` | `107`, `1h 47m` |
| `vote_average`, `rating_percent`, `vote_count`, `popularity` | `7.8`, `78`, ... |
| `genres` | `["Animation", "Action"]` |
| `language`, `original_language` | `English`, `en` |
| `adult` | boolean |
| `poster_url`, `backdrop_url`, `logo_url` | ready-to-use full-size images (`/original`) |
| `poster_path`, `backdrop_path`, `logo_path` | raw TMDB paths if you want other sizes |
| `trailer_youtube_id`, `trailer_url`, `trailer_embed_url` | `null` when no trailer (Deep Blue) |
| `categories` | `["Top_Rated", "Action"]` |

`movies.json` also has `genre_ids`, `keywords`, `review_count` and an `images` object with every poster/backdrop/logo path.

Some values can be `null` (a tagline, a trailer, a poster for *The Shadow's Edge*), so guard before rendering.

Other image sizes: `https://image.tmdb.org/t/p/<w342|w500|w780|w1280|original><path>`

## Optional: more data

`scripts/fetch_extra.py` adds `director`, `cast` (top 10), `certification`, `budget`, `revenue`, `status`, `homepage`, `collection`, `countries`, `spoken_languages` and `studios`, then rebuilds the category files:

```
TMDB_KEY=your_key python3 scripts/fetch_extra.py
```

Run it from the repo root. `scripts/build_categories.py` rebuilds the category files from `movies.json` without needing a key.
