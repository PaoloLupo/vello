# Gaanim patches to vello_shaders

Gaanim builds `vello_shaders` from this branch (see `GAANIM.md` at the root)
through its `[patch.crates-io]`, so these fixes reach the preview, the exports
and the web players.

## Non-separable blends of grays

`clip_color` in `shader/shared/blend.wgsl` divides by `l - n` when a channel
is below 0 and by `x - l` when one is above 1, without checking the
denominator. After `set_lum`, a gray has three equal channels, so the
denominator is 0, or rounds a little below it, and the division gives NaN.
The fine stage stores NaN as 0: the pixel comes out black. It happens even
where the blended layer's source is clear, since the blend runs over the whole
layer and `mix(backdrop, NaN, 0)` is still NaN. In Gaanim a drawable with
`.blend("luminosity")` drew black columns on a gray ramp beneath its layer
box; on other GPUs `hue`, `saturation` and `color` did too.

Such a gray is now clipped to its luminosity, clamped to `[0, 1]`. The
denominators stay above the `1e-6` threshold for any color whose channels
differ by more than about `1e-5`, so other colors blend as before.
linebender/vello#1999 reports the same code in `vello_cpu`.

Regression check:

- `just dev test -p gaanim_export -p gaanim_bundle --lib non_separable`: a
  hue, saturation, color and luminosity layer with a clear source leaves all
  256 gray levels beneath it unchanged (gray 29 came out black before).

Drop this commit when rebasing onto an upstream release that guards
`clip_color`.
