---
layout: post
author: Zach B
tags: [breakdown]
---

This short article is on relative /  [intra-package references in Python](https://docs.python.org/3/tutorial/modules.html#intra-package-references), based on a recent undertaking at my work place and the lack of documentation for my specific use case. 

### The Problem: Monolithic Project

I had a directory of several dozen scripts which called each other. The core developers knew how to extend this system but training new team members would be time consuming and the tech debt associated with this layout would continue to impede future features down the road. My objective was to organize the scripts into their respective components and folders, while enabling their functions to be imported across the project. 

### The Issue: Testing Imports

After reading stack overflow answers and documentation, none of the code to validate successful imports was working. An [example project](https://github.com/PuzzleZach/ImportTesting/) can be found on my GitHub. Within Test/test_builder.py we have a script to import our API builder and return if it is successful or not.

<details><summary>Test/test_builder.py</summary><p>

```python3
from ..lib.core.api_factory import api_builder

def test_builder():
  endpoint = api_builder("/get-health")
  known_body = {"days_interval": 3}
  return endpoint.body == known_body
```
            
</details>

When we run this with `python3 test_builder.py` an error appears claiming that there is no parent package. If we go up to the parent and run a driver function, we also get an error that we are importing beyond the top level package.

### The Solution: Test at the Top

When testing sibling directories or parents, Python cannot resolve the paths and will fail to find them. We have to run our code from outside of the project.

<details><summary>ImportTesting/TestDriver.py</summary><p>
  
```python3
from .Test.test_auth import test_auth
from .Test.test_builder import test_builder

# Would use unittest in practice but this is faster.
if __name__ == "__main__":
  auth = test_auth()
  build = test_builder()
  print(f"The result of our auth test is {auth} and the result for our build test is {build}")
```

</details>

This was an easy problem to solve after some thought and now we have neatly organized and decoupled libraries and components to build off of in the future. Added benefits include training new developers on single components first, making onboarding easier for them and allowing them to better keep track of dependencies. 

To run the example, we go to the directory above our project (in this case GitHub) and run `python3 -m ImportTesting.TestDriver` so that Python is able to resolve the imports successfully across the entire project.
