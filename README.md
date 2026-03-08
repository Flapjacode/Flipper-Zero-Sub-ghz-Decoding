# Flipper-Zero-Sub-ghz-Decoding

## Big Picture: How Sub-GHz Works on Flipper Zero

Flipper uses a CC1101 RF transceiver radio chip to send and receive signals between 300 MHz and ~928 MHz.

Common devices using these frequencies:

Garage door remotes

Gate openers

Key fobs

Doorbells

Smart switches

Alarm sensors

Wireless pagers

Most US remotes are around 433.92 MHz.

The process is basically:

Remote → Radio signal → Flipper receives pulses → Flipper interprets bits

or

.SUB file → Encoder → Radio signal → Receiver device
The Three Ways Flipper Captures Signals
1️⃣ Read RAW

This captures actual signal timing.

Example:

RAW_Data:
133 -4806 163 -468 453 -166

Meaning:

Tone for 133 µs
Silence for 4806 µs
Tone for 163 µs
Silence for 468 µs

So the Flipper is basically recording:

ON duration
OFF duration
ON duration
OFF duration

This is the lowest level capture.

Advantages:

✔ works for unknown protocols
✔ useful for reverse engineering

Disadvantages:

✖ large files
✖ retransmission may not always work perfectly

2️⃣ Read (Protocol Mode)

Instead of saving raw pulses, Flipper tries to decode them.

Example result:

Protocol: Princeton
Key: 52811C
Bit: 24

This means the signal matched a known protocol.

Advantages:

✔ smaller files
✔ cleaner retransmission
✔ easier to edit

3️⃣ BinRAW

Intermediate format.

It converts pulse timing into binary using a base timing value (TE).

Example:

Bit_RAW: 130
Data_RAW: 00 00 01 1D ...

Which becomes:

1000 1110 1000 ...
Static vs Rolling Codes (Huge Difference)
Static Code

Every button press sends the same code.

Example:

010100101000000100011100

This can be cloned.

Typical devices:

cheap RF outlets

old gate remotes

wireless doorbells

Rolling Code (Dynamic)

Each press changes the code.

Example sequence:

Press 1 → 39AF13
Press 2 → 82D991
Press 3 → F102C7

These are generated using cryptographic algorithms like:

KeeLoq rolling code

Security+ systems

Somfy

Nice Flor

Flipper cannot replay these directly in official firmware.

Instead you must:

pair a new remote

or implement the algorithm

Modulation (How the Data Is Carried)

The radio signal is the carrier frequency (ex: 433 MHz).

The data is encoded using modulation.

OOK / ASK (AM)

Simplest.

1 = signal ON
0 = signal OFF

This is what most cheap remotes use.

Flipper presets:

AM650
AM270
FSK (FM)

Instead of turning signal on/off, it shifts frequency.

1 = frequency slightly higher
0 = frequency slightly lower

Used in:

car key fobs

pagers

some alarms

Flipper presets:

FM238
FM476
Important Radio Parameters
Frequency

Example:

433920000 Hz

That’s:

433.92 MHz
Deviation

For FSK signals.

Example:

15.869 kHz

Meaning the signal shifts:

Carrier + 15.869 kHz
Carrier - 15.869 kHz
Bandwidth

Defines how much frequency range the receiver listens to.

Example:

650 kHz

Large bandwidth:

✔ easier capture
✖ more noise

Small bandwidth:

✔ cleaner
✖ easier to miss signal

Data Rate

Example:

3794 baud

Meaning ~3794 bits per second.

Higher rates improve timing accuracy.

SUB File Structure

Typical decoded file:

Filetype: Flipper SubGhz Key File
Version: 1
Frequency: 433920000
Preset: FuriHalSubGhzPresetOok650Async
Protocol: Princeton
Bit: 24
Key: 52811C
TE: 154

Meaning:

frequency → 433.92 MHz
modulation → OOK
protocol → Princeton
data → 52811C
timing → 154 microseconds
RAW File Example
Protocol: RAW
RAW_Data: 133 -4806 163 -468 ...

Positive numbers = signal
Negative numbers = silence

How Protocol Decoders Work

Inside firmware there are decoder modules.

Located in:

lib/subghz/protocols/

Each protocol has functions like:

feed()
reset()
serialize()
deserialize()

Example:

feed(level, duration)

Receives each pulse and tries to reconstruct bits.

CC1101 Registers (The Radio Brain)

Every preset (AM650, FM238, etc.) is just a set of register values.

Example:

02 0D
03 07
12 30
10 17

These configure things like:

modulation

bandwidth

data rate

power output

filters

Transmission Power

Controlled by PATable.

Typical values:

C0 = ~10 dBm ≈ 10 mW
00 = off
Why Some Signals Won't Clone

Usually because of:

1️⃣ rolling code
2️⃣ wrong modulation
3️⃣ wrong deviation
4️⃣ wrong data rate
5️⃣ encryption

Practical Workflow (Real Usage)

Typical process for cloning a remote:

Step 1

Frequency analyzer

Press remote.

See:

433.92 MHz
Step 2

Read RAW

Settings:

Frequency: 433.92
Modulation: AM650

Press remote.

Step 3

Check if Flipper decodes protocol.

If yes:

Save protocol

If no:

save RAW
Step 4

Replay

Press:

Emulate
Range (Typical)

Internal antenna:

50–150 ft

External CC1101 + antenna:

200–300+ ft
The Hidden Superpower: Custom Presets

You can create your own modulation profiles by editing:

SDCARD/subghz/assets/setting_user

Example:

Custom_preset_name: MyPreset
Custom_preset_module: CC1101
Custom_preset_data:
02 0D 03 07 08 32 ...

This lets you decode weird devices.

Why This Matters for Developers

This is the path to building:

custom decoders

custom RF tools

signal analyzers

RF fuzzers

RF security testing tools

Which is exactly what firmware like:

Flipper Zero Unleashed Firmware

RogueMaster Firmware

expand on.
