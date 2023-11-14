---
layout: post
author: Zach B
tags: [jam, breakdown]
---

This short article is on relative /  [intra-package references in Python](https://docs.python.org/3/tutorial/modules.html#intra-package-references), based on a recent undertaking at my work place and the lack of documentation for my specific use case. 

### The Problem: Monolithic Project

I had a directory of several dozen scripts which called each other. The core developers knew how to extend this system but training new team members would be time consuming and the tech debt associated with this layout would continue to impede future features down the road. My objective was to organize the scripts into their respective components and folders, while enabling their functions to be imported across the project. 

### The Issue: Testing Imports

After reading stack overflow answers and documentation, none of the code to validate successful imports was working. An example directory format is below:
<details><summary>Project Directory</summary><p>

  ```
├───component

|        __init__.py

│       HealthSystem.py

│       SlackSystem.py

│

└───lib

  ├───auth
  
  |       __init__.py
  
  │       auth_system.py
  
  │
  
  ├───core
    
  |      __init__.py
  
  |      core_test.py
    
  |      api_factory.py
```            
</details>

Inside of core_test.py

```python3
from ..auth.auth_system import authenticate

def test():
  request = authenticate()
  if request:
    print("Authentication works")

if "__name__" == "__main__":
  test()

```

An error appears claiming that there is no parent package.

### The Solution: Test at the Top

When testing sibling directories or parents, Python cannot resolve the paths and will fail to find them. I moved my test cases to their own directory and wrote a driver at the very top of my project letting me reach into any folder and run them as needed. 

An example of how I ran the tests from the root (top most level) of the project:

```python3
from tests.testauth import testauth

if "__name__" == "__main__":
  testauth()
```

This was an easy problem to solve after some thought and now we have neatly organized and decoupled libraries and components to build off of in the future. Added benefits include training new developers on single components first, making onboarding easier for them and allowing them to better keep track of dependencies. 
