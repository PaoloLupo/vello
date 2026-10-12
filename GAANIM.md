# Gaanim's Vello branch

`gaanim-0.9` is Vello `v0.9.0` (commit `875f324f`) with the fixes Gaanim needs,
one commit each. Gaanim's workspace takes `vello_encoding` and
`vello_shaders` from it at a pinned revision; `vello` itself still comes from
crates.io:

```toml
[patch.crates-io]
vello_encoding = { git = "https://github.com/PaoloLupo/vello", rev = "<commit>" }
vello_shaders = { git = "https://github.com/PaoloLupo/vello", rev = "<commit>" }
```

Each crate's `GAANIM-PATCH.md` explains its fixes and how to check them:

- `vello_encoding`: the image atlas across vector-only frames, bump buffers
  sized from the scene, gradient ramps kept between renders and sampled
  faster, and a lint expectation newer toolchains no longer need.
- `vello_shaders`: non-separable blends that turned some grays black.

To move Gaanim to a newer Vello, start a branch from that release's tag,
cherry-pick the commits it still needs, and point `rev` in Gaanim at it.
Commit to a branch and push before changing `rev`: Gaanim's `Cargo.lock` is
not versioned, so the revision is what keeps every checkout on the same code.
