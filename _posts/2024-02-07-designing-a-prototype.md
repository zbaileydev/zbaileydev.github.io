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

1. Create a Render Texture 
2. Update a Target Texture of a 2nd camera to that Render Texture
3. Set the Render Texture as the Albedo of a material
4. Throw that material on a quad and you have a camera you can move around (security cameras, monitors, the list goes on)

### Detecting Enemies

The quickest way to simulate taking a photo is to turn on a mesh collider that is the same shape as the camera view. I scrapped one together in Unity using ProBuilder to make a polygon that does the job well enough to capture any enemies that are on the camera screen.

In order to ensure that the player is able to detect enemies, I use a coroutine that enables the meshCollider and pauses for a second.

```
IEnumerator WaitForShot()
{
    meshCollider.enabled = true;
    yield return new WaitForSeconds(1f);
    meshCollider.enabled = false;
}
```

This gives the camera enough time for its `OnTriggerEnter` event to run and detect an enemy, allowing the player to access functions like dealing damage and gaining experience. 

One notable downside is this goes through walls, so I am in the process of adding raycasting to the trigger so that Enemy objects are only assigned if they are in the open.

```
public bool CheckForWall(Transform enemyTransform)
    {
        RaycastHit hit;

        if(Physics.Raycast(cameraCheck.position, transform.up, out hit, 30.0f))
        {
            if (hit.collider.gameObject.tag == "Wall")
            {
                float wallDistance = Vector3.Distance(cameraCheck.position, hit.point);
                float enemyDistance = Vector3.Distance(cameraCheck.position, enemyTransform.position);
                // Now that we have the enemy distance and wall distance, check which is closer.
                // If the enemy is closer than the wall, this will return true.
                return enemyDistance < wallDistance;
            }
        }
        // If we do not find a wall, assume true.
        return true;
```

This function will only run if we capture an enemy in our MeshCollider. We launch a ray from just outside our camera (a transform called `cameraCheck` handily helps us orient ourself) and check if we hit any walls. Then we compare the distance between the wall and the found enemy to see which is closer. There are edge cases where this might not work, but as long as the player centers their camera on an enemy peeking out from behind a wall they should be able to engage them reliably. 

### Enemy AI

For this prototype, I want to create 2 enemy types: a patroling guard and an aggressive enemy. More details coming soon!

