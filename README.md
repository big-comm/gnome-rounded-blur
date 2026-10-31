# gnome-rounded-blur

BigCommunity package of [gnome-rounded-blur](https://github.com/kancko/gnome-rounded-blur),
a small native library that adds a blur effect with rounded corners to GNOME Shell.

## Why it is needed

GNOME Shell's own `Shell.BlurEffect` blurs a plain rectangle. Shell surfaces
with rounded corners (panel menus, the calendar, notifications, the dock and the
Big Clipboard panel) then show square blurred tips outside their rounded edges.

This library provides `Blur.BlurEffect`, which applies the rounded mask inside
the blur pass itself. A separate mask effect cannot do this: masking inside the
blur is blurred back into a rectangle, and an offscreen mask outside it stops
the effect from sampling what is behind the surface.

It is used by:

- **Frosted Glass** (`frosted-glass@communitybig.org`, part of Big Gnome Center):
  rounded live blur on all Shell surfaces.
- **Big Clipboard** (`big-clipboard@communitybig.org`): the optional blurred panel
  background. Without this library that option has no effect.

## Behavior and safety

- The library is optional at runtime. Before loading it, each extension checks
  it in a separate `gjs` process with every symbol resolved (`LD_BIND_NOW=1`).
  If the check fails or times out, the extension falls back to `Shell.BlurEffect`
  and GNOME Shell keeps running.
- GNOME Shell loads the library once per session. Log out and back in after
  installing or upgrading it.
- It requires GNOME 51 or later. On GNOME 50 the package is installed together
  with the GNOME 51 upgrade by a full system update (`pacman -Syu`).

## Package contents

| Path | Purpose |
|---|---|
| `/usr/lib/libblur-effect-1.0.so*` | Native library |
| `/usr/lib/girepository-1.0/Blur-1.0.typelib` | GObject introspection data (`gi://Blur`) |
| `/usr/share/gir-1.0/Blur-1.0.gir` | Introspection source |
| `/usr/include/blur-effect-1.0/` and `/usr/lib/pkgconfig/blur-effect-1.0.pc` | Development files |

## Building

The PKGBUILD builds a pinned upstream commit with Meson against the installed
mutter, then checks that the library loads and creates a rounded effect:

```sh
cd pkgbuild
makepkg -si
```

## Maintenance

The library links against mutter's versioned libraries (`libmutter-clutter-51`,
`libmutter-cogl-51`). **Rebuild this package for every new GNOME major release.**
Until it is rebuilt, the extensions detect the mismatch and use rectangular blur.

To update upstream, change `_upstream_commit` in `pkgbuild/PKGBUILD` to a reviewed
commit and confirm that `check()` passes against the target mutter.

## License

The packaged library is licensed under GPL-3.0-or-later by its upstream authors.
The packaging files in this repository are licensed under the MIT License.
