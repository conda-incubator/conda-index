# conda-index
conda index, formerly part of conda-build. Create `repodata.json` for
collections of conda packages.

## Summary of changes from the previous `conda-build index` version

* Approximately 2.2x faster conda package extraction, by extracting just the
  metadata to streams instead of extracting packages to a temporary directory.

* On the first run, `conda_index` will convert the filesystem-based
  `<subdir>/.cache` folders to a single `<subdir>/.cache/cache.db` sqlite3
  database. `sqlite3` must be compiled with the JSON1 extension. JSON1 is built
  into SQLite by default as of SQLite version 3.38.0 (2022-02-22). If the
  conversion is interrupted, `conda_index` will have to re-extract missing
  packages into the new cache; delete cache.db to re-convert.

* Uses new code to get metadata from packages without using temporary files;
  closes the package early if all metadata has been found. This is about twice
  as fast.

* Uses a sqlite metadata cache that is orders of magnitude faster than the old
  many-tiny-files cache.

* The first time `conda index` runs, it will convert the existing file-based
  `.cache` to a sqlite3 database `.cache/cache.db`. This takes about ten minutes
  per subdir for conda-forge. (If this is interrupted, delete `cache.db` to
  start over, or packages will be re-extracted into the cache.)

* No longer cache `paths.json` (only used to create `post_install.json` and not
  referenced later in the indexing process). Saves 90% disk space in `.cache`.

* Each subdir `osx-64`, `linux-64` etc. has its own `cache.db`; conda-forge’s
  1.2T osx-64 subdir has a single 2.4GB `cache.db`. Storing the cache in fewer
  files saves time since there is a per-file wait to open each of the
  many tiny `.json` files in old-style `.cache/`.

* `cache.db` is highly compressible, like the text metadata. 2.4G → zstd → 88M

* Updated Python and dependency requirements.

* Format with `black`