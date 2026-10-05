# Changes this fork carries

This branch is Google's Play Games plugin for Unity
(`https://github.com/playgameservices/play-games-plugin-for-unity`, called upstream here) at upstream's tag
`v2.3.0`, which Ateo builds and publishes as `com.google.play.games` on its registry `https://upm.ateo.dev`.
Every change the branch carries against that tag is listed below, one entry each: what it does and why,
where it lives and its commit, its upstream issue or pull request, and when it is dropped. A change not
listed here is not carried, apart from this file itself.

Every move to a newer upstream release walks this list. An entry is kept when the new release still needs
it, and dropped when upstream fixed the same thing in its own way and that fix is good. The list is then
brought in line with the rebased branch. How an update is taken, and the rule for when this fork may change
upstream's code at all, are on the handoff page of the package that pins this build, `ch.ateo.mobile`, in
`Documentation~/gpgs-fork-handoff.md`.

Paths are relative to the package folder, `Assets/Public/GooglePlayGames/com.google.play.games`. A commit is
named by its subject, with the hash it has on this branch.

## The changes

1. **A saved game is not committed after its write to the device failed.** In `CommitUpdate` and in the
   conflict resolver's `ResolveConflict`, a `writeBytes` that returned false reported `BadInputError` and
   then went on to `commitAndClose` or `resolveConflict` anyway. `writeBytes` returns false only for open
   contents whose write to disk failed partway, so the commit uploaded whatever was on disk under the new
   metadata, and the callback reported a second result. Both sites now return after the error callback, and
   the log says the write to the device failed. The status stays `BadInputError`, so callers see the same
   first result as before. It lives in `Runtime/Scripts/Platforms/Android/AndroidSavedGameClient.cs`,
   marked `ATEO PATCH (failed write)` at both sites. The commit is "Stop a saved game commit after its write
   to the device failed", `7f1c234b`. Upstream, the same change is a pull request to be opened from the
   branch `fix/saved-game-commit-after-failed-write`, and
   [issue 3156](https://github.com/playgameservices/play-games-plugin-for-unity/issues/3156) reads like this
   failure. It is dropped when the fork moves to an upstream release that carries that pull request, or a fix
   of upstream's own that stops both sites after a failed write.
2. **The generated Android library gets its `project.properties`.** The plugin's Android setup writes its
   manifest to `Assets/Plugins/Android/GooglePlayGamesManifest.androidlib/AndroidManifest.xml` in the game's
   project. Upstream's `.unitypackage` brings the rest of that folder, `project.properties` among it, but a
   package installed from a registry cannot place files under the game's `Assets`, and upstream's generator
   writes only the manifest. Without `project.properties` the folder is not a valid Android library project
   and the manifest is not merged into the build. `GenerateAndroidManifest` now writes it beside the manifest
   when it is missing. It lives in `Editor/GPGSUtil.cs`, marked `ATEO PATCH (registry install)`. The commit
   is "Generate project.properties alongside the androidlib AndroidManifest", `f4a02a7c`. Upstream, no issue
   or pull request names it. It is dropped when upstream's generator writes `project.properties` itself, or a
   registry install is shown to build with the manifest merged without it.
3. **Two folder metas that have no folder are removed.** `Editor/m2repository.meta` and
   `Runtime/Scripts/Plugins.meta` describe folders that upstream's `.unitypackage` creates empty. Git keeps
   no empty folder, so a package packed from this repository would ship each meta with nothing beside it.
   The commit is "Remove the orphan folder metas the package ships without their folders", `d85e7d0e`.
   Upstream, no issue or pull request names it. It is dropped when upstream's package folder no longer
   carries either meta without its folder.
4. **The version names the Ateo build.** `package.json` reads `2.3.0-ateo.1`: the Ateo build of upstream
   2.3.0, which games pin exactly. A fix on the same upstream base raises the last number, and a move to a
   newer upstream release starts again at `-ateo.1`. The commit is "Version 2.3.0-ateo.1", `eb080c06`.
   Upstream has no counterpart. It is dropped only when Ateo stops building its own package of the plugin.
