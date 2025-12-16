# PebbleTimely ISO Week Bug Fix Notes

## Problem
The watchface shows week 50 instead of week 51 (current date: Dec 16, 2025 = ISO week 51).

## Root Cause
The `update_week_text()` function in `src/Timely.c` (lines 750-762) uses `strftime()` with `%V` format specifier:

```c
void update_week_text(TextLayer *which_layer) {
  static char week_text[] = "W00";
  char week_format[] = "W%V"; // V = ISO 8601 week number (00-53)
  if (settings.week_format == 1) {
    week_format[2] = 'U';
  } else if (settings.week_format == 2) {
    week_format[2] = 'W';
  }
  strftime(week_text, sizeof(week_text), week_format, currentTime);
  text_layer_set_text(which_layer, week_text);
}
```

The Pebble SDK uses newlib's `strftime()` implementation, which may have bugs with the `%V` (ISO 8601 week) specifier.

## Solution
Replace the `strftime()` call with a custom ISO 8601 week calculation function based on the algorithm from StackOverflow (https://stackoverflow.com/questions/42568215/iso-8601-week-number-in-c).

The algorithm:
1. Find the day of week (Monday = 0, Sunday = 6)
2. Offset to Monday of current week
3. Add 3 days to get to Thursday
4. The week number is (day_of_year / 7) + 1

This works because ISO 8601 defines week 1 as the week containing the first Thursday of the year.
