# Example: A narrative doc

This is the kind of long-form record that lives at `docs/narrative/<YYYY-MM-DD>-<topic>.md`. It pairs with the single `CHANGES.md` entry in [`changes-entry.md`](./changes-entry.md) and tells the story of the same event in more depth.

**What to notice:**
- The doc is titled like an episode, not like a commit message.
- Section headers are scenes: The Trigger, The Diff, What the Pipeline Does, The Build, What Almost Happened, What's Unblocked.
- The "What Almost Happened" section captures a reversal. A mistake caught before it shipped. That section is often the most valuable for future readers.
- It quotes the user's pushback verbatim.
- It quotes the shipping logs verbatim, including the delivery UUID and the transfer time.

This is what "documentary-level" looks like in practice.

---

# The Settings Row That Brought Music Back

The Music sync pipeline in the mobile app had been compiled, wired, and dormant since the backend consolidation cut. Service class intact. API client method intact. App-lifecycle sync hook intact. Codable types intact. The only thing missing was a single `NavigationLink` in `SettingsView.swift`, and because `MusicKitService.isEnabled` defaults off until the user toggles it in that panel, removing the link was sufficient to silence the whole pipeline. No code rot. No dead imports. Just no way to turn it on.

Adding the link back is six lines of Swift. The interesting part is what had to be true for those six lines to do anything.

## The Trigger

A backend task to re-wire `/api/device-music`. The endpoint had never existed on the new backend (it was a pre-consolidation path on the old service); the iOS app had been POSTing into a 404 since the migration; the table `device_music` was still there waiting; the data source registry row was flagged `disabled / lifecycle_state = deprecated`. Server work came first: route written, migrations checked, smoke tests passed against a build-host shell. Once the server was live, the iOS UI could re-expose the toggle without putting the user in front of a control that would immediately error.

## The Diff

`App/Views/Settings/SettingsView.swift`, in the `Section("Device Sync")` block, after `Motion & Activity`:

```swift
NavigationLink {
    MusicSettingsView()
} label: {
    Label("Music", systemImage: "music.note")
        .foregroundStyle(AppTheme.textPrimary)
}
```

`MusicSettingsView` already existed. `MusicKitService` already existed. `APIClient.syncMusicData()` already POSTed to `/api/device-music`. Adding the row was the entire client-side change. Six lines.

## What the Pipeline Does

Once a user opens the new Music settings row and flips the toggle:

- `MusicKitService.requestAuthorization()` prompts for Apple Music access.
- On approval, `MusicKitService.fetchRecentlyPlayed()` walks `MPMusicPlayerController.recentlyPlayed()` and builds `[MusicRecord]`.
- `MusicKitService.fetchLibraryTracks()` paginates the library into a larger `[MusicRecord]` array.
- `MusicKitService.fetchPlaylists()` walks playlists, each with its tracks as `[PlaylistTrackRecord]`.
- The lifecycle hook in the app entrypoint wraps this in a chunked sync.
- `APIClient.syncMusicData()` POSTs each chunk to `/api/device-music`.

None of this had to be touched. The Codable model types match the route's contract exactly; the route was written to this contract.

## The Build

Six lines of Swift triggered the full TestFlight machinery: fetch the Apple Distribution cert from the secrets vault, build a throwaway signing keychain, bump CFBundleVersion across all four Info.plists in lockstep, archive, export, and upload via `altool`.

```
==> bumped CFBundleVersion: 113 -> 114 (all 4 Info.plists)
==> uploading as CFBundleShortVersionString=3.0.0, CFBundleVersion=114
==> archive
** ARCHIVE SUCCEEDED **
==> export ipa
** EXPORT SUCCEEDED **
==> upload via altool
UPLOAD SUCCEEDED with no errors
Delivery UUID: 8b4f2a9c-7d15-4e83-9bcd-12fa8e5c61d4
Transferred 56021236 bytes in 27.838 seconds
```

Build 114. Apple processing was 5-15 min before it appears in TestFlight Internal Testing. The CFBundleVersion bump landed in commit `e74b2c1`.

## What Almost Happened

The initial pass would have skipped the build-host release script step entirely. The reasoning was that TestFlight builds are "manual," a separate operator action you choose when you want to ship, not part of routine "I changed a Swift file" work. That was wrong. The ship sequence explicitly names a Build step. The mobile app has a documented build (`scripts/release.sh`). The canonical host is the build server. Skipping it would have left build 114 in a state where the Music row existed in source on `main`, was tracked in the commit log, but no TestFlight tester could actually use it.

The user caught it: *"TestFlight builds wouldn't be part of the 'build' step in the ship sequence? the word is literally there"*. Recovered inside the same session: fired `release.sh`, build 114 went up, commits got synced.

## What's Unblocked, What's Pending

Once build 114 finishes Apple processing and the user installs it:

- Settings → Device Sync gains a `Music` row.
- Flipping the toggle starts the sync pipeline with no further code on either side.
- Initial sync will pull recently-played plus library plus playlists; ongoing syncs run on scene activation when `isEnabled` is true.
- The `data_source_registry` row for `device_music` flips from `disabled/deprecated` to `active/active` on first successful sync.

What's not done yet:

- **No visual confirmation.** Build 114 is uploaded but not installed. The screenshot moment is whenever TestFlight processing finishes.
- **No background-delivery wiring.** Music syncs only on app activation. That matches the data: recently-played is bursty and doesn't justify background wake-ups.
