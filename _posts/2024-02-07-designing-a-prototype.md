---
layout: post
author: Zach B
tags: [design]
---

The new year is off to a fantastic start, I am 90% through CS50G and have begun working on my final project. Coming up with an idea was easy, but it was a challenge to determine how it would actually integrate with gameplay. 

The concept: **Pokemon with Cameras**. 

I have started getting into photography and wanted to create a role-playing game where you gain powerful cameras to battle enemies and take photos. For this prototypes we will have the following game objects in Unity:

1. A Player that has a stats system, can engage in combat with a Camera item, and has a UI representation for their health and experience.
2. An Enemy that has multiple states so variants can be passive or aggressive, but will attack the Player.
3. A Camera that has stats such as shots, time between shots, if it can flash, etc.
4. A GameManager that handles pausing the game.
5. An options menu that persists in the start screen and pause screen.

### Creating a Camera

This was challenging process because I had to balance the effectiveness of various techniques for catching the area in front of the player against what I could easily implement. I decided to create a custom mesh collider that mimicked the polygon used for a camera and attached it to the camera body. That way when the player tried to trigger a photo, it could check to see if any Enemy objects were inside of it and deal damage to them.
