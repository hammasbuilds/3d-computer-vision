# Numbers in the README that are deliberately NOT results.
#
# 1.25 is the NAME of the threshold metric, not a measured value: delta<1.25,
# delta<1.25^2, delta<1.25^3 are the standard accuracy-under-threshold metrics. The
# measured quantities are the percentages beside them, and those are checked.
1.25
#
# These two are from the write-up of two EARLIER broken versions of this notebook, kept
# because the bugs are the point of that section. 102,684 was the file count wrongly
# classified as depth maps when the path test matched the mount path; 95,534 was the
# absurd abs_rel the unbounded 1/disparity produced. Neither is a result of the run that
# results.json records, so neither is in it.
102684
95534
