# Shoko WebAOM Renamer

The classic WebAOM renamer for Shoko Server, and the server's default way of
renaming and moving your files. You describe your file names with a small
script in the [WebAOM scripting language](https://wiki.anidb.net/WebAOM#Scripting),
and the renamer sorts each file into a folder for its series.

Official server builds come with the plugin installed and turned on. That
bundled copy can't be uninstalled, but a newer release installed over it can.
If you built the server yourself, install the plugin from the Shoko Core
Repository under Settings → Plugin Management → Browse.

Found a problem? Open an issue in the
[issue tracker](https://github.com/ShokoAnime/Shoko.Plugin.WebAOM/issues) or
ask on [Discord](https://discord.gg/shokoanime).

## Is it the right renamer for you?

The renamer is feature-frozen. It follows the WebAOM scripting language,
which isn't ours to extend, so it gets bug fixes but no new features. It
works well for simple file names and the two folder layouts described below.

If you want a folder structure of your own, or your files spread over
several managed folders, try the [Lua renamer](https://github.com/Mik1ll/LuaRenamer)
plugin, or write a relocation provider of your own.

## Naming files

The renamer reads your script one line at a time. A line starting with `DO`
always runs; any other line runs only when its tests pass. Each line does one
of three things:

- `ADD` appends text to the name, with tags filled in, such as `%ann` (the
  anime's main title), `%eng` (its English title), `%enr` (the episode
  number), `%epn` (the episode's English title), `%grp` (the release group),
  `%res` (the resolution) or `%CRC` (the file's CRC32).
- `REPLACE` finds and replaces text in the name built so far.
- `FAIL` stops, and the file keeps its current name.

The file always keeps its extension, and characters a path can't hold are
replaced. The scripting language is documented on the
[AniDB wiki](https://wiki.anidb.net/WebAOM#Scripting).

The renamer needs to know which episodes a file holds, so a file Shoko hasn't
recognized yet is left alone.

## Moving files

By default, a file goes to the folder that already holds the series' latest
episode, as long as that folder has room. Otherwise it goes to the first drop
destination with room, in a folder named after the anime.

With group aware sorting on, a file goes to `<group>/<series>` in the first
drop destination, or just `<series>` when the group holds only one series.
Adult series go to a destination with `Hentai` in its path when there is one,
and other series stay out of it.

## Settings

Each relocation preset using the renamer has settings of its own:

| Setting | Default | What it does |
|---|---|---|
| Max Episode Length | `33` | Episode titles longer than this (`1` to `250`) are cut short with `…`. |
| Use Group Aware Sorting | off | Sorts files by your Shoko groups, as described above. |
| Script | the sample | The WebAOM script that names your files. |

A new preset starts with a sample script, which gives names like
`[Group]_Title_-_01_(1920x1080_Blu-ray_H264)_[CRC32].mkv`. When the server
starts with no presets at all, it creates one called "Default" from it.

## For developers

```
dotnet build Shoko.Plugin.WebAOM.slnx -c Release
dotnet test Shoko.Plugin.WebAOM.slnx -c Release
```

The plugin builds against the published `Shoko.Abstractions` package. To
build against a Shoko checkout instead, which also runs the configuration
analyzer, pass `-p:ShokoRepositoryPath=/path/to/Shoko`.

The renamer is `WebAOMRenamer`, an `IRelocationProvider<WebAOMSettings>`, and
every tag it knows is in `Constants.FileRenameTag`. The thumbnail and icon in
`source/Assets/` are embedded in the assembly.

To release, publish a GitHub release with a `v<version>` tag on `stable`. The
workflow builds and tests the plugin, attaches the archive to the release and
adds the release to the manifest on the `metadata` branch.
