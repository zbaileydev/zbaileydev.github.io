---
layout: post
author: Zach B
tags: [tools]
---

Hey everyone! Today I want to walk you all through how I built a tool to assist me with texturing assets from free marketplaces like Quaternius. Often when these models are imported, they are missing the material necessary to add color to them. Of course, the content creators provide an Atlas for you to apply to your models but it can be painstaking to attach them to an entire folder.

That is where the AtlasApply tool comes in. 

```c#
public Material newAtlasMaterial;
public string projectName;
public string modelFolder;
public string fileExtension = "*.fbx";
```

We have 4 public variables to assign in the game object we apply this script to:
- The atlas we want to map to each object model.
- The name of the project we are working inside of.
- The path to the folder within the project (“Assets”, “Models”)
- The extension of our models (obj, fbx, etc)

Using a ContextMenu attribute, we tell Unity to expose this function when the user clicks on the gear icon of the component. 

```c#
string folderPath = Path.Combine(Application.dataPath, modelFolder);
string[] modelPaths = Directory.GetFiles(folderPath, fileExtension);
```

With these 2 lines we build a path to our model folder and use the Directory class from System.IO to get all files in that folder that match our extension. This gets us all of the FBX files for example.

```c#
string newPath = modelPath.Split(projectName)[1].Replace("\\", "/").Substring(1);
GameObject model =  AssetDatabase.LoadAssetAtPath<GameObject>(newPath);       	 
if (model == null) continue;
Renderer[] renderers = model.GetComponentsInChildren<Renderer>();
```

For each path to a model, we convert the backslashes to forward slashes in order to load in the asset through Unity’s AssetDatabase, which requires forward slashes. If we cannot load the model, we continue through the loop. If we successfully load the model, we grab all the Renderers from it.

```c#
Material[] materials = renderer.sharedMaterials;
for (int i = 0; i < materials.Length; i++) {
if (materials[i].name == "Atlas") {
materials[i] = newAtlasMaterial;   
            }
}
renderer.materials = materials;
```

Then for each Renderer we will grab its shared materials and loop over them, searching for whichever one is named “Atlas”. When we find it, we set its value to the new atlas material we had selected previously. We then set the renderer’s materials to the array, whose elements we just modified.

```c#
 AssetDatabase.Refresh();
```

Once all of our loops are finished, we refresh the asset database to apply these changes. This was a fun tool to work on and you can find the [code on my GitHub](https://github.com/PuzzleZach/UnityTools/blob/main/AtlasApply.cs).
