+++
title = "Behn"

[extra]
category = "arvo"

[glossaryEntry.behn]
name = "behn"
symbol = ""
usage = "arvo"
desc = "Timing vane of Arvo. Allows for applications to schedule events."

+++

**Behn** is the timing [vane](/glossary/vane) (kernel module). It allows for applications to schedule events, which are managed in a simple priority queue. For example, [Clay](/glossary/clay), the Urbit filesystem, uses Behn to keep track of time-specific file requests. [Eyre](/glossary/eyre), the Urbit web vane, uses Behn for timing-out HTTP sessions.

Behn is located in `/base/sys/vane/behn.hoon` within [Arvo](/glossary/arvo).

### Further Reading

- [The Behn reference](/system/kernel/behn): A technical guide to the Behn vane.
