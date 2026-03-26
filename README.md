<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/DisplayXR/displayxr-runtime/main/doc/displayxr_white.png" width="100">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/DisplayXR/displayxr-runtime/main/doc/displayxr.png" width="100">
  <img alt="DisplayXR" src="https://raw.githubusercontent.com/DisplayXR/displayxr-runtime/main/doc/displayxr.png" width="100">
</picture>

# Kooima Projection

Generalized off-axis frustum projection math library for 3D displays. Implements the [Kooima algorithm](http://csc.lsu.edu/~kooima/articles/genperspective/) — the correct way to render perspective-correct stereoscopic 3D for physical displays with eye tracking.

## What It Does

Given a viewer's eye positions and a physical display's dimensions, this library computes **asymmetric frustum projection matrices** that produce geometrically correct 3D. Each eye gets a different frustum that accounts for its off-axis position relative to the display surface.

```
         Eye (tracked)
          \
           \  ← asymmetric frustum
            \
    ┌────────┴────────┐
    │  Physical Display │   ← known size in meters
    └─────────────────────┘
```

This is fundamentally different from symmetric projection (which assumes the viewer is centered) and is required for autostereoscopic/lenticular 3D displays.

## Two Pipelines

### Display-Centric (`display3d_view.*`)

The physical display is the reference frame. The library maps the display into your app's virtual world and computes view + projection matrices from each eye's position relative to the screen surface.

**Use when:** Your app has a concept of a "virtual display" — the screen is a window into a 3D world, and the viewer looks through it.

```c
Display3DTunables tunables = {
    .ipd_factor = 1.0f,              // full stereo separation
    .parallax_factor = 1.0f,         // full eye tracking
    .perspective_factor = 1.0f,      // 1:1 perspective
    .virtual_display_height = 0.5f   // display is 0.5 app-units tall
};

Display3DScreen screen = { .width_m = 0.344f, .height_m = 0.194f };

Display3DView views[2];
display3d_compute_views(
    eye_positions, 2,   // N eye positions from tracker
    &nominal_viewer,    // default viewer position
    &screen,            // physical display dimensions
    &tunables,
    &display_pose,      // display position/orientation in world
    near, far,
    views               // output: view_matrix + projection_matrix per eye
);
```

### Camera-Centric (`camera3d_view.*`)

The app defines a virtual camera (position, orientation, vFOV) and eye tracking produces per-eye asymmetric views around that camera. The convergence distance controls where left/right views overlap.

**Use when:** Your app has a traditional camera (FPS game, 3D viewer) and you want to add eye-tracked stereo without rethinking your rendering pipeline.

```c
Camera3DTunables tunables = {
    .ipd_factor = 1.0f,
    .parallax_factor = 1.0f,
    .inv_convergence_distance = 2.0f,  // converge at 0.5m
    .half_tan_vfov = 0.577f            // 60 degree vFOV
};

Camera3DView views[2];
camera3d_compute_views(
    eye_positions, 2,
    &nominal_viewer,
    &screen,
    &tunables,
    &camera_pose,       // camera position/orientation in world
    near, far,
    views
);
```

## Eye Factor Processing

Both pipelines share a common first stage that processes raw eye positions:

1. **IPD Factor** (`ipd_factor`, 0-1) — scales the inter-eye distance around the midpoint. 0 = mono (both eyes at center), 1 = full physical IPD.

2. **Parallax Factor** (`parallax_factor`, 0-1) — lerps the eye center toward the nominal (default) viewer position. 0 = no head tracking (fixed viewpoint), 1 = full tracking.

These factors are applied before the view/projection computation, allowing smooth transitions between mono/stereo and tracked/untracked modes.

## Files

| File | Description |
|------|-------------|
| `include/display3d_view.h` | Display-centric API — structs and function declarations |
| `include/display3d_view.c` | Display-centric implementation |
| `include/camera3d_view.h` | Camera-centric API — structs and function declarations |
| `include/camera3d_view.c` | Camera-centric implementation |

## Integration

Copy the files into your project. The only dependency is `<openxr/openxr.h>` for `XrVector3f`, `XrPosef`, `XrFovf` types. If you're not using OpenXR, you can replace these with your own equivalent structs.

```cmake
# CMake example
add_library(kooima_projection STATIC
    include/display3d_view.c
    include/camera3d_view.c
)
target_include_directories(kooima_projection PUBLIC include)
target_link_libraries(kooima_projection PUBLIC OpenXR::openxr_loader)  # for XR types
```

**Matrix convention:** All output matrices are **column-major** (OpenGL / Vulkan / Metal). DirectX callers should transpose when loading into row-major `XMMATRIX`.

## N-View Multiview

Both APIs accept an array of N eye positions and produce N views. This works for stereo (2 views), quad (4 views), or any N-view light field display:

```c
XrVector3f eyes[4] = { ... };  // 4 eye positions from tracker
Display3DView views[4];
display3d_compute_views(eyes, 4, &nominal, &screen, &tunables, &pose, near, far, views);
// views[0..3] each have their own view_matrix + projection_matrix
```

## Documentation

- [Math Reference](docs-math-reference.md) — full pipeline derivation with diagrams
- [DisplayXR Runtime](https://github.com/DisplayXR/displayxr-runtime) — the OpenXR runtime that uses this library
- [Kooima's Original Paper](http://csc.lsu.edu/~kooima/articles/genperspective/) — the foundational algorithm

## License

[ISC License](LICENSE) — same as the DisplayXR runtime.

## Auto-Synced

These files are the canonical copies from the [displayxr-runtime](https://github.com/DisplayXR/displayxr-runtime) repository (`test_apps/common/display3d_view.*` and `camera3d_view.*`). To contribute, open a PR on the runtime repo.
