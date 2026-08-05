Check for translation key drift in `packages/i18n/src/locales`.

Treat the English locale as the source of truth.

Report:

1. Keys present in English but missing from other locales.
2. Keys present in other locales but absent from English, which are likely stale.
3. Keys whose placeholders or interpolation tags differ between English and a
   translation, since these break at runtime.
4. Entries that appear untranslated — a non-English locale whose value is byte
   identical to English, excluding proper nouns and product names.

Present the result as a table of locale, key, and problem type. Summarize the
per-locale totals first so the scale of drift is clear before the detail.

Do not edit any locale files. Report only.
