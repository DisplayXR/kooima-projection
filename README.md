# Kooima Projection

Generalized off-axis frustum projection math library. Implements the [Kooima algorithm](http://csc.lsu.edu/~kooima/articles/genperspective/) for asymmetric perspective projection — the correct way to render 3D content for physical displays.

## Features

- Off-axis frustum projection for any display geometry
- View-agnostic: works for stereo, mono, or N-view multiview via baseline/parallax generalization
- Zero runtime dependencies — pure math library
- C11 with no external dependencies

## Status

Extraction planned from `displayxr-runtime` (`src/xrt/auxiliary/math/m_stereo3d.*`). The projection math is view-agnostic and has no dependency on the runtime.

## Background

Traditional symmetric frustum projection assumes the viewer is centered on the display. Real 3D displays require **asymmetric (off-axis) frustums** that account for the viewer's actual position relative to the display surface. This library computes those frustums correctly.

## Related

- [displayxr-runtime](https://github.com/DisplayXR/displayxr-runtime) — Core OpenXR runtime (current home of this code)
- [Kooima's Generalized Perspective Projection](http://csc.lsu.edu/~kooima/articles/genperspective/)
