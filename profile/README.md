<p align="center"><img src="https://raw.githubusercontent.com/go-quake1/brand/main/social/go-quake1.png" alt="go-quake1" width="720"></p>

# go-quake1

**Pure-Go Quake (id Tech 1, 1996) — bare-metal, no cgo.**

`go-quake1` is the pure-Go port of the [Quake](https://en.wikipedia.org/wiki/Quake_(video_game))
engine — original 1996 NetQuake codebase, hand-ported from
[tyrquake](https://disenchant.net/tyrquake/) (commit
`6531579`) — targeting the [TamaGo](https://github.com/usbarmory/tamago)
bare-metal Go runtime via the
[cloud-boot](https://github.com/cloud-boot) UEFI loader. `CGO_ENABLED=0`
across the org, no syscalls, no SDL, no GL — the engine drives the
framebuffer through the [go-virtio](https://github.com/go-virtio) GPU
driver and the [go-asmgen](https://github.com/go-asmgen) Plan 9 SIMD
generator on the six 64-bit Go targets.

Sibling orgs cover the rest of the id Tech engine series:
[**go-quake2**](https://github.com/go-quake2) (id Tech 2, 1997) and
[**go-quake3**](https://github.com/go-quake3) (id Tech 3, 1999).

## Repositories

| Repo | Latest | Role |
|---|---|---|
| [`engine`](https://github.com/go-quake1/engine) | — | The Quake engine — common + server + client + soft renderer + sound, in pure Go |
| [`brand`](https://github.com/go-quake1/brand) | — | Logos, social previews, favicons |
| [`.github`](https://github.com/go-quake1/.github) | — | This profile + shared org workflows |

Planned siblings: `docs` (mkdocs+mike spec-traceability + porting notes),
`go-quake1.github.io` (Hugo family landing).

## How it works

The engine is split into the same layers as the upstream:

- **common** — `mathlib`, `crc`, `cmd`, `cvar`, `zone`, `qstr`,
  `sizebuf`, `qpath`, `qparse`, `msg`, `qargs`, `pak`, `vfs`, `wad`,
  `keys`, `protocol`, `anorms`
- **models** — `bspfile`, `bsptrace`, `mdl`, `spr`, `model` (the magic-
  bytes dispatcher)
- **progs** — full QuakeC bytecode VM: 65 opcodes, edict layout,
  parser, `OP_STATE` + builtins
- **server / client / sound / renderer** — in progress; see
  [`engine`](https://github.com/go-quake1/engine) for the current state

Every ported package mirrors a tyrquake source file 1:1 under
[`reference/`](https://github.com/go-quake1/engine/tree/main/reference)
so the diff between Go and C is auditable.

## Project standards

- **Pure Go.** `CGO_ENABLED=0`. No SDL/GL/OpenAL/libc. The framebuffer
  ships through go-virtio's GPU driver and audio through its `snd`
  channel; hot loops accelerated by go-asmgen-emitted Plan 9 SIMD.
- **100% statement coverage** on every package — error branches, not
  just happy paths. Measured, not asserted.
- **Reference-mirror traceability.** Every Go file links to the
  tyrquake C function it derives from, by pinned commit + filename.
- **Provable test protocol.** Carries forward the
  [godoom](https://github.com/go-doom/engine) 4-gate protocol
  (engine-determinism BYTE-EQUAL · guest-GPU chi² bounded · audio
  events BYTE-EQUAL · audio WAV RMS bounded).
- **6-arch CI.** All six 64-bit Go targets — `amd64`, `arm64`,
  `riscv64`, `loong64`, `ppc64le`, `s390x` — green on each PR.
- **BSD-3-Clause** wrapper on the Go port + **GPL-2.0-or-later** carve-
  out on the engine subtree (mandated by upstream id Software).

## Why pure-Go Quake?

The same reason as [go-virtio](https://github.com/go-virtio) and
[go-tpm2](https://github.com/go-tpm2): **one self-contained Go binary,
zero cgo, runs on bare metal** — the TamaGo target the cloud-boot
loader hands control to. A C engine drags glibc + SDL + GL + OpenAL +
the cgo runtime; pure-Go drags exactly `runtime` + `math`. The Quake
soft renderer fits the model: no GPU shaders, all CPU rasterisation —
exactly the workload pure-Go + go-asmgen SIMD is built for.

## Who uses it

[cloud-boot](https://github.com/cloud-boot) — the bare-metal TamaGo
+ UEFI demo target. Once the engine is feature-complete, the
[godoom](https://github.com/go-doom/engine) provable-test
discipline carries forward unchanged: same gates, same reference
recordings, against the same go-virtio + go-asmgen substrate.

## Landing page

Project landing page (Hugo): coming soon — <https://go-quake1.github.io>.
