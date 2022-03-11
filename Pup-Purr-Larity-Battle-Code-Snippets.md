# Pup-purr-larity Battle Code Snippet(s) 
With the code snippet(s) presented here I will only show the ones necessary to perform the actions this to prevent overcomplicating.
<br>
<br>
## Code snippet(s) on the rotational movement of a paper-like character, inspiration from the paper mario series. This code was written inside of a Coroutine so that it would only be played when necessary.

**CatMovementScript.cs**

```cs
public class CatMovementScript : Monobehaviour
{
	// Flip the sprite of the body
	private IEnumerator FlipBodyAnimation()
	{
  		m_direction = m_isFlipped ? 1 : -1; // Based your walk direction based on if you are flipped or not

  		m_currentFlipTimer = 0; // Reset the timer
		// If the debugger doesn't want a constant time till for the animation, the boolean , m_randomFlipTime needs to be ticked
  		m_pickedTime = m_randomFlipTime ? Random.Range(m_timeTillFlip.x, m_timeTillFlip.y) : m_pickedTime = m_timeTillFlip.x;

  		float newRotation = m_isFlipped ? 0 : 180;
  		Quaternion targetRotation = Quaternion.Euler(0, newRotation, 0);

  		m_isFlipped = !m_isFlipped;
				
		// Flip the sprite
  		while (m_rotationTimer < 0.99)
 		{
    			transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, m_rotationTimer);
    			m_rotationTimer += Time.deltaTime * m_currentRotationSpeed; // * speed gain || / speed lowered

    			yield return new WaitForEndOfFrame();
   		}

   		transform.rotation = targetRotation;

   		m_rotationTimer = 0;
   		m_canFlip = true;
	}
}
```
**Result:**

![ezgif-1-d049e6a1e6](https://user-images.githubusercontent.com/78432932/157834693-00760167-04a4-4b90-b303-9230b005c450.gif)
#

## One of the mechanics of the game was fixing broken machines, here are code snippets on how a broken machine is found by the Cat when the player clicks on it and also how it becomes fixed again.

**WorldScript.cs**

```cs
public class WorldScript : MonoBehaviour
{
	public static WorldScript instance;
    private void Awake()
    {
      	if (instance == null)
       	{
       		instance = this;
       	}
       	else if (instance != this)
       	{
       		Destroy(this);
       	}
    }

	[Header("Machines Settings")]
    public MachineScript[] m_machineScripts;	// There are 3 machines that can become broken at some point 
    public float m_TimeTillNextIssue = 20f;		// Time it takes before a new issue appears
    public float m_currentTimeTillIssue;		// Current time that counts up to the moment for a new issue

	private void Update()
    {
       	CreateIssue();
    }

    public void CreateIssue()
    {
       	m_currentTimeTillIssue += Time.deltaTime;

       	if (m_currentTimeTillIssue > m_TimeTillNextIssue)	// When this is true..
      	{
       		int value = Random.Range(0, m_machineScripts.Length);	// .. pick a random machine..
            
       		if (!m_machineScripts[value].m_isFixed)		// .. and trigger its BreakDown() function.
       		{
           		m_machineScripts[value].BreakDown();
           		var type = m_machineScripts[value].m_type;	
           		MonitorManager.Instance.HandleBreakdown(type);	// Send the information about a broken machine to the monitors of the Dog
       		}

            m_currentTimeTillIssue = 0;
            m_TimeTillNextIssue = Random.Range(20f, 30f);	// Choose a new random time to cause another issue to occur
        }
    }
}
```

**MachineScript.cs**

```cs
public enum MachineType {Oil, Systems, Oxygen}	// Machine types
public class MachineScript : MonoBehaviour
{
    public MachineType m_type;				// The machine type of this object
    public CatMovementScript m_movementScript;
    public bool m_isFixed = false;			// Is this machine fixed by the cat?
    public bool m_isGettingFixed = false;	// Is this machine currently being fixed by the cat?

    public GameObject m_exclamationPoint;	// Indicator to show this machine is broken
	
	[Header("Systems Type : Settings")]
    public Sprite m_blueScreenOfDeath;		
    public Sprite m_regularScreen;
    public GameObject m_computerSprite;		// Sprite of the game object that needs to change

    private void OnMouseDown()
    {
		// If the machine is broken and the player clicks on it, send this script to the Cat
        if (!m_isFixed && m_movementScript.m_isFocused && m_exclamationPoint.activeInHierarchy && !m_isGettingFixed)
        {
            Debug.Log("Cat goes fixing");
            m_isGettingFixed = true;
            m_movementScript.MoveToBrokenMachine(this);
        }
    }
    
	// If the cat has reached the machine start the Coroutine to fix this machine 
    public void StartFixing()
    {
        StartCoroutine(FixMachine());
    }
	
	// Reference from the WorldScript.cs to let a breakdown of this machine occur
    public void BreakDown()
    {
        if (this.m_type == MachineType.Systems)
        {
            m_computerSprite.GetComponent<SpriteRenderer>().sprite = m_blueScreenOfDeath;
        }

        m_isFixed = false;
        m_exclamationPoint.SetActive(true);
        tag = "Needs Fixing";
    }
	
	// Wait a fixed amount of time and tell the Cat when this script is done being broken
    private IEnumerator FixMachine()
    {
        yield return new WaitForSeconds(3);
        tag = "Fixed";

        m_isFixed = true;
        m_isGettingFixed = false;

        m_movementScript.m_catState = CatMovementScript.CatState.Wandering;

        m_exclamationPoint.SetActive(false);

        if (m_isTheComputer)
        {
            m_computerSprite.GetComponent<SpriteRenderer>().sprite = m_regularScreen;
        }

        MonitorManager.Instance.HandleFix(m_type);
        
		// Apply a simple cool-down so that the WorldScript.cs doesn't break this machine again
		yield return new WaitForSeconds(2);
        m_isFixed = false;
    }
}
```

**CatMovementScript.cs**

```cs
public class CatMovementScript : MonoBehaviour
{
	public enum CatState { Wandering, GoingToFixSpot, Fixing, Distracted, IsPlaying }
	public CatState m_catState;     // State of the cat
	public bool m_isFocused = true;             // Check to see if body is still focused
	
	public MachineScript m_machineTarget;  // Machine Target for the Cat to walk to
	
	void Update()
    {
        switch (m_catState)
        {
            case CatState.Wandering:
                if (m_animator.GetInteger("catAnimState") != 0)
                {
                    m_animator.SetInteger("catAnimState", 0);
                }
                WanderingState();
                break;
            case CatState.GoingToFixSpot:
                if (m_animator.GetInteger("catAnimState") != 0)
                {
                    m_animator.SetInteger("catAnimState", 0);
                }
                GoingToFixSpotState();
                break;
            case CatState.Fixing:
                if (m_animator.GetInteger("catAnimState") != 2)
                {
                    m_animator.SetInteger("catAnimState", 2);
                }
                FixingState();
                break;
        }
    }
	
	// Move the body of the cat based on direction and speed
    private void MoveBody()
    {
        Vector3 currentPosition = transform.position;

        currentPosition.x += m_movementSpeed * m_direction * Time.deltaTime;

        transform.position = currentPosition;
    }
	
	private void WanderingState()
    {
        MoveBody();
        m_currentFlipTimer += Time.deltaTime;

        if (WorldScript.instance.m_exclamationPoint.activeInHierarchy)
        {
            WorldScript.instance.m_exclamationPoint.SetActive(false);
        }

        if (m_canFlip)
        {
            if (m_currentFlipTimer > m_pickedTime || transform.position.x < m_borderPoints[0].position.x || transform.position.x > m_borderPoints[1].position.x)
            {
                m_canFlip = false;
                StartCoroutine(FlipBodyAnimation());
                Debug.Log("FLIP!");
            }
        }
    }

    // Move the body to the machine that needs fixing
    private void GoingToFixSpotState()
    {
        MoveBody();
        m_currentFlipTimer = 0;

        if (Vector3.Distance(transform.position, m_machineTarget.transform.position) < 2.5f)
        {
            m_catState = CatState.Fixing;
            m_machineTarget.StartFixing();
            Debug.Log("LESS THAN 2.5");
        }
    }

    // Set the target position to be that of the givenMachine that needs fixing + Change cat state
    public void MoveToBrokenMachine(MachineScript givenMachine)
    {
        m_catState = CatState.GoingToFixSpot;

        m_machineTarget = givenMachine;
        m_currentRotationSpeed = 1.2f;

        if (transform.position.x > m_machineTarget.transform.position.x & !m_isFlipped || transform.position.x < m_machineTarget.transform.position.x & m_isFlipped)
        {
            m_canFlip = false;
            StartCoroutine(FlipBodyAnimation());
        }

        m_currentRotationSpeed = m_setRotationSpeed;
    }

    // If the Cat is fixing a machine 
    private void FixingState()
    {
        m_currentFlipTimer = 0;

        if (m_machineTarget != null)
        {
            m_machineTarget = null;
        }
    }
}
```
**Result:**

![ezgif-2-0561b16f3c](https://user-images.githubusercontent.com/78432932/157848732-2d7c371f-0f2b-4d6e-a6d0-5f77202fd8e5.gif)
