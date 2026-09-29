---
doc: scope
status: approved
---

# Wild Wolves Agenda Bridge

One line: A personal agenda assistant that turns scheduled events from the Wild Wolves Discord server into tentative Outlook calendar events.

## The Unique Kernel
It watches the Discord event scheduler for one specific server, rather than trying to understand every message. It turns an event that matters to the learner into a tentative Outlook item that the learner can accept or decline.

## Who It's For
The learner, who wants better control of a busy personal agenda. Today, Discord events and the personal calendar are separate, so relevant events can be missed or need to be copied manually.

## The Core Loop
The assistant checks scheduled events in the Wild Wolves server, identifies an event, and uses its title, date, time, description, and location to prepare an Outlook calendar item. The learner reviews the tentative item in Outlook and accepts or declines it.

## Inspiration & Identity
No existing app or tool inspired this project. It should feel practical, personal, and easy to understand: a small bridge between the learner's main Discord community and their agenda.

## Why This Matters to the Learner
The learner wants to get better control of their agenda while learning basic Python, how an AI coding assistant works, and how to think from a real problem toward a working program. The project connects to a Discord community the learner actively uses and keeps the first build grounded in a real need.

## What "Working" Looks Like
A scheduled event in the Wild Wolves Discord server is found and appears as a tentative event in the learner's Outlook calendar with a useful title, date, time, and relevant context. The learner can open it, accept or decline it, and see that the event is now represented in their personal agenda. The compelling moment is seeing a community event become a usable personal calendar item without copying it by hand.

## The POC Boundary
- Read scheduled events from the Wild Wolves Discord server only.
- Use the event's available title, date, time, description, and location.
- Create a tentative Outlook calendar event for review.
- Demonstrate the learner accepting or declining the proposed event.

## Later
- WhatsApp message scanning and sending an acceptance or decline reply back through WhatsApp.
- Email invitation handling.
- Support for additional Discord servers.
- More advanced interpretation of event context and conflict handling.
- Automatic reminders, recurring events, and a broader personal scheduling assistant.

## Explicitly Cut
- Scanning ordinary Discord messages: the Discord scheduled-event feature supplies a cleaner and more reliable source for the proof of concept.
- WhatsApp and email integrations: they add authentication and message-handling complexity before the central Discord-to-Outlook loop is proven.
- Multiple servers: limiting the build to Wild Wolves keeps the first experiment focused and relevant.
- A polished standalone interface: Outlook remains the place where the learner reviews and decides, so a separate dashboard is not needed to prove the idea.
