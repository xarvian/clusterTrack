# Coerce an object to clusterTrack data format

Converts an object with spatial coordinates and a timestamp column to
the `data.table` format with an `sf` geometry column used by
clusterTrack.

## Usage

``` r
# S3 method for class 'data.frame'
as_ctdf(
  x,
  coords = c("longitude", "latitude"),
  time = "time",
  s_srs = 4326,
  t_srs = "+proj=eqearth",
  ...
)
```

## Arguments

- x:

  A `data.frame` object.

- coords:

  Character vector of length 2 specifying the coordinate column names.
  Defaults to `c("longitude", "latitude")`.

- time:

  Name of the POSIXt time column. Will be renamed to `"timestamp"`
  internally.

- s_srs:

  Source spatial reference. Default is EPSG:4326

- t_srs:

  Target spatial reference passed to
  [`sf::st_transform()`](https://r-spatial.github.io/sf/reference/st_transform.html).
  Default is "+proj=eqearth".

- ...:

  Currently unused

## Value

An object of class `ctdf` (inherits from `data.table` and `data.frame`),
with an `sfc_POINT` geometry column named `location`.

## Details

Rows are sorted by timestamp, geometry is transformed to `t_srs`, and
clusterTrack columns are initialized. Existing reserved columns are
overwritten with a warning.

The converted object is checked for duplicate locations with the same
timestamp and temporal gaps exceeding
`getOption("clusterTrack.max_gap", 24)` hours. Duplicate warnings
identify row positions in the returned, timestamp-sorted `ctdf`.

## ctdf columns

In addition to columns retained from the input, a `ctdf` contains the
following standardized columns:

- `timestamp`: Observation time. Rows are ordered by this column when
  the `ctdf` is created.

- `location`: Point geometry in the target coordinate reference system.

- `.id`: Sequential row identifier assigned after ordering by
  `timestamp`. It is used internally to track locations through the
  clustering workflow and does not refer to the row number in the
  original input.

- `cluster`: Final cluster assignment. It is initialized to `NA`.
  [`cluster_track()`](https://xarvian.github.io/clusterTrack/reference/cluster_track.md)
  assigns positive integers to clustered locations and `0` to unassigned
  locations.
  [`aggregate_ctdf()`](https://xarvian.github.io/clusterTrack/reference/aggregate_ctdf.md)
  may subsequently merge and renumber clusters.

- `lof`: Local Outlier Factor score for each location. It is initialized
  to `NA` and populated by
  [`ctdf_lof()`](https://xarvian.github.io/clusterTrack/reference/ctdf_lof.md)
  for locations with `cluster > 0`. Unassigned locations retain `NA`.

## See also

[`as_ctdf()`](https://xarvian.github.io/clusterTrack/reference/as_ctdf.md),
[`as_ctdf.sf()`](https://xarvian.github.io/clusterTrack/reference/as_ctdf.sf.md)

## Examples

``` r
data(mini_ruff)
x = as_ctdf(mini_ruff)
plot(x)

```
