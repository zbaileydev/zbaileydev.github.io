---
layout: post
author: Zach B
tags: [jam, breakdown]
---

This is going to be a short reflection and breakdown of the systems I used in a narrative adventure submitted to  Brackey's Game Jam 2022.1. The original 1-line concept for this game was:

> What if I combine fake doors with a fire-escape game? 

The player would have to find out how to escape a building that was on fire but some of the doors were not able to be opened, led to dead-ends, or secret passages could open to escape a level. This was planned to be an isometric style game, but for the sake of prototyping it was pushed to 1st person. 

## Character Controller

How should the player interact with the world? First they need to walk around in it. One major mistake I made was an artifact of using 1st person mode. I did not have any meshes for the player and therefore did not notice the players RigidBody kept felling to the ground and colliding with random objects or doorways, which took several hours of debugging to figure out why collisions would not work as intended. I froze the position and rotation of the RB and shrunk the player object's radius to ensure collisions only happened when they touched doors or other objects. 

## Core Loop

The player would start in a small office area and turn around to search for the exit, following visual cues. The character's internal thoughts would trigger at various moments along with the fire system, which appeared to "follow" the character around. 

The entire "fire system" comprises of the following code. 

```c#
public class FireSpawner : MonoBehaviour{
    public float TimeToSpawn = 10f;
    
    void Start(){
        Invoke("StartFire", TimeToSpawn);
    }

    void StartFire(){
        GameObject child = gameObject.transform.GetChild(0).gameObject;
        child.SetActive(true);
    }
}
```

A public float is modified as various FireSpawners are created and placed around the map. When the level starts, after enough time has passed the StartFire function will turn on a child object which has a tag and fire particles. When laid out, this gives the impression of a spreading fire but it is tedious setting up each one. It would be more beneficial to draw a path and have the fire move along the path incrementaly rather than jumping entire blocks likethe manual implementation. 


```c#
public class DoorCollision : MonoBehaviour{
    public GameObject door;
    Renderer doorRenderer;
    Collider doorCol;

    void Start(){
        doorRenderer = GetComponent<Renderer>();
        doorCol = GetComponent<Collider>();
    }

    void OnCollisionEnter(Collision col){
        if (col.gameObject.CompareTag("Player")){
            Color doorColor = doorRenderer.material.color;
            doorColor.a = 0.8f;
            doorRenderer.material.SetColor("_Color", doorColor);
            doorCol.enabled = false;
        }
    }
}
```

The door collision system is also incredibly basic. It holds a reference to itself, which is never called so this was an oversight. We pull its Renderer and Collider components and then on collisions check for if the player touched the door. If this condition is true, the renderer changes the alpha color of the door's material to make it transparent and the collider is disabled so the player can walk through it. A better system would be to generate a prompt in the player controller asking them to open a door when they step in front of one and are looking at it.

## Putting it Togethers

```c#
public Canvas gameMessage;
public GameObject WinText;
public GameObject LoseText;

void OnCollisionEnter(Collision col){
    if (col.gameObject.CompareTag("ExitDoor")){
        Debug.Log("End of Level!");
        CheckLevel();
    }
    else if (col.gameObject.CompareTag("Fire")){
        Debug.Log("You are on fire! Game over!");
        Invoke("FireText", 2);
        Invoke("GameOver", 5);
    }
}

void CheckLevel(){
    int nextScene = SceneManager.GetActiveScene().buildIndex + 1;

    // Verify we have another scene, minus 2 since we have a 'win' scene
    if (SceneManager.sceneCountInBuildSettings - 2 >= nextScene) SceneManager.LoadScene(nextScene);
    else{
        // Loads the last scene, which is our win scene
        SceneManager.LoadScene(SceneManager.sceneCountInBuildSettings - 1);   
    }
}

void GameOver(){
    SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
}

void FireText(){
    gameMessage.gameObject.SetActive(true);
    LoseText.SetActive(true);
}

public void OpenMainMenu(){
    UnityEngine.SceneManagement.SceneManager.LoadScene(0);
}

public void QuitGame(){
    Application.Quit();
}
```

A GameManager script is placed on the character and helps them interact with the world by tracking if they got hit by fire or exited the level. It also handles level loading, displays UI text, and quits the game for the player. Ideally all of the collision checking should be done inside of a player-centric script instead of being thrown in with other functionality that calls multiple objects. For a short game jam submission it worked fine and the end experience was intriguing.  

