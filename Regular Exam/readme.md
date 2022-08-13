# Regular exam - Unity 3D Basics (april 2021) @ softuni.bg

## Tasks

1. MenuScene -> Make **PlayButton** to able to load **RacingScene** *(MenuSceneManager.cs)*  

2. Make car to move with **WASD** or arrow keys and use **Speed** property and Input.GetAxis() to control car speed *(CarScript.cs)*  
    - Use **Rigidbody.position** to change the position of the car.

3. Make it that way, so when car hits other car (or truck) decrease property of **CarScript.Health** with 1, and when car hits **HealthItem** increase it with 1 *(CarScript.cs)*

4. Configure **HealthSlider** bar to reflect property of **CarScript.Health**  *(CarScript.cs)*

5. Make **HealthItem** prefab to rotate around it axis, when it is active on the road. - *(SelfRotateScript.cs)*
    - Use **_rotateSpeed** property.

6. Make that way that gameobjects on the road not to be instantiated, but taken from pool itself,by writing the logic in GetRoadItemFromPool method and then you use the method at same script.  *(RoadScript.cs)*

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

- In Windows Explorer/MacOSX Finder open Assets folder from the project and **compress/archive** these folders ->  **Scenes + Scripts**.
- Upload archive in **.zip/.rar/.7zp** format.

## Solutions

### Task 1

Modify ``LoadRacingScene()`` methon in **MenuSceneManager.cs** like this:

``` csharp
...

public void LoadRacingScene()
{
    SceneManager.LoadScene("RacingScene");
}

...
```

after that you have to attach **MenuSceneManager** from hierarchy to ``OnClick`` event of Canvas > **PlayButton** object and select ``LoadRacingScene`` method as callback.

![task1](https://raw.githubusercontent.com/tmollov/softuni-unity-basic-exam-solutions/main/Regular%20Exam/readme_resources/task1-1.png)

### Task 2

First create this method in **CarScript.cs**:

``` csharp
void CarControls()
{
    Vector3 input = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical"));
    _rigidbody.MovePosition(transform.position + input * Speed);
}
```

Then call it on the ``Update()`` method in else block like this:

```csharp
void Update()
{
    if (Health <= 0)
    {
        // GameOver
        RoadScript.RoadSpeed = 0f;
        GameOverMenu.SetActive(true);
    }
    else
    {
        CarControls(); // Calling our controls
        _rigidbody.position = new Vector3(_rigidbody.position.x, _rigidbody.position.y,
            Mathf.Clamp(_rigidbody.position.z, MinZ, MaxZ));
    }
}
```

### Task 3 & 4

There is ``OnTriggerEnter()`` method in **CarScript.cs**.  
We will use it to detect collisions and update health bar slider.

The code looks like this:

```csharp
private void OnTriggerEnter(Collider coll)
{
    // Detect hit with either tag "Health" or tag "Enemy"
    if (coll.CompareTag("Health"))
        Health += 1;

    if (coll.CompareTag("Enemy"))
        Health -= 1;

    // Update health bar slider
    Healthbar.value = Health;
}
```

### Task 5

We are using ``Update()`` method to rotate health items around their axis.

```csharp
void Update()
{
    transform.Rotate(Vector3.up * (Time.deltaTime * _rotateSpeed));
}
```

### Task 6

Our task here is to get one non-active item from the pool:

```csharp
private GameObject GetRoadItemFromPool(int type)
{
    // Using the passed `type` get one free (non active) item from the pool and return it
    for (int i = 0; i < PoolSize; i++)
    {
        var item = _roadItemsPool[type, i];
        if (!item.activeSelf)
        {
            return item;
        }
    }
    return null;
}
```

then we need to modify ``AddNewRoadItem()`` method and use **GetRoadItemFromPool()** in it:

```csharp
private void AddNewRoadItem()
{
    var allRoadItemTypes = AllRoadTypesPrefabs.Length;
    var roadItemType = Random.Range(0, allRoadItemTypes);

    // Instead of Instantiating new item we are using GetRoadItemFromPool() method to get one
    var newRoadItem = GetRoadItemFromPool(roadItemType);
    if (newRoadItem is null)
        return;

    newRoadItem.gameObject.SetActive(true);

    var xPosition = Random.Range(_xPositionMinMax.x, _xPositionMinMax.y);
    var zPosition = Random.Range(_zPositionMinMax.x, _zPositionMinMax.y);

    newRoadItem.transform.position = new Vector3(xPosition, 0f, zPosition);
}
```
