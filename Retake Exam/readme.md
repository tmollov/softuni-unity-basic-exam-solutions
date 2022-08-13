# Regular exam - Unity 3D Basics (april 2021) @ softuni.bg

## Tasks

1. MenuScene -> Configure buttons to work on click so: **( MenuSceneScript.cs )**
    - PlayButton must load MainScene
    - ExitButton must exit game

    All remaining task are for ``MainScene``  

2. When user clicks left mouse button ``PlayerShipScript.Fire()`` should be called. **( PlayerShipScript.cs )**

3. There is a metod ``BuildingScript.OnTriggerEnter(Collider)`` - find out why it is not working despite it is added to prefabs.   Fix them for collision detection to work. Prefabs are called -> `"b1; b2; b3; b6; b8; b10"`

4. Make it so that when a building is blown up by a missile strike
``PlayerShipScript.Score`` should increase with 1. **( BuildingScript.cs )**

5. Make ``ScoreSlider`` bar to reflect the value of ``PlayerShip.Score`` - **( PlayerShipScript.cs )**

6. Correct the method ``PlayerShipScript.Fire`` to launch a rocket only if the mouse is clicked  
on a building and aim the missile at the building
as soon as you instantiate it.

    Tips:  
        - Input.mouseposition  
        - Physics.Raycast  
        - gameObject.tag  
        - Transform.LookAt  

Maximum points : 100

Points by task  
task 1 - 5 points  
task 2 - 10 points  
task 3 - 15 points  
task 4 - 20 points  
task 5 - 25 points  
task 6 - 25 points  

If a given task is partially completed, a part of the points will be assigned to it.  
Required points for assessment (5/5+ out of 6) -> 70 or more.  

Hint: Before you start, look at the scenes carefully. What GameObjects are already there, what scripts are already added and see all scripts from Scripts folder + all prefabs in Prefabs folder from the project. Also pay attention to the prefab tags.

When you are ready:

You have to upload only the solutions files - without whole project.  
This goes like:

- In Windows Explorer/MacOSX Finder open Assets folder from the project and **compress/archive** these folders ->  **Prefabs + Scenes + Scripts**.
- Upload archive in **.zip/.rar/.7zp** format.

## Solutions

### Task 1

Attach **MenuSceneManager** from hierarchy to ``OnClick`` event of Canvas > **PlayButton** object and select ``OnLoadGameScene`` method as callback.
![task1](https://raw.githubusercontent.com/tmollov/softuni-unity-basic-exam-solutions/main/Retake%20Exam/readme-resources/task-1.png)

Repeat same for ``ExitButton``, but use ``OnExitGame`` method instead.

### Task 2

Add following code in ``Update`` method of **PlayerShipScript.cs**:

```csharp
...
void Update()
{
    ...
    
    
    if (Input.GetKeyDown(KeyCode.Mouse0))
        Fire();
    
}
... 
```

### Task 3 & 4

The reason `OnTriggerEnter` is not working is that they need a `RigidBody` in order to detect collisions.

Select all `b1; b2; b3; b6; b8; b10` prefabs and add new component RigidBody to them.  
Uncheck `Use Gravity` and check `Is Kinematic`.

![task3](https://raw.githubusercontent.com/tmollov/softuni-unity-basic-exam-solutions/main/Retake%20Exam/readme-resources/task-3.png)

### Task 5

Add this method to `PlayerShipScript` and call it from `Update` method:

```csharp
...

public void RefreshScore()
{
    scoreBar.value = Score;
}

void Update()
{
    ...

    if (Input.GetKeyDown(KeyCode.Mouse0))
        Fire();

    RefreshScore();
}

...
```

### Task 6

Correct `PlayerShipScript.Fire` method like this:

```csharp
...

private void Fire()
{
    var camera = Camera.main;
    var mousePos = Input.mousePosition;
    mousePos.z = camera.transform.position.z;
    var ray = camera.ScreenPointToRay(mousePos);
    if (Physics.Raycast(ray, out var hit))
    {
        if (hit.transform.CompareTag("Building"))
        {
            var launchPosition = RocketLaunchPositionGO.transform;
            var rocket = Instantiate(RocketPrefab, launchPosition.position, launchPosition.rotation);
            rocket.transform.LookAt(hit.transform);
        }
    }
}

...
```

## Bug fix

You might saw that explosions are instatiated but never destroyed.  
In order to stop this behavior add this logic to `Explode` method in `BuildingScript`:

```csharp
private void Explode()
{
    var position = transform.position;
    var explosion = Instantiate(ExplosionPrefab, position, Quaternion.identity);
    explosion.SetActive(true);
    var player = GameObject.FindGameObjectWithTag("Player");
    var stress = Vector3.Distance(player.transform.position, position) / 20;
    stress = 1 - Mathf.Clamp01(stress);
    Camera.main.GetComponent<StressReceiver>().InduceStress(stress);

    gameObject.SetActive(false);
    Destroy(explosion, 2f); // Destroying explosion after 2 seconds
}
```
