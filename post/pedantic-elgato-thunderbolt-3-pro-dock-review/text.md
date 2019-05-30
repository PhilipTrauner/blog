# Pedantic Review: Elgato Thunderbolt 3 Pro Dock

<p align="center">
	<img src="dock.png" width="500px" style="margin-top: 50px;"/>
</p>
<figcaption>Elgato Thunderbolt 3 Pro Dock</figcaption>

> Every once in a while, a revolutionary product comes along, that rids its users of some annoying little time-waste that stands between them and productivity.
>
> — Steve Jobs (possibly altered to fit the narrative of this review)

In my mind, Thunderbolt 3 docks are one such kind of product. Plug in *one* cable, and immediately pick up where you left off... that's the concept, at least.

Let's see if the [Elgato Thunderbolt 3 Pro Dock](https://www.elgato.com/de/dock/thunderbolt-3-pro) delivers on that promise.


## Design
The dock looks and feels premium. It has some heft to it (it weighs roughly half a kilogram), so it won't easily slide off ones desk because of stiff cabling and such. The  aluminum chassis is aesthetically pleasing and the whole device looks *very* similar to the back of a Mac Mini (which is good).

## I/0
* Back
	* Gigabit Ethernet
	* 3.5mm headphone jack (line out)
	* 2 x USB 3.1 Gen 2 Type C connector (10GB/s; 1.5A)
	* 2 x Thunderbolt 3
	* DisplayPort 1.2

<p align="center">
	<img src="back.png" width="500px"/>
</p>
<figcaption>Ports on the back of the dock</figcaption>

* Front
	* SD card reader
	* Micro SD card reader
	* 3.5mm headphone jack (line in / out)
	* 2 x USB 3.1 Gen 2 Type A connector (5GB/s; 1.5A)

The port selection is overall solid, but a second DP port certainly wouldn't have hurt, as there's only 2 Thunderbolt 3 ports, one of which will always be occupied by the connected computer.

## Price
All Thunderbolt 3 docks are pretty expensive, and this one is no exception coming in at around <b style="color: #B12704;">350 USD / EUR</b>, but it still manages to be the most expensive out of the bunch. This can largely be attributed to the fact that Elgato has a very specific target audience in mind: **Apple customers**.

That's not just an assumption though, as basically all their marketing material only depicts Apple hardware, and the fact that they do not provide a Windows version of their *Thunderbolt Dock Utility*.

## Software
The Dock Utility provides two optional pieces of functionality:

1. High-power USB support
2. An eject all menu bar application (for storage devices attached to the dock)

High-power USB support is achieved through two kernel modules (`ElgatoThunderbolt2DockChargingSupport.kext`, `ElgatoThunderboltDockChargingSupport.kext`) which basically tell `IOKit` that 15 Watts are available instead of 5.

A third module is thrown into the mix (`ElgatoThunderboltDockAudioRename.kext`) that updates the internal name of the USB audio devices that the dock exposes (to `Elgato Thunderbolt Dock Audio` / `Elgato Thunderbolt 2 Dock Audio`).

If neither feature is desired there is no point in installing the Dock Utility, as blindly installing kernel extensions of dubious quality is a great way to cause system instability.

## Audio quality
> No setup is complete without epic sound. Thanks to a maximum sample rate of 96 kHz, and sample size of 24 bits, [...] you can plug in your high-fidelity headset without disconnecting your desktop speakers.
>
> — Elgato marketing blurb

**Fecal matter of male cattle**. *Both* 3.5mm socket emit a quiet but still incredibly annoying static hiss. The fact that an Apple USB-C to 3.5mm headphone jack adapter produces better results is honestly staggering.
That being said, its only really problematic for headphones, as the hiss practically isn't noticeable on most speakers, ... which doesn't change the fact that such poor audio quality is simply *not* acceptable.

# Conclusion
Its alright, but certainly not worth the asking price. I won't be returning my unit, as its basically the only good-looking Thunderbolt 3 dock out there, and because a USB-C to 3.5mm headphone jack adapter sufficiently solves the audio quality issues.

**tldr**: If design is the deciding factor, go with this one. If it isn't then the [CalDigit TS3 Plus](https://www.caldigit.com/ts3-plus) is probably a better investment.
