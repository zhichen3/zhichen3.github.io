+++
title = "Ploopy Trackball Mouse"
author = ["zhi"]
date = 2026-08-18T00:00:00-04:00
tags = ["ploopy"]
categories = ["tutorial"]
type = "posts"
draft = false
weight = 1002
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [What is Ploopy?](#what-is-ploopy)
- [Ploopy Classic Trackball](#ploopy-classic-trackball)
- [QMK Setup](#qmk-setup)
- [Enable Drag Scroll via QMK](#enable-drag-scroll-via-qmk)

</div>
<!--endtoc-->


## What is Ploopy? {#what-is-ploopy}

[Ploopy](https://ploopy.co/) is a company located in Canada that makes open-source mouse
with 3D printing material. They're especially known for
creating trackball mouses with modern sensors.
Ploopy devices use [QMK](https://qmk.fm/) firmware by default, this
gives you a lot of flexibility to customize your mouse.

---


## Ploopy Classic Trackball {#ploopy-classic-trackball}

Ploopy [Classic](https://ploopy.co/classic-2-trackball/) is their most iconic model,
a modern, open-source 3D-printed imitation to
the Microsoft Trackball Explorer.

One of the weakest point on the Ploopy Classic (I think)
is the scroll wheel.
It is not pleasant to use. To fix this, it would be nice
if we can hold or click a button, which then enables drag scroll,
so that we can simply use the trackball itself to do scrolling.
This is especially helpful if we need to scroll through a
wide range of pages. Luckily, QMK Ploopy firmware already supports
this, but we need to compile the firmware with the setting enabled,
and flash the firmware to the mouse.

---


## QMK Setup {#qmk-setup}

We need to first set up QMK by simply following their [documentation](https://docs.qmk.fm/newbs_getting_started).

First we need to install QMK via

```bash
curl -fsSL https://install.qmk.fm | sh
```

Now we need to setup the QMK environment. It is generally
preferred to setup based on your own fork on QMK firmware.
To do this, simply go to their [GitHub](https://github.com/qmk/qmk_firmware) and create a fork,
but this is not necessary.
Now we set up the QMK environment via

```bash
qmk setup <github_username>/qmk_firmware -H <path>
```

If you didn't create your own fork of qmk firmware,
simply leave that option blank. Then put the `<path>`
where you want to install QMK home directory.
I personally prefer to put it in `$HOME/Desktop`.
The default location is to put it in `$HOME`.
Now thats done, we can compile our own version of
Ploopy Classic Trackball Mouse.

---


## Enable Drag Scroll via QMK {#enable-drag-scroll-via-qmk}

After QMK is set up, go to the default ploopy mouse firmware in
`/qmk_firmware/keyboards/ploopyco/trackball/`. Here `trackball`
refers to the ploopy trackball mouse.
For the Ploopy trackball mouse, there are a total of 5 buttons.
This means we have 2 extra buttons, located to the right of
trackball, for customizations. These were used for page backward
and forward by default, but I never use these.
Instead I want to map:

1.  One button to cycle through different DPI settings
2.  One button once clicked, it toggles to drag scrolling mode.

We first need to map keys (buttons) on our mouse to have the
corresponding function, and thankfully, there are already these
key-codes available. To create a custom keymap, let's create
a subdirectory called `custom`,
`/qmk_firmware/keyboards/ploopyco/trackball/keymaps`,
to contain our custom settings.
We already have a default keymap located in `/trackball/keymaps/default`.
For starter, copy the `keymap.c` file in `/default` to `/custom`.
The original `keymap.c` should look like:

```c
#include QMK_KEYBOARD_H

const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    [0] = LAYOUT( /* Base */
        MS_BTN1, MS_BTN3, MS_BTN2,
          MS_BTN4, MS_BTN5
    ),
};
```

Now we want to edit our `/keymaps/custom/keymap.c` to have:

```c
#include QMK_KEYBOARD_H

const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    [0] = LAYOUT( /* Base */
        MS_BTN1, MS_BTN3, MS_BTN2,
          DPI_CONFIG, DRAG_SCROLL
    ),
};
```

where I have changed `MS_BTN4` to `DPI_CONFIG` and `MS_BTN5` to `DRAG_SCROLL`.
Now we want to add a configuration file to have some setting.
Create a `config.h` file in `/keymaps/custom` to have

```c
#pragma once

// Invert Drag Scroll Direction
// Scroll trackball upwards to scroll backwards
#define PLOOPY_DRAGSCROLL_INVERT 1

// Different DPI Settings
#define PLOOPY_DPI_OPTIONS { 400, 600, 800, 1200, 1600 }
#define PLOOPY_DPI_DEFAULT 1

// Fix Drag Scroll DPI to 100
#define PLOOPY_DRAGSCROLL_DPI 100
```

Now we can compile via:

```bash
qmk compile -kb ploopyco/trackball/<rev-#> -km custom
```

where `<rev-#>` is the revision number of your ploopy classic
trackball, this you need to check yourself.
For ploopy classic 2, `<rev-#>` is `rev1_007`.

After its compiled, unplug your ploopy mouse,
then hold down button 4 (the one right next to trackball)
and plug in again, and then release the button.
Now even though its plugged in, you won't be able to use
the mouse.
Now in terminal, we flash the firmware via

```bash
qmk flash -kb ploopyco/trackball/<rev-#> -km custom
```

If all steps are done correctly, the your ploopy mouse
should have the ability to do drag scroll once you hit button 5.

We can clean up the build file via

```bash
qmk clean
```

---
