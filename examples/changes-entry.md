# Example: A `CHANGES.md` entry

This is what one entry in a working `CHANGES.md` looks like. The full file has dozens of dated entries, newest on top. This one is about re-enabling a dormant iOS Music sync feature after a backend route was restored.

**What to notice:**
- The opening sentence names the specific dormant state (compiled-but-dark since a specific date) and lists the files that were already in place.
- The reason the feature was dark is given (`isEnabled` defaults to false). That detail explains the architecture, not just the diff.
- The fix is quantified ("six lines of Swift") and shown verbatim.
- The build artifact is named (build 114, delivery UUID, file size, transfer time).
- A link to the narrative doc closes the entry.

The entry is one paragraph plus a code block plus four metadata lines. It is not long. It is dense.

---

## 2026-05-16

### Music sync resurrection. Settings link plus TestFlight build 114

The Music sync pipeline has been compiled-but-dark since the 2026-04-26 backend consolidation. `MusicKitService.swift`, the `APIClient.syncMusicData()` method, the lifecycle hooks in the app entrypoint that call `musicService.sync()` on scene activation, and the Codable types matching the server contract: all intact. The only thing that came out in the consolidation was the `Music` row in `Settings → Device Sync`, and because `MusicKitService.isEnabled` defaults to `false` until the user toggles it from that panel, removing the link silenced the whole pipeline. Six lines of Swift restored it.

```swift
NavigationLink {
    MusicSettingsView()
} label: {
    Label("Music", systemImage: "music.note")
        .foregroundStyle(AppTheme.textPrimary)
}
```

- **Build:** 114 (delivery UUID `8b4f2a9c-7d15-4e83-9bcd-12fa8e5c61d4`, 56 MB in 27.8s)
- **Commit:** `e74b2c1`
- **Cross-repo:** Backend route added in companion repo, same day
- **Full story:** `docs/narrative/2026-05-16-music-resurrection.md`

**What's unblocked:** Once build 114 finishes Apple processing and installs, Settings → Device Sync gains a Music row. Toggling it starts MusicKit sync; recently-played, library, and playlists start landing in the backing tables.
