# Media Player controls for Lua with Mpris and DBus

Use the DBus
[Media Player Remote Interfacing Specification (Mpris)](https://specifications.freedesktop.org/mpris-spec/latest/)
to control your media player (e.g.[ VLC](https://www.videolan.org/),
[QuodLibet](https://quodlibet.readthedocs.io/)).

# Installation

## Using Luarocks

Probably, the easiest way to install this widget is to use `luarocks`:

    luarocks install media_player

You can use the `--local` option if you don't want or can't install
it system-wide

This will ensure that all its dependencies are installed.

## NixOS

If you are on NixOS, you can install this package from
[nix-stefano-m-overlays](https://github.com/stefano-m/nix-stefano-m-nix-overlays).

# Usage

---------

**NOTE**

This library leverages [GLib's GIO
GDBusProxy](https://developer.gnome.org/gio/stable/GDBusProxy.html#GDBusProxy.description)
objects via
[`dbus_proxy`](https://luarocks.org/modules/stefano-m/dbus_proxy). That means
that you **must** use the code inside a [GLib main event
loop](https://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html#glib-The-Main-Event-Loop.description)
for it to work. For example, use it with [Awesome WM](https://awesomewm.org/)
or create your own event loop.

---------

Require the `media_player` module and then create a media player interface
for each player that implements the Mpris specification.

When creating a media player with a given `NAME`, the module will
attempt to connect to a DBus destination called `org.mpris.MediaPlayer2.<NAME>`.

A media player is created with

```lua
MediaPlayer = require("media_player")
player = MediaPlayer:new(name)
```

Use the `is_connected` attribute on the player object to check whether the
corresponding application can controlled.

For example, say that we want to control `vlc`, but the application has not
been started. In this case `is_connected` will be `false` and trying to access
any attribute on the Proxy object will result in an error.

Instead, when `is_connected` is `true`, you can, for example, use the
`PlayPause`, `Stop`, `Previous` and `Next` methods to control the player.

For more detail, see also the
[dbus_proxy](https://github.com/stefano-m/lua-dbus_proxy/) documentation.

The `Position` property and `position_as_str` method return the position of the
track in microseconds and as `HH:MM:SS` respectively.

The `Metadata` property returns the current track's metadata in a table as per
the
[metadata specification](https://www.freedesktop.org/wiki/Specifications/mpris-spec/metadata/).

The `info` method returns a subset of the metadata in a table with the following
keys:

* `album`: name of the album
* `title`: title of the song
* `year`: song year
* `artists`: comma-separated list of artists (may be just one artist)
* `length`: total lenght of the track as `HH:MM:SS`

See the generated documentation for more detailed information.

## An example using the Awesome Window Manager

Require the `media_player` module in your Awesome configuration file
`~/.config/awesome/rc.lua` and then create a media player interface for each
player that implements the Mpris specification.

You can create as many players as you want and bind them to different keys.

If you want to display information about the current track, you can use
`Metadata` or `info` to extract it and then use it e.g. with
Awesome's
[`naughty.notify`](https://awesomewm.org/doc/api/modules/naughty.html#notify).

For example:

```lua
MediaPlayer = require("media_player")
quodlibet = MediaPlayer:new("quodlibet")
vlc = MediaPlayer:new("vlc")
```

Then you can bind the keys.  In this example, the basic controls are set up,
plus a notification and bindings to quit the application.

```lua
awful.util.table.join(
  -- QuodLibet bound to the media keys
  awful.key({}, "XF86AudioPlay", function () quodlibet.is_connected and quodlibet:PlayPause() end),
  awful.key({}, "XF86AudioStop", function () quodlibet.is_connected and quodlibet:Stop() end),
  awful.key({"Control"}, "XF86AudioStop", function () quodlibet.is_connected and quodlibet:Quit() end),
  awful.key({}, "XF86AudioPrev", function () quodlibet.is_connected and quodlibet:Previous() end),
  awful.key({}, "XF86AudioNext", function () quodlibet.is_connected and quodlibet:Next() end),
  -- modkey + i shows useful information from QuodLibet
  awful.key({modkey}, "i", function ()
      local info = quodlibet.is_connected and quodlibet:info() or
          {title = "quodlibet", album = "not available"}
      naughty.notify({title=info.title, text=info.album})
  end)
  -- VLC bound to modkey + media keys
  awful.key({modkey}, "XF86AudioPlay", function () vlc.is_connected and vlc:PlayPause() end),
  awful.key({modkey}, "XF86AudioStop", function () vlc.is_connected and vlc:Stop() end),
  awful.key({"Shift", "Control"}, "XF86AudioStop", function () vlc.is_connected and vlc:Quit() end),
  awful.key({modkey}, "XF86AudioPrev", function () vlc.is_connected and vlc:Previous() end),
  awful.key({modkey}, "XF86AudioNext", function () vlc.is_connected and vlc:Next() end)
)
```

Since Awesome calls the functions without passing the object as first parameter,
we must wrap the call to the object's methods in an anonymous function.

# Documentation

The documentation can be generated using [LDoc](http://stevedonovan.github.io/ldoc/).
Running `ldoc .` in the root of the repository will generate HTML documentation
in the `docs` directory.
