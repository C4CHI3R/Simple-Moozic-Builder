# Changelog

## Unreleased

Fixes for the window freezing ("Not Responding") when building very large packs.

### Fixed

- **UI freeze on big builds.** Converting and building used to run on the Tk UI
  thread, so the window couldn't repaint or take input until the whole pack was
  done. Both now run on a background worker thread that reports back through a
  queue; the window stays responsive the whole time.
- **Build can be cancelled.** While a convert/build is running the Build button
  turns into **Cancel**. Closing the window mid-job also stops it cleanly.
- **Half-written `.ogg` files.** Conversions now encode into `<name>.ogg.part`
  and only rename it into place once the encode finishes. Killing the app (or a
  crash) mid-song no longer leaves a truncated `.ogg` that the next run treated as
  "up to date". Applies to the soundfile and ffmpeg paths, plain conversion, and
  mixtape creation.
- **Old truncated files get repaired.** A converted `.ogg` is now only trusted if
  its final Ogg page carries the end-of-stream flag. Files cut short by an earlier
  interrupted run show as `stale - incomplete, needs re-convert` and are redone
  on the next convert/build.
- **One bad file no longer aborts everything.** A song that fails to convert is
  collected and reported together at the end ("Build the pack without them?")
  instead of stopping the whole batch. Previously a conversion error raised
  `SystemExit`, which slipped past the build's `except Exception` and killed the
  build; those songs are now skipped.
- **Completion counts** (cassettes / vinyls) only include songs that actually
  made it into the pack.

### Performance

- Conversion progress no longer rescans every row (with a filesystem `resolve()`
  each) for every converted song; rows are indexed by source file once up front.
- The song list is repainted at most about once a second during a conversion
  instead of fully rebuilt after every song.
- Resampling non-44.1 kHz audio is done in bounded chunks. The old version built
  full-length float arrays, which spiked memory on long tracks.
- ffmpeg no longer pops a console window for every file in a large batch (Windows).

### Notes

- "Convert Selected" (right-click menu) and the refresh button are disabled while a
  convert/build job is running, so the song list can't change underneath it.
