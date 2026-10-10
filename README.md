# CINEVERSE Movies JSON

A JSON dataset for the **CINEVERSE** movie website. It stores movie data under one unified `Movies` object, with merged category arrays, YouTube trailer keys, TMDB image paths, keywords, and reviews.

## 1. Put the JSON file on GitHub

1. Create a **public** GitHub repository (or use an existing public repository).
2. Upload the JSON file to the repository's root and name it `movies.json`.
3. Commit your changes to the `main` branch.
4. Build the Raw URL using this format:

```text
https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPOSITORY/main/movies.json
```

Replace `YOUR_USERNAME` and `YOUR_REPOSITORY` with your GitHub username and repository name. If the file is in a folder or your branch has another name, update the URL accordingly.

> **Privacy note:** The Raw URL works without authentication for a public repository. For a private repository, do not put a GitHub token in frontend JavaScript. Use a backend/serverless function to read the file securely and return the data to your website.

## 2. Fetch the JSON with JavaScript

```js
const MOVIES_JSON_URL =
  "https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPOSITORY/main/movies.json";

async function loadMovies() {
  const response = await fetch(MOVIES_JSON_URL);

  if (!response.ok) {
    throw new Error(`Could not load movie data: ${response.status}`);
  }

  return response.json();
}

// Example usage
loadMovies()
  .then((data) => {
    console.log(data);
    console.log(data.Movies.Top_Rated);
  })
  .catch((error) => console.error(error));
```

Call `loadMovies()` once when your site starts, then reuse the returned `data` rather than fetching the whole file repeatedly.

## 3. JSON structure

All category arrays are grouped under one `Movies` object. The former `cartoon` and `general` collections have been merged.

```text
movies.json
└── Movies
    ├── Top_Rated
    ├── Action
    ├── Adventure
    ├── Animation
    ├── Comedy
    ├── Crime
    ├── Documentary
    ├── Drama
    ├── Family
    ├── Fantasy
    └── Science Fiction
```

Each category is an **array of movie objects**. A movie can appear in more than one category, so repeated movie IDs across different categories are expected. The current dataset contains 192 category entries across 11 categories, representing 168 unique movie IDs.

## 4. Access movies by category

```js
// Top-rated movies
const topRatedMovies = data.Movies.Top_Rated;

// Action movies
const actionMovies = data.Movies.Action;

// Category names containing spaces use bracket notation
const scienceFictionMovies = data.Movies["Science Fiction"];

// Render a category
 topRatedMovies.forEach((movie) => {
  console.log(movie.title, movie.vote_average);
});
```

To create one list of unique movies from all categories, deduplicate by TMDB movie ID:

```js
function getUniqueMovies(moviesByCategory) {
  const allMovies = Object.values(moviesByCategory).flat();
  return [...new Map(allMovies.map((movie) => [movie.id, movie])).values()];
}

const allMovies = getUniqueMovies(data.Movies);
console.log(allMovies);
```

## 5. Useful movie fields

| Field | Description |
| --- | --- |
| `id` | TMDB movie ID |
| `title` | Display title |
| `original_title` | Original movie title |
| `overview` | Movie synopsis |
| `release_date` | Release date (`YYYY-MM-DD`) |
| `vote_average` | TMDB average score |
| `vote_count` | Number of TMDB votes |
| `genres` | Array of genre objects with `id` and `name` |
| `runtime` | Runtime in minutes, when available |
| `tagline` | Movie tagline, when available |
| `video` | YouTube video key, or `null` when no key was available |
| `images` | Arrays of TMDB image paths: `backdrops`, `logos`, and `posters` |
| `keywords.keywords` | Array of keyword objects with `id` and `name` |
| `reviews` | Array of review objects |

Some values may be empty or `null`, so guard against missing data before displaying them.

## 6. Display reviews

`reviews` is already an array. Each review can contain fields such as `author`, `content`, and `created_at`.

```js
function renderReviews(movie) {
  movie.reviews.forEach((review) => {
    console.log(`${review.author}: ${review.content}`);
    console.log(review.created_at);
  });
}

const movie = data.Movies.Top_Rated[0];
renderReviews(movie);
```

## 7. Build trailer links

The `video` property stores the YouTube key, not a complete URL. For an embed:

```js
function getTrailerEmbedUrl(movie) {
  return movie.video
    ? `https://www.youtube.com/embed/${movie.video}`
    : null;
}

const movie = data.Movies.Top_Rated[0];
const trailerUrl = getTrailerEmbedUrl(movie);

if (trailerUrl) {
  console.log(trailerUrl);
}
```

Example HTML once you have a valid trailer URL:

```html
<iframe
  src="https://www.youtube.com/embed/VIDEO_KEY"
  title="Movie trailer"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  allowfullscreen
></iframe>
```

Replace `VIDEO_KEY` with `movie.video`. Only create the iframe when `movie.video` is not `null` or empty.

## 8. Display TMDB images

The JSON stores TMDB **file paths**, not complete image URLs. Add the desired TMDB image size before the path.

```js
const TMDB_IMAGE_BASE = "https://image.tmdb.org/t/p/";

function getImageUrl(filePath, size = "w500") {
  return filePath ? `${TMDB_IMAGE_BASE}${size}${filePath}` : null;
}

const movie = data.Movies.Top_Rated[0];
const posterPath = movie.images?.posters?.[0]?.file_path;
const posterUrl = getImageUrl(posterPath, "w500");

if (posterUrl) {
  console.log(posterUrl);
}
```

Common size choices include `w342` or `w500` for posters and `w780` or `w1280` for backdrops. Check that the image array contains an item before using it.

## 9. Filter and search

```js
// Find a movie by its TMDB ID
const foundMovie = allMovies.find((movie) => movie.id === 166428);

// Search by title (case-insensitive)
function searchMovies(movies, query) {
  const normalizedQuery = query.trim().toLowerCase();
  return movies.filter((movie) =>
    movie.title.toLowerCase().includes(normalizedQuery)
  );
}

const results = searchMovies(allMovies, "batman");
```

## Notes

- The examples use **vanilla JavaScript** and do not require a framework.
- Use `try/catch` or `.catch()` to handle loading failures.
- A public JSON file can be read by anyone who knows its URL; do not store secrets or private user data in it.
- The file contains movie metadata and TMDB image paths. Image paths are not themselves complete URLs.
