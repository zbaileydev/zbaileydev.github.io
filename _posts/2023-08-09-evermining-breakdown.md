---
layout: post
---

This is a breakdown of systems I designed for Evermining, a minining simulator submitted to "My First Game Jam: Summer 2022".

## Table of contents
- [Table of contents](#table-of-contents)
- [Plan](#plan)
- [Character Movement](#character)
- [Mining](#mining)
- [UI](#UI)


## [Plan](#plan)

Evermining was planned out as a 2D mining game that would follow the jam's theme of "healing" by letting you harvest gems which you could combine together for spells such as healing yourself or others. My development time was cut short by a week because I was sick so I pivoted to a MVP of having a mining loop where rocks spawn in and the player harvests them. The control set relies on the player using the WASD keys to move around the level and using the left mouse button to swing their pickaxe. 

Any gems will be automatically picked up if the player gets close enough to them and once they reach 20 of each type they will win the game. There is also a quit key "P" which allow the player to quit the game. 

## [Character Movement](#character)

For the character, I wanted something squishy that was fun to move around with, so I downloaded a sprite knight from [Kenney](https://kenney.nl/assets/tiny-dungeon) and placed it into my scene. After the initial controller was set up, I worked on creating a squishing effect when the player jumped and a slight tilt towards whichever direction they were running. We'll go over the key sections of the helper script called by our main movement controller.

```c#
  public GameObject playerSprite;
  float rotSpeed = 5f;
  bool shouldRotate = false;
  Transform originalStartRot;
  Transform originalEndRot;
  Vector3 m_from = new Vector3(0.0F, 0.0F, 0.0F);
  Vector3 m_to = new Vector3(0.0F, 0.0F, -10.0F);
  bool hammerRotated = false;
```
Our initial variables are what you might expect; a GameObject referencing our knight, a speed for how fast to rotate it, a bool to keep track of whether we are able to rotate, Transforms of the rotations start and end location, and 2 Vector3's which represent the Z direction that the knight tilts towards.


```c#
void Update(){
    if (shouldRotate) {SquishRun();}
    else {ResetRotation();}
} 

public void SquishRun(){
    Quaternion fromRot = Quaternion.Euler(m_from);
    Quaternion toRot = Quaternion.Euler(m_to);
    float lerp = 0.5f * (1.0f + Mathf.Sin(Mathf.PI * Time.realtimeSinceStartup * rotSpeed));
    playerSprite.transform.localRotation = Quaternion.Lerp(fromRot, toRot, lerp);
}

void ResetRotation(){
      playerSprite.transform.localRotation = Quaternion.identity;
  }
```
In the Update() call, we check if should be rotating our character. Since this is initialized to false, ResetRotation() is called to ensure we start off with no rotation. If we are able to rotate, SquishRun is called which transforms the V3 into euler angles and passes them into a lerp between 0 and -10 on the Z axis. The sprite then bounces back and forth between these positions as the player moves.


```c#
public void FlipPlayer(GameObject target, float x, float y){
    target.transform.localScale = new Vector2(x, y);
}

public void JumpPlayer(float x, float y){
    playerSprite.transform.localScale = new Vector2(x, y);
}
```
The fun part is how we handle the player turning. We just flip the player and all its children, meaning SquishRun does not need to be modifed. We also warp the scale of the player when they are jumped, creating a stretching effect along with the side-to-side bob.


```c#
public void RotateHammer(GameObject target, float direction){
    if (direction == 1f){
        target.transform.localRotation = Quaternion.identity;
        hammerRotated = false;
    }
    else if(!hammerRotated){
        target.transform.Rotate(0f, 0f, 80f, Space.World);
        hammerRotated = true;
    }
}

public void UpdateRotate(bool rot){
    shouldRotate = rot;
}

```
The final functions in our helper class rotate the mining hammer, which is called not for swinging the hammer, but for rotating the hammer when the sprite changes direction on the X axis so it is always facing the right direction. There is also a helper function to update our shouldRotate bool. 



## [Mining](#mining)

## [UI](#UI)

