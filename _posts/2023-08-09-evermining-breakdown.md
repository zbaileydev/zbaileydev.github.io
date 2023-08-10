---
layout: post
---

This is a breakdown of systems I designed for Evermining, a minining simulator submitted to "My First Game Jam: Summer 2022".

## [Contents](#contents)
- [Plan](#plan)
- [Player Movement](#player-movement)
- [Controller](#controller)
- [Mining](#mining)
- [UI](#ui)


## [Plan](#plan)

Evermining was planned out as a 2D mining game that would follow the jam's theme of "healing" by letting you harvest gems which you could combine together for spells such as healing yourself or others. My development time was cut short by a week because I was sick so I pivoted to a MVP of having a mining loop where rocks spawn in and the player harvests them. The control set relies on the player using the WASD keys to move around the level and using the left mouse button to swing their pickaxe. 

Any gems will be automatically picked up if the player gets close enough to them and once they reach 20 of each type they will win the game. There is also a quit key "P" which allow the player to quit the game. 

## [Player Movement](#player-movement)

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

[Back to Top](#contents)

## [Controller](#controller)

Now that we have covered our helper script to enhance the player's movement, let's explore how this is called by the actual controller. 

```c#
  public AudioSource steps;
  public AudioSource music;
  
  public float speed = 10f;
  public float jumpThrust = 15f;
  public GameObject hammer;
  
  bool isGrounded = true;
  bool stepSounds = false;
  
  PlayerGraphics playerGraphics;
  Rigidbody2D rb;

  void Start(){
      rb = GetComponent<Rigidbody2D>();
      playerGraphics = GetComponent<PlayerGraphics>();
      music.Play();
  }
```
Our variables are sound effects for steps and music, some public floats for the speed, a reference to the hammer, and bools for tracking our state. We also have a PlayerGraphics reference which connects our helper script to this one. The start function also does what you would expect; assigning our components to variables and starting the music. Ideally music would play from a GameManager, but this prototype does not have one.

```c#
 void Update(){
    if (Input.GetKeyDown(KeyCode.P)) Application.Quit();
    
    if (rb.velocity.y == 0){
        isGrounded = true;
        playerGraphics.JumpPlayer(1f, 1f);
    }

    if (Input.GetKeyDown(KeyCode.W) && isGrounded){
        float stetchHeight = 1.10f;
        float stetchWidth = 0.90f;
        playerGraphics.JumpPlayer(stetchWidth, stetchHeight);
        HandleJump();
        isGrounded = false;
    }
    MoveHorizontal();
  }

  void HandleJump(){
      rb.AddForce(transform.up * jumpThrust, ForceMode2D.Impulse);
  }

  void OnCollisionEnter2D(Collision2D other){
    if (other.gameObject.tag == "Floor" && !isGrounded) isGrounded = true;
    }
```
In our core game loop, the first condition we check for is if the P key is pressed leading the game to quit. We then check the RigidBody's velocity in the Y axis to see if we are jumping. If we are not in the air, we ensure our sprite's localScale is set to a float of 1. Otherwise if we are on the ground and press the W key, we then initiate a jump which stretches the sprite vertically. The jump just adds force in the Y direction.

Finally, just in case our knight is touching the floor but the isGrounded flag has not reset itself, we force the flag back to enable jumping. The next section is going to cover the MoveHorizontal() function.

```c#
void MoveHorizontal(){
    float inputX = Input.GetAxis("Horizontal");
    Vector3 movement = new Vector3(speed * inputX, 0, 0);
    movement *= Time.deltaTime;
    transform.Translate(movement);

    if (inputX != 0 && !stepSounds)
    {
        steps.Play();
        stepSounds = true;
    }

    if (inputX < 0){
        playerGraphics.FlipPlayer(this.gameObject, -1f, 1f);
        playerGraphics.FlipPlayer(hammer, -1f, 1f);
        playerGraphics.RotateHammer(hammer, -1f);
        playerGraphics.UpdateRotate(true);
    }
    else if (inputX > 0){
        playerGraphics.FlipPlayer(this.gameObject, 1f, 1f);
        playerGraphics.FlipPlayer(hammer, 1f, 1f);
        playerGraphics.RotateHammer(hammer, 1f);
        playerGraphics.UpdateRotate(true);
    }
    else if (inputX == 0){
        playerGraphics.UpdateRotate(false);
        steps.Stop();
        stepSounds = false;
    }
```
Each frame we grab our X-axis input and translate it into movement based on how long it has been since the last frame. SFX plays for the walk noise if stepSounds is false. The largest part of this function is calling functions to flip the player and the hammer they wield. If we are moving to the left, our inputX is going to be less than 0 which is what the first condition is checking for. We call FlipPlayer on the gameobject itself and the hammer, then we rotate the hammer and update the bool tracking that it has rotated. If we are not moving then the step SFX stops and we stop tilting the sprite. 

[Back to Top](#contents)

## [Mining](#mining)

[Back to Top](#contents)

## [UI](#ui)

[Back to Top](#contents)
