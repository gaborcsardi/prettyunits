
# dev

* `pretty_dt()`, `pretty_ms()` and `pretty_sec()` now handle `NA` values
  properly, and return `NA_character_` for them (#10, @petermeissner).
* `pretty_round()` and `pretty_signif()` preserve the requested number of digits
  as character strings (#14, @billdenney).
* Add `pretty_p_value()` to convert p-values to character strings where small
  values are shown like "<0.0001" (#13, @billdenney)

* `pretty_bytes()` now formats quantities just below the units better.
  E.g. 1MB - 1B is formatted as `"1 MB"` instead of `""1000 kB"` (#18).

# 1.0.2

* `pretty_bytes()` always uses two fraction digits for non-integers.
  This looks nicer in a progress bar, as the width of string does not
  change so much.

# 1.0.1

First version with a NEWS file.

* Get rid of `R CMD check` notes.

# 1.0.0

Last version without a NEWS file.
