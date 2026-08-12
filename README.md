# homebrew-march

Homebrew tap for [march](https://github.com/melvinsh/march) — unattended Arch
Linux ARM VMs on QEMU with a hardware-accelerated Hyprland desktop.

```sh
brew trust melvinsh/march
brew install melvinsh/march/march
```

That pulls the whole stack: march itself, plus `qemu-march`, a QEMU built with
both virgl GPU acceleration and user-mode networking.

`brew trust` comes first because Homebrew refuses to load formulae from
untrusted third-party taps, and `march` depends on `qemu-march` from this same
tap — so installing without it stops partway with *"Refusing to load formula
melvinsh/march/qemu-march from untrusted tap"*. Trusting the tap also taps it,
so there is no separate `brew tap` step.

## Where these come from

`Formula/` in this repository is **generated**. Every file in it is a copy of
[`Formula/` in melvinsh/march](https://github.com/melvinsh/march/tree/main/Formula),
written by `docs/publish-formulae.sh` there, which refuses to finish unless the
two directories are identical. Edit the formulae in march and publish from
there; an edit made here is lost at the next release.

## Formulae

| Formula | What it is |
| --- | --- |
| `march` | The TUI. Depends on everything below. |
| `qemu-march` | QEMU 11.0.3 with virgl **and** libslirp, keg-only so it sits alongside your existing `qemu`. |

Neither of the two QEMU builds you can otherwise get works for march:
Homebrew's `qemu` has networking but no OpenGL, so guests render on the CPU;
the community virgl taps have OpenGL but omit `libslirp`, so guests have no
network — which march's installer depends on entirely. Both also take the name
`qemu`, so installing one removes the other. `qemu-march` combines the two and
is keg-only, leaving your stock `qemu` linked and working.

## Tap trust

Trusting the whole tap is what you want here, because `march` and
`qemu-march` are separate formulae and both must be loadable:

```sh
brew trust melvinsh/march
```

Trusting only `--formula melvinsh/march/march` is not enough — the install
will fail when it reaches `qemu-march`.

The GPU stack comes from the `startergo` taps (ANGLE, libepoxy,
virglrenderer), which Homebrew taps automatically as dependencies. If it
reports those as untrusted as well:

```sh
brew trust startergo/angle startergo/libepoxy startergo/virglrenderer
```

## Patches

`Formula/patches/` carries out-of-tree work that upstream has not merged:

- `qemu-11.0.3-macos-gl.patch` — teaches QEMU's Cocoa display to render
  through OpenGL, which is what makes `virtio-gpu-gl` usable on macOS.
- `virglrenderer-macos-venus-transport.patch` — three fixes that let
  virglrenderer's Venus render server start on macOS. Not applied by these
  formulae; Venus remains unusable on Apple Silicon for a separate reason
  documented in the march README.

Provenance for each is in `Formula/patches/NOTICE` and `NOTICE-venus`.
