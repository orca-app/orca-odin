# Orca Odin Bindings Generator

This repo contains a single python script `gen.py` to convert the automatically generated `api.json` file from the [orca repo](https://github.com/orca-app/orca/blob/main/src/api.json) to odin bindings.

# Odin Support

Odin natively supports orca through a special target called: `-target:orca_wasm32`.

The generated bindings are already contained in the current odin `core:sys/orca` folder: [Odin](https://github.com/odin-lang/Odin/tree/master/core/sys/orca).

# How To Use

Orca is natively supported with odin as of the september 2024 release. This repository should not be used by normal users - rather it should be run and updated in the before mentioned `core:sys/orca` folder.

# Example

Given a `src` folder containing a file with the below code:

```odin
package src

import oc "core:sys/orca"

main :: proc() {
    
}
```

Will build with the command:

```
odin build src -target:orca_wasm32 -out:module.wasm 
orca bundle --name output module.wasm
```

Orca has to be properly installed and existing in the path variables.

# Other Examples

Printing Window Size Changes: 
```odin
package src

import "base:runtime"
import "core:fmt"
import oc "core:sys/orca"

main :: proc() {

}

@(export)
oc_on_resize :: proc "c" (width, height: u32) {
    context = runtime.default_context()
    text := fmt.ctprintf("Width, Height: %d, %d", width, height)
    oc.log_info(text)
}
```

Using `core:log`:
```odin
package src

import "base:runtime"
import "core:log"
import oc "core:sys/orca"

logger: log.Logger

main :: proc() {
    logger = oc.create_odin_logger()
}

@(export)
oc_on_resize :: proc "c" (width, height: u32) {
    context = runtime.default_context()
    context.logger = logger
    log.infof("width, height: %d %d", width, height)
}
```

Basic Rectangle Fill:
```odin
package src

import oc "core:sys/orca"

surface: oc.surface = {}
renderer: oc.canvas_renderer = {}
canvas: oc.canvas_context = {}

main :: proc() {
    renderer = oc.canvas_renderer_create()
    surface = oc.canvas_surface_create(renderer)
    canvas = oc.canvas_context_create()
}

@(export)
oc_on_frame_refresh :: proc "c" () {
    oc.canvas_context_select(canvas)
    oc.set_color_rgba(1, 1, 1, 1)
    oc.clear()

    oc.set_color_rgba(0, 0, 0, 1)
    oc.rectangle_fill(50, 50, 100, 100)

    oc.canvas_render(renderer, canvas, surface)
    oc.canvas_present(renderer, surface)    
}
```

Basic Text Rendering:
```odin
package src

import oc "core:sys/orca"

surface: oc.surface = {}
renderer: oc.canvas_renderer = {}
canvas: oc.canvas_context = {}
font: oc.font = {}

main :: proc() {
    renderer = oc.canvas_renderer_create()
    surface = oc.canvas_surface_create(renderer)
    canvas = oc.canvas_context_create()

    ranges := [?]oc.unicode_range {
        oc.UNICODE_BASIC_LATIN,
        oc.UNICODE_C1_CONTROLS_AND_LATIN_1_SUPPLEMENT,
        oc.UNICODE_LATIN_EXTENDED_A,
        oc.UNICODE_LATIN_EXTENDED_B,
        oc.UNICODE_SPECIALS,
    }

    font = oc.font_create_from_path("segoeui.ttf", 5, &ranges[0])    
}

@(export)
oc_on_frame_refresh :: proc "c" () {
    oc.canvas_context_select(canvas)
    oc.set_color_rgba(1, 1, 1, 1)
    oc.clear()

    oc.set_font(font)
    oc.set_font_size(40)

    oc.set_color_rgba(0, 0, 0, 1)
    oc.text_fill(50, 50, "Hello World")

    oc.canvas_render(renderer, canvas, surface)
    oc.canvas_present(renderer, surface)    
}
```

Mouse following Rectangle:
```odin
package src

import oc "core:sys/orca"

surface: oc.surface = {}
renderer: oc.canvas_renderer = {}
canvas: oc.canvas_context = {}
mouse: oc.vec2

main :: proc() {
    renderer = oc.canvas_renderer_create()
    surface = oc.canvas_surface_create(renderer)
    canvas = oc.canvas_context_create()
}

@(export)
oc_on_raw_event :: proc "c" (event: ^oc.event) {
    if event.type == .MOUSE_MOVE {
        mouse = { event.mouse.x, event.mouse.y }
    }    
}

@(export)
oc_on_frame_refresh :: proc "c" () {
    oc.canvas_context_select(canvas)
    oc.set_color_rgba(1, 1, 1, 1)
    oc.clear()

    oc.set_color_rgba(0, 0, 0, 1)
    oc.rectangle_fill(mouse.x - 50, mouse.y - 50, 100, 100)

    oc.canvas_render(renderer, canvas, surface)
    oc.canvas_present(renderer, surface)    
}
```

# Further C to Odin Samples

Other more indepth examples have been ported from C to Odin and shared on the odin examples [repository](https://github.com/odin-lang/examples/tree/master/orca)