# Netflix Data Cleaning & Analysis (SQL Server)

This is a ETL  project I built to practice cleaning and analyzing messy real-world data using SQL Server. I took the Netflix Movies and TV Shows dataset from Kaggle and worked through the whole process of turning it from a raw, messy CSV into clean, structured tables that I could actually run analysis on.
I extracted the data and loaded it into SQL Server by using SQL Alchemy and then Transformed and Analyzed it using SQL inside SQL Server Database.

## Why I built this

The raw dataset looks fine at first glance, but once I started querying it I realized columns like `director`, `cast`, `country`, and `listed_in` all have multiple values crammed into one field, separated by commas. That's a pain to filter or group by, so a big part of this project was just about normalizing the data properly before doing any real analysis.

## What I used

- Microsoft SQL Server (T-SQL)
- Netflix Titles dataset from Kaggle (~8,800 rows)

## The dataset

Basic columns: `show_id`, `type` (Movie/TV Show), `title`, `director`, `cast`, `country`, `date_added`, `release_year`, `rating`, `duration`, `listed_in` (genre), `description`.

## Cleaning steps

**1. Removed bad rows**
There were a handful of rows where the data was clearly misaligned (columns shifted, weird parsing issues from the original CSV). I found the `show_id`s for these and just deleted them out of the raw table.

**2. Removed duplicates**
Some shows show up more than once under different `show_id`s. I caught these by grouping on uppercased `title` + `type`, then used `ROW_NUMBER()` partitioned by `(title, type)` to keep only one row per show.

**3. Fixed data types / weird columns**
- Converted `date_added` from text to an actual `date` type
- Noticed some rows had `duration` and `rating` swapped, so I added a fix for that — if `duration` was null, pull the value from `rating` instead

After all that, I loaded the cleaned data into a new table called `netflix`.

## Splitting out the multi-value columns

Since `director`, `cast`, `country`, and `listed_in` can each hold several comma-separated values, I split each one into its own table using `STRING_SPLIT()` + `TRIM()`, all linked back by `show_id`:

- `netflix_director` — one row per show/director
- `netflix_cast` — one row per show/cast member
- `netflix_country` — one row per show/country
- `netflix_genre` — one row per show/genre

**Filling in missing countries:** A lot of rows just don't have a country listed. So for shows where the director had a country listed on some other title, I used that to fill in the gap. Not perfect, but it recovers a decent chunk of missing data instead of just leaving it blank.

## Final tables

```
netflix            -> show_id, type, title, date_added, release_year, rating, duration, description
netflix_director   -> show_id, director
netflix_cast        -> show_id, cast
netflix_country     -> show_id, country
netflix_genre       -> show_id, genre
```

## Analysis I ran

1. **Directors who made both movies and TV shows** — counted each separately per director
2. **Which country has the most comedy movies** — joined genre + country + type
3. **Top director by year added** — using a CTE + `ROW_NUMBER()` to rank directors per year
4. **Average movie duration by genre** — had to strip " min" out of the duration text and cast it to int
5. **Directors who've made both horror and comedy movies** — grouped and filtered with `HAVING COUNT(DISTINCT genre) = 2`



