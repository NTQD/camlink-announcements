# CamLink announcements feed

CamLink downloads `announcements.json` from
`https://raw.githubusercontent.com/NTQD/camlink-announcements/main/announcements.json`
(`BuildConfig.ANNOUNCEMENTS_FEED_URL`) about every 12 hours while the phone has internet,
and at each app start, and adds each new entry to the Notification Center (doc/BUGLOG.md UI-170).

The feed lives in its own **public** repository because `NTQD/CamLink2` is private: a private
file would need a GitHub token inside the app, and anyone could extract it from the APK.

## Publishing an announcement

1. Edit `announcements.json` in the public repository (template: `announcements.json` here).
2. Add an entry with a new `id`. Ids are never reused: a phone that has seen an id once never
   adds it again, even after the user deletes the notice.
3. Commit to `main`. GitHub's raw file cache may delay it by about 5 minutes; phones pick it up
   at their next check (next app start, or within 12 hours).

Remove old entries whenever you like; phones keep the notices they already have.

## Format (`schema` 1)

| Field | Required | Rule |
|---|---|---|
| `id` | yes | 1–64 characters: letters, digits, `.`, `_`, `-` |
| `date` | yes | `YYYY-MM-DD`; shown as the notice's time |
| `title` | yes | object by language (`en`, `vi`, ...), at most 120 characters each |
| `body` | no | same, at most 1000 characters each |
| `expires` | no | `YYYY-MM-DD`, last day it is added (UTC) |
| `minVersionCode` / `maxVersionCode` | no | app `versionCode` range, inclusive |

- The app shows the phone's app language, else `en`, else any language given.
- Text is plain: no links or formatting. Control characters are removed.
- At most 20 entries are read and the file must stay under 64 KB; an invalid entry is skipped.
- A file with another `schema` number is ignored as a whole.

## Testing on a phone

Close the app completely and open it again: the start check runs as soon as the phone is online.

```bash
adb shell am force-stop com.example.camlink
```

Forcing the periodic job (`cmd jobscheduler run -f`) does not work: WorkManager skips periodic
work that is run before its period is up.
