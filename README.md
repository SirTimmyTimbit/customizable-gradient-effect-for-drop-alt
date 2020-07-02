# Customizable gradient effect for Drop Alt keyboards

This effect lets you create a custom RGB gradient for in-switch and/or underglow lights right from the keyboard. I have no way to test this on Ctrl but it should work on it too.

![Header](https://i.imgur.com/te7Kyhh.jpg)

### 1. Enable custom user effects

Open `qmk_firmware/keyboards/massdrop/alt/rules.mk` and find the following lines of code:

```C
# Custom RGB matrix handling
RGB_MATRIX_ENABLE = custom
```

Underneath that, add:

```C
RGB_MATRIX_CUSTOM_USER = yes
```

You can add that line anywhere in that file, we're adding it in that section for posterity. 

### 2. Copy the effect source file

Create a new directory named `effects` inside `qmk_firmware/keyboards/massdrop/alt/keymaps/YOURKEYMAP/`. Copy `stt_custom_gradient.c` from this repository to that folder.

### 3. Create rgb_matrix_user.inc if it doesn't exist

Open `rgb_matrix_user.inc` in `qmk_firmware/keyboards/massdrop/alt/keymaps/YOURKEYMAP/` to edit. If the file doesn't exist then create it.

In that file add:

```C
/*   Custom gradient that can be tweaked by keycodes
...........................................................*/
RGB_MATRIX_EFFECT(STT_CUSTOM_GRADIENT)
#ifdef RGB_MATRIX_CUSTOM_EFFECT_IMPLS
#include "effects/stt_custom_gradient.c"
#endif
```
### 4. Edit `keymap.c`

Open `qmk_firmware/keyboards/massdrop/alt/keymaps/YOURKEYMAP/keymap.c` in your favorite editor. 

#### 4.1 Add the global variables

**Find:**

```C
#include QMK_KEYBOARD_H
```

**Add underneath:**

```C
// For STT_CUSTOM_GRADIENT
HSV stt_gradient_0 = {205, 250, 255};
HSV stt_gradient_100 = {140, 215, 125};
bool stt_reflected_gradient = false;
uint8_t stt_gp_i = 0;

typedef struct {
    HSV gradient_0;
    HSV gradient_1;
    bool reflected;
} STT_CUSTOM_PRESETS;
```

#### 4.2  Add the keycodes used to manipulate the gradient into the list of custom keycodes

**Find**

```C
enum alt_keycodes {
    U_T_AUTO = SAFE_RANGE, //USB Extra Port Toggle Auto Detect / Always Active
    U_T_AGCR,              //USB Toggle Automatic GCR control
    ...
    MD_BOOT,               //Restart into bootloader after hold timeout
};
```

Before `};`, add the following keycodes to the enum:

```C
    STT_G0_HUI,                 //Custom gradient color 1 hue increase
    STT_G0_HUD,                 //Custom gradient color 1 hue decrease
    STT_G0_SAI,                 //Custom gradient color 1 saturation increase
    STT_G0_SAD,                 //Custom gradient color 1 saturation decrease
    STT_G0_VAI,                 //Custom gradient color 1 value increase
    STT_G0_VAD,                 //Custom gradient color 1 value decrease
    STT_G100_HUI,               //Custom gradient color 2 hue increase
    STT_G100_HUD,               //Custom gradient color 2 hue decrease
    STT_G100_SAI,               //Custom gradient color 2 saturation increase
    STT_G100_SAD,               //Custom gradient color 2 saturation decrease
    STT_G100_VAI,               //Custom gradient color 2 value increase
    STT_G100_VAD,               //Custom gradient color 2 value decrease
    STT_GRADIENT_PRESETS,       //Gradient presets
    STT_REFLECTED_GRADIENT,     //Toggle between linear and reflected gradient
    STT_GRADIENT_FLIP,          //Flip the gradient colors
```

#### 4.3  Add the keycodes to your keymap

Add the keycodes listed above to your keymap, in the `const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS]` array. 

I prefer them on a separate layer. I have them on my third layer, activated by holding Fn + left Alt.

![Example keymap layout](https://i.imgur.com/aEsIdCZ.jpg)

> Row 1 (red) controls the left side of the gradient. 

> Row 2 (blue) controls the right side

> Row 3 has presets and toggles.

That's just the layout I like, feel free to arrange them in a way you prefer.

#### 4.4  Define the presets and the keycodes

**Find:**

```C
bool process_record_user(uint16_t keycode, keyrecord_t *record) {
```

**Add underneath:**

```C
uint8_t color_adj_step = 5;
STT_CUSTOM_PRESETS stt_gradient_presets[] = {
    {{205, 250, 255}, {140, 215, 125}, false },
    {{41, 255, 255}, {233, 245, 255}, false },
    {{45, 245, 155}, {160, 255, 80}, false },
    {{173, 245, 40}, {41, 255, 205}, true },
    {{32, 255, 165}, {217, 185, 70}, false },
    {{240, 255, 145}, {115, 255, 245}, true },
    {{118, 255, 255}, {242, 255, 255}, false },
    {{118, 255, 255}, {242, 255, 255}, false },
    {{212, 0, 0}, {223, 235, 165}, true },
};
uint8_t stt_gp_length = sizeof(stt_gradient_presets)/sizeof(stt_gradient_presets[0]);
```

**Find:**

```C
switch (keycode) {
```

**Add underneath:**

```C
case STT_G0_HUI:
    if (record->event.pressed) {
        stt_gradient_0.h += color_adj_step;
        dprintf("Gradient 0 HSV: %d, %d, %d\n", stt_gradient_0.h, stt_gradient_0.s, stt_gradient_0.v);
    }
    return false;
case STT_G0_HUD:
    if (record->event.pressed) {
        stt_gradient_0.h -= color_adj_step;
        dprintf("Gradient 0 HSV: %d, %d, %d\n", stt_gradient_0.h, stt_gradient_0.s, stt_gradient_0.v);
    }
    return false;
case STT_G0_SAI:
    if (record->event.pressed) {
        stt_gradient_0.s = (stt_gradient_0.s + color_adj_step * 2 <= 255) ? stt_gradient_0.s + color_adj_step * 2 : 255;
        dprintf("Gradient 0 HSV: %d, %d, %d\n", stt_gradient_0.h, stt_gradient_0.s, stt_gradient_0.v);
    }
    return false;
case STT_G0_SAD:
    if (record->event.pressed) {
        stt_gradient_0.s = (stt_gradient_0.s - color_adj_step * 2 >= 0) ? stt_gradient_0.s - color_adj_step * 2 : 0;
        dprintf("Gradient 0 HSV: %d, %d, %d\n", stt_gradient_0.h, stt_gradient_0.s, stt_gradient_0.v);
    }
    return false;
case STT_G0_VAI:
    if (record->event.pressed) {
        stt_gradient_0.v = (stt_gradient_0.v + color_adj_step * 2 <= 255) ? stt_gradient_0.v + color_adj_step * 2 : 255;
        dprintf("Gradient 0 HSV: %d, %d, %d\n", stt_gradient_0.h, stt_gradient_0.s, stt_gradient_0.v);
    }
    return false;
case STT_G0_VAD:
    if (record->event.pressed) {
        stt_gradient_0.v = (stt_gradient_0.v - color_adj_step * 2 >= 0) ? stt_gradient_0.v - color_adj_step * 2 : 0;
        dprintf("Gradient 0 HSV: %d, %d, %d\n", stt_gradient_0.h, stt_gradient_0.s, stt_gradient_0.v);
    }
    return false;
case STT_G100_HUI:
    if (record->event.pressed) {
        stt_gradient_100.h += color_adj_step;
        dprintf("Gradient 100 HSV: %d, %d, %d\n", stt_gradient_100.h, stt_gradient_100.s, stt_gradient_100.v);
    }
    return false;
case STT_G100_HUD:
    if (record->event.pressed) {
        stt_gradient_100.h -= color_adj_step;
        dprintf("Gradient 100 HSV: %d, %d, %d\n", stt_gradient_100.h, stt_gradient_100.s, stt_gradient_100.v);
    }
    return false;
case STT_G100_SAI:
    if (record->event.pressed) {
        stt_gradient_100.s = (stt_gradient_100.s + color_adj_step * 2 <= 255) ? stt_gradient_100.s + color_adj_step * 2 : 255;
        dprintf("Gradient 100 HSV: %d, %d, %d\n", stt_gradient_100.h, stt_gradient_100.s, stt_gradient_100.v);
    }
    return false;
case STT_G100_SAD:
    if (record->event.pressed) {
        stt_gradient_100.s = (stt_gradient_100.s - color_adj_step * 2 >= 0) ? stt_gradient_100.s - color_adj_step * 2 : 0;
        dprintf("Gradient 100 HSV: %d, %d, %d\n", stt_gradient_100.h, stt_gradient_100.s, stt_gradient_100.v);
    }
    return false;
case STT_G100_VAI:
    if (record->event.pressed) {
        stt_gradient_100.v = (stt_gradient_100.v + color_adj_step * 2 <= 255) ? stt_gradient_100.v + color_adj_step * 2 : 255;
        dprintf("Gradient 100 HSV: %d, %d, %d\n", stt_gradient_100.h, stt_gradient_100.s, stt_gradient_100.v);
    }
    return false;
case STT_G100_VAD:
    if (record->event.pressed) {
        stt_gradient_100.v = (stt_gradient_100.v - color_adj_step * 2 >= 0) ? stt_gradient_100.v - color_adj_step * 2 : 0;
        dprintf("Gradient 100 HSV: %d, %d, %d\n", stt_gradient_100.h, stt_gradient_100.s, stt_gradient_100.v);
    }
    return false;
case STT_GRADIENT_PRESETS:
    if (record->event.pressed) {
        stt_gp_i = (stt_gp_i + stt_gp_length ) % stt_gp_length;

        stt_gradient_0 = stt_gradient_presets[stt_gp_i].gradient_0;
        stt_gradient_100 = stt_gradient_presets[stt_gp_i].gradient_1;
        stt_reflected_gradient = stt_gradient_presets[stt_gp_i].reflected;

        stt_gp_i += 1;
    }
    return false;
case STT_REFLECTED_GRADIENT:
    if (record->event.pressed) {
        stt_reflected_gradient = !stt_reflected_gradient;
    }
    return false;
case STT_GRADIENT_FLIP:
    if (record->event.pressed) {
        HSV temp_color = stt_gradient_0;
        stt_gradient_0 = stt_gradient_100;
        stt_gradient_100 = temp_color;
    }
    return false;
```

> **Note** I'm printing out the debug output in the console as well. If you turn on debug you will see the color values of the gradient as you adjust it. You can take these values and add to `stt_gradient_presets[]` to have your own custom presets.

### 5. Compile & Flash!

That's it! After flashing, use the keycodes listed in `4.2`to play with the gradient. 

# But wait, there's more!

I'm not a big fan of the default rainbow gradient scroll effect on the Alt / Ctrl. Gradient bands are too narrow and there are too many colors on at once. 

I've changed the scale to make the bands wider, wide enough that you only see about 3 colors max at once. 

It's super easy to add this effect

![Example keymap layout](https://i.imgur.com/ecMUaKd.jpg)

### 1. Copy the effect source file

If one doesn't exist, create a new directory named `effects` inside `qmk_firmware/keyboards/massdrop/alt/keymaps/YOURKEYMAP/`. Copy `stt_wide_rainbow_gradient.c` from this repository to that folder.

### 2. Create rgb_matrix_user.inc if it doesn't exist

Open `rgb_matrix_user.inc` in `qmk_firmware/keyboards/massdrop/alt/keymaps/YOURKEYMAP/` to edit. If the file doesn't exist then create it.

In that file add:

```C
/*   Wide Rainbow Cycle Left to Right
...........................................................*/
RGB_MATRIX_EFFECT(STT_WIDE_RAINBOW_GRADIENT)
#ifdef RGB_MATRIX_CUSTOM_EFFECT_IMPLS
#include "effects/stt_wide_rainbow_gradient.c"
#endif
```

### 3. Compile & Flash!

You can use both custom gradient effect and wide rainbow gradient effect together.

# Compatibility & Support

I've tested this on my low profile Alt and it works fine. I'm fairly sure this will work in both Alt and Ctrl, low or high profile. At most thixs will require very minimal work to port.

Sadly I won't have my Alt for much longer. I won't be able to provide any future support / troubleshooting help for it. 

There will be no future updates on this effect from me.. Feel free to fork it and continue the work. Originally I had planned animating the gradient, adding more features and customizations but I'm moving onto other keyboards. 
