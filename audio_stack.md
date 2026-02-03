# The Linux Audio Stack

Linux audio is built as a layered architecture:

**Application → Audio API → Sound Server → ALSA → Kernel Drivers → Hardware**

This is not accidental. Each layer solves a different engineering problem.

Linux audio layers commonly include:

* Applications (players, browsers, DAWs)
* User-space APIs and libraries (ALSA lib, PulseAudio API, JACK API)
* Sound servers (PulseAudio, PipeWire, JACK)
* Kernel subsystem (ALSA kernel drivers)
* Hardware (sound card, DAC, speakers, microphone)

This layered design is documented as the standard Linux audio model.

---

## Hardware Layer: Sound Cards and Digital Audio

At the lowest level, audio hardware consists of:

* ADC (Analog-to-Digital Converter) for microphones
* DAC (Digital-to-Analog Converter) for speakers
* Sound card controller
* Buffers and clocks

The hardware processes PCM (Pulse Code Modulation) audio streams.

The hardware does not understand applications. It only understands raw digital samples.

---

## Kernel Layer: ALSA (Advanced Linux Sound Architecture)

### What ALSA is

ALSA is the Linux kernel’s sound subsystem.

It provides:

* Device drivers for sound cards
* Kernel modules for PCM, mixer, MIDI, timers
* Interfaces like `/dev/snd/*`
* Communication between kernel and user space

ALSA kernel modules include:

* `snd_pcm` for audio streams
* `snd_mixer` for volume and mixing
* `snd_timer` for timing
* `snd_seq` for MIDI

The kernel layer’s role is simple:

> Communicate with audio hardware.

This is the lowest software layer in Linux audio.

---

### ALSA User Space Library (libasound)

ALSA also has a user-space library (`libasound`).

This library:

* Exposes APIs like `snd_pcm_open()`
* Sends IOCTL calls to kernel drivers
* Allows applications to play or record audio

Important point:

ALSA alone can play sound, but it has limitations.

Example limitations:

* Hardware often cannot mix multiple audio streams
* One application may lock the device
* Complex routing is difficult

This is why sound servers exist.

---

## Sound Servers

A sound server is a user-space process that sits between applications and ALSA.

Its purpose is to:

* Mix multiple audio streams
* Route audio between devices
* Resample audio
* Manage volume per application
* Provide network audio
* Provide low-latency pipelines

Applications send audio to the sound server instead of directly to ALSA.

### PulseAudio: Desktop Sound Server

#### Role of PulseAudio

PulseAudio is a sound server that sits above ALSA.

Typical architecture:

**Application → PulseAudio → ALSA → Hardware**

PulseAudio:

* Accepts audio from multiple applications
* Mixes and routes streams
* Sends final output to ALSA

It uses a client-server model where applications talk to the PulseAudio daemon rather than hardware.

PulseAudio’s main goals:

* Desktop usability
* Per-application volume control
* Bluetooth audio
* Network audio streaming

In many systems, ALSA is configured to use a virtual device provided by PulseAudio.

#### Strengths and Weaknesses of PulseAudio

Strengths:

* Easy desktop audio management
* Flexible routing
* Multiple streams

Weaknesses:

* Higher latency
* Less suitable for professional audio

This is why JACK existed.

### JACK: Professional Low-Latency Audio

JACK is another sound server focused on:

* Real-time audio
* Very low latency
* Professional audio routing

It is widely used in music production.

JACK allows applications to connect audio streams like a virtual patch bay.

However, JACK and PulseAudio historically conflicted.

### PipeWire: Unified Modern Audio System

#### What PipeWire Is

PipeWire is a modern multimedia engine designed to replace:

* PulseAudio (desktop audio)
* JACK (professional audio)
* Parts of ALSA user-space features

PipeWire sits between applications and kernel drivers.

Architecture:

**Application → PipeWire → ALSA → Hardware**

PipeWire interacts with ALSA in two ways:

1. Uses ALSA to access hardware
2. Provides virtual ALSA devices that redirect audio into PipeWire

ALSA remains essential because it provides kernel driver access.

#### Compatibility Layers

PipeWire includes compatibility components:

* `pipewire-pulse` → emulates PulseAudio
* `pipewire-jack` → emulates JACK
* ALSA plugins → redirect ALSA apps to PipeWire

This allows legacy applications to work without modification.

#### Why PipeWire Matters

PipeWire solves major historical problems:

* Unifies desktop and professional audio
* Improves Bluetooth support
* Reduces latency
* Simplifies routing
* Allows PulseAudio and JACK apps to coexist ([Linux Content][5])

---

## End-to-End Audio Flow Example

Let us trace what happens when you play music in VLC.

### Case A: Traditional PulseAudio System

1. VLC generates PCM audio samples.
2. VLC sends samples to PulseAudio.
3. PulseAudio mixes audio with other applications.
4. PulseAudio sends final stream to ALSA.
5. ALSA kernel drivers send data to sound card.
6. Sound card converts digital audio to analog output.

Flow:

**VLC → PulseAudio → ALSA → Kernel → Hardware**

### Case B: Modern PipeWire System

1. VLC sends audio using PulseAudio API.
2. `pipewire-pulse` receives the stream.
3. PipeWire builds a processing graph.
4. PipeWire sends audio to ALSA.
5. ALSA kernel drivers send data to hardware.

Flow:

**VLC → PipeWire → ALSA → Kernel → Hardware**

Even if VLC thinks it is talking to PulseAudio, it is actually talking to PipeWire.

### Case C: Direct ALSA Access (Rare but possible)

1. Application writes directly to ALSA device.
2. ALSA kernel drivers send audio to hardware.

Flow:

**App → ALSA → Kernel → Hardware**

Downside: no mixing or routing.

---

## Conceptual Model: Responsibilities by Layer

### Kernel Layer (ALSA drivers)

Responsibilities:

* Hardware communication
* DMA buffers
* Interrupt handling
* Device management

### ALSA User Space

Responsibilities:

* APIs for applications
* Basic mixing and plugins
* Device abstraction

### Sound Servers (PulseAudio / PipeWire / JACK)

Responsibilities:

* Mixing multiple streams
* Routing audio
* Resampling
* Policy management
* Session management
* Bluetooth and network audio

### Applications

Responsibilities:

* Generate or consume audio
* Use APIs (ALSA, PulseAudio, JACK, PipeWire)

---
