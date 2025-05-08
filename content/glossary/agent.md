+++
title = "Agent"

[glossaryEntry.agent]
name = "agent"
symbol = ""
usage = "Arvo"
desc = "A userspace application managed by Gall."

+++

**Agents** are the main kind of userspace application on Urbit. They have a persistent state and API that handles events and produces effects. Agents are managed by the [Gall](/glossary/gall) [vane](/glossary/vane) (kernel module). Agents are sometimes just called "apps", though that is a little ambiguous as a single app installed in Landscape may have multiple agents working together in the background.

Gall agents can variously be treated as databases with developer-defined logic, services, daemons, or a kind of state machine.

### Further Reading

- [App School](/courses/app-school): A comprehensive guide to writing Gall agents.
