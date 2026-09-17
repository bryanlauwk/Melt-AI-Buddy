# MELT / DURI Compatibility and Build Studio Overhaul

## Decision

MELT Core V1 can use the Mac AI Buddy architecture without abandoning the current BOM.

The electrical architecture is a strong match. The remaining uncertainty is mechanical: the five printed parts, the actual SG90 horns, fasteners, OLED board, XIAO camera assembly, USB cable and wire bends must be fit-checked and measured before the simulation or shell is treated as production-accurate.

**Recommendation:** keep the printed Core V1 as the engineering skeleton, adapt the Mac AI Buddy software architecture, and make the Clear Jacket a removable cosmetic layer that never carries servo loads.

## Compatibility matrix

| Area | Mac AI Buddy | MELT / DURI parts | Assessment | Required action |
|---|---|---|---|---|
| Controller | Seeed Studio XIAO ESP32S3 Sense | Same board | Exact match | Retain USB-C access, antenna clearance and reset access. |
| Camera | XIAO Sense onboard OV2640 | XIAO Sense camera; live test passed | Sensor variant must be verified | Preserve lens sightline and flex-cable clearance. |
| Display | 1.5-inch SH1107 128×128 is the default; SSD1306 128×64 is supported | 0.96-inch SSD1306 128×64 module | Supported variant, not the default | Select the SSD1306 driver, verify I²C address, and redraw facial expressions for the wide 128×64 canvas. |
| Servo driver | PCA9685 at I²C address 0x40 | Same board type | Exact functional match | Confirm actual header orientation and use the firmware channel definitions. |
| Pan/tilt | Two SG90 servos | Two SG90 servos | Exact component class | Fit-test bodies and horns; measure neutral position and safe limits before powered motion. |
| I²C | XIAO D4/GPIO5 SDA, D5/GPIO6 SCL | Same intended bus | Strong match | Share SDA/SCL between OLED and PCA9685; confirm unique addresses and common ground. |
| Power | 5 V / 4 A in the upstream build with onboard audio | 5 V / 3 A available | Likely adequate for the current core, not yet load-tested | Test servo peak current and voltage drop. Re-evaluate before adding speaker/amplifier; add bulk capacitance near PCA9685 V+. |
| Audio | Optional MAX98357 speaker amp and MAX4466 microphone | Not currently in the working BOM | Not required for Phase 1 | Use the laptop microphone and speaker first. Treat body audio as a later plug-in module. |
| Mechanical body | Mac-style custom enclosure | MELT five-part black PETG Core V1 plus future Clear Jacket | New mechanical implementation | Verify every interface physically; do not infer fit from concept renders. |
| Breadboard | Development aid only | Available on bench | Prototype-only | Do not include the full breadboard in the final moving core. Replace with secured wiring or a small carrier after validation. |

## Kinematic architecture

The simulation and physical build must share the same parent-child structure:

1. **Fixed base:** base tray, PCA9685, power distribution and cable strain relief.
2. **Pan axis:** lower SG90 mounted so its output shaft is vertical. It rotates the complete neck, tilt assembly and head left/right.
3. **Tilt carrier:** yoke attached to the pan output.
4. **Tilt axis:** upper SG90 mounted so its output shaft is horizontal. It rotates only the head tray up/down.
5. **Head payload:** OLED bezel, XIAO ESP32S3 Sense, camera, and the shortest practical wiring service loop.
6. **Clear Jacket:** removable cosmetic shell attached to the fixed structure with motion clearance around the head and neck.

The pan and tilt servos must not appear at the same orientation. Their rotational axes are perpendicular.

## Firmware and software adaptation

The upstream [Mac AI Buddy](https://github.com/AyhanSh/Mac-AI-Buddy) already provides the right overall split:

- ESP32 firmware controls the OLED, camera and PCA9685 servos.
- A laptop-side Python process handles microphone input, AI providers, speech and high-level robot commands.
- The robot is controlled through HTTP endpoints such as status, speech, camera capture and look direction.
- OpenAI and Gemini providers can sit behind the same robot interface.

MELT-specific changes:

- Select the SSD1306 128×64 display option instead of the default SH1107 128×128 option.
- Recompose the face art for the wide screen; do not scale the square face assets blindly.
- Treat the source-code servo channel definitions as authoritative: pan channel 3 and tilt channel 4. Verify the physical cable positions before enabling movement.
- Begin with laptop microphone and speaker. This keeps the first demonstrator lighter and avoids adding power/noise problems.
- Keep the already-passed XIAO camera test as a known-good baseline.
- Add automatic visitor tracking only after the mechanics are safe. The upstream system can accept look commands, but continuous local face tracking is not a completed core feature.
- Preserve the HTTP boundary so a later Raspberry Pi, mini-PC, phone or cloud gateway can replace the Mac without redesigning the robot electronics.

## Build Studio hero: realistic interactive digital twin

The hero should no longer be a static CAD illustration. It should be a measured digital twin tied to the actual build.

### Default presentation

- Photorealistic 3/4 assembly view in the centre.
- One-click side, front and top views.
- Side-by-side real build photo using the same approximate camera angle.
- A persistent status label: **Concept**, **Nominal dimensions**, **Measured**, **Fit verified**, or **Motion tested**.

### Clickable parts

Selecting a real component should:

- isolate and highlight it;
- show its real photo and BOM identity;
- show exact, nominal or measured dimensions with confidence labels;
- reveal its mounting points, horn, screws and cable;
- open the exact build step and shopping gap if a fastener is missing.

### Pan and tilt interaction

- Clicking the lower servo isolates the **pan** chain and displays the vertical axis.
- Clicking the upper servo isolates the **tilt** chain and displays the horizontal axis.
- A slider rotates the correct child assembly around the real output-shaft pivot.
- A centre button returns both axes to the measured neutral pose.
- A short sweep can be previewed, but initial limits must be marked **simulation only**.
- Physical limits replace nominal values only after hand-clearance and low-speed tests.
- Collision overlays warn about head-to-base contact, horn-to-bracket contact, shell interference and wire over-bending.

### Reality comparison

The user can upload front, side and 3/4 build photos. The studio then keeps:

- the real photo on one side;
- the digital twin on the other;
- synchronized view selection;
- measurement prompts for any mismatch;
- a checklist for mounting, cable routing and missing hardware.

This turns the hero into the main build instrument, not decorative marketing.

## Delivery plan

### Gate A — physical fit test

After collecting the five PETG parts:

1. Photograph and measure every printed part.
2. Dry-fit both SG90 bodies with no power.
3. Identify the correct horns, screw diameters and screw lengths.
4. Place the OLED and XIAO/camera while preserving USB, antenna, reset and camera clearances.
5. Record head depth, neck clearance and base footprint.
6. Route a loose wire loop through the full intended movement.
7. Mark each interface as pass, revise or blocked.

### Gate B — digital twin

- Import the five source STLs.
- Replace generic electronics blocks with dimensioned component models.
- Set pivots from the real horn centres.
- Bind each component to the kinematic hierarchy.
- Add 3/4, side, front and top camera presets.
- Add fit-confidence labels and real-photo comparison.

### Gate C — motion simulation

- Add pan and tilt sliders.
- Show axes, swept volumes and collision warnings.
- Start with conservative temporary limits.
- Replace them with measured safe limits from the physical core.

### Gate D — electronics bring-up

Test independently, in this order:

1. XIAO power and serial.
2. Camera.
3. OLED.
4. PCA9685 without servos.
5. Each servo at neutral with an unloaded horn.
6. Pan assembly at low speed.
7. Tilt assembly at low speed.
8. Both axes with the real head payload.

### Gate E — Mac AI Buddy adaptation

- Integrate the licensed or permissioned upstream implementation while preserving attribution and history.
- Apply the SSD1306 configuration and MELT pin/channel map.
- Retain laptop audio for the first complete demo.
- Calibrate endpoints and document them in the repository.
- Add conversation, expressions and deliberate movements.

### Gate F — character and shell

- Design the Clear Jacket around the motion-tested core.
- Keep the electronics module removable.
- Make patches, inserts and resin icons cosmetic and replaceable.
- Validate camera visibility, Wi-Fi antenna performance, ventilation, USB access and serviceability.

## Go / no-go assessment

**Go:** the current electronics are compatible enough to continue, and the Mac AI Buddy split is a suitable foundation for MELT/DURI.

**Do not yet claim:** final mechanical fit, safe full servo range, standalone operation, automatic tracking or production-ready battery/power behaviour. These become verified claims only after the physical gates above.

The 0.96-inch OLED is not a project blocker. It changes the face proportions and bezel design, not the fundamental architecture.

## Upstream permission and provenance

Bryan confirmed author permission on 17 September 2026. The supplied ZIP snapshot is now preserved in `upstream/Mac-AI-Buddy/`, with upstream attribution in `UPSTREAM.md`. No upstream firmware was modified or flashed as part of the simulation work.

## Simulation delivered

The Build Studio now has clickable frame/electronics, perpendicular pan and tilt joints, relative-angle sliders, a ±10° preview, camera presets and real component photos. STL geometry is source-based; mounting transforms and electronics geometry remain provisional. It does not perform collision, torque or wiring validation and cannot control hardware.
