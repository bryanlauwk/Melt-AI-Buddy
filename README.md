# MELT AI Buddy

MELT / DURI adapts the Mac AI Buddy architecture to Bryanlauwk Create's palm-size interactive character and the Core V1 black PETG mechanical skeleton.

## Current status

Planning and physical fit-test stage.

The upstream implementation is [AyhanSh/Mac-AI-Buddy](https://github.com/AyhanSh/Mac-AI-Buddy). Bryan confirmed author permission on 17 September 2026. The supplied source archive is preserved in `upstream/Mac-AI-Buddy/`; see `UPSTREAM.md` for provenance. This permission does not establish a general open-source licence for others.

## Confirmed architecture match

- Seeed XIAO ESP32S3 Sense with onboard OV2640 camera
- SSD1306 128 x 64 OLED supported by the upstream firmware's display-driver switch
- PCA9685 I2C servo driver
- Two SG90 servos for pan and tilt
- Laptop-side Python brain communicating with the ESP32 over HTTP
- OpenAI or Gemini provider behind the same robot interface

## MELT-specific differences

- 0.96-inch SSD1306 instead of the upstream default 1.5-inch SH1107
- 5 V / 3 A supply for the current core; re-evaluate before adding onboard speaker and amplifier
- Core V1 five-part PETG frame and removable cosmetic Clear Jacket
- Laptop microphone and speaker first; onboard audio becomes an optional module
- Physical fit checks and measured motion limits are required before firmware movement

## Development gates

1. Fit both SG90 bodies, horns and fasteners to Core V1.
2. Confirm OLED and XIAO mounting, USB access and camera sightline.
3. Record safe pan and tilt limits.
4. Build the interactive digital twin from the measured assembly.
5. Adapt firmware pinout, display driver and servo channels.
6. Run camera, OLED and neutral-servo tests independently.
7. Add behaviour, conversation and optional local tracking.

See `docs/COMPATIBILITY-AND-OVERHAUL.md` for the detailed review and product plan.
