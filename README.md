# homebrew-march

Homebrew tap for [march](https://github.com/melvinsh/march) — unattended Arch
Linux ARM VMs on QEMU with a hardware-accelerated Hyprland desktop.

```sh
brew install melvinsh/march/march
```

That single command taps this repository and pulls the whole stack: march
itself, plus `qemu-march`, a QEMU built with both virgl GPU acceleration and
user-mode networking.

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

Homebrew requires third-party taps to be trusted before it will run their
formulae. If `brew install` reports that formulae are being ignored:

```sh
brew trust --formula melvinsh/march/march
brew trust --formula melvinsh/march/qemu-march
```

The GPU stack comes from the `startergo` taps (ANGLE, libepoxy,
virglrenderer), which Homebrew taps automatically as dependencies and which
may need trusting the same way.

## Patches

`Formula/patches/` carries out-of-tree work that upstream has not merged:

- `qemu-11.0.3-macos-gl.patch` — teaches QEMU's Cocoa display to render
  through OpenGL, which is what makes `virtio-gpu-gl` usable on macOS.
- `virglrenderer-macos-venus-transport.patch` — three fixes that let
  virglrenderer's Venus render server start on macOS. Not applied by these
  formulae; Venus remains unusable on Apple Silicon for a separate reason
  documented in the march README.

Provenance for each is in `Formula/patches/NOTICE` and `NOTICE-venus`.
