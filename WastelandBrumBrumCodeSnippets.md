# Wasteland Brum Brum Code Snippet(s)


## Making the player car controller
I learned via a tutorial about this topic that you can make an object behave like a car if you apply a Sphere Collider and let this interact with the world. For the rotating on steep inclines, the car uses a single raycast to detect how steep the object is and changes the pitch of the car based on that value.

 **CarControllerScript.cs**
 
 ```cs
 public class CarControllerScript : MonoBehaviour
{
	  [Header("References")]
    public Rigidbody m_sphereBody;

	  [Header("Movement")]
    public float maxSpeed = 130;           //How fast the car goes in kmp
    public float reverseSpeed = 60;           //How fast the car goes backwards in kmp

    public float m_accelBoost = 3;          // If the speed < 0 we want to get back to driving forward really quick
    public float m_maxForwardAccel = 1;     // Forward acceleration speed
    public float m_reverseAccel = 4f;       // Backward acceler](url)ation speed

    public float m_accel;

    public float m_rollOffStrength = 3f;                 // How quick does the car lose speed when only rolling

    [Tooltip("Rotation Speed of the car")]
    [SerializeField] private float m_mobility = 60f;      // Rotation speed of the car
    [Space(8, order = 1)]
    public float m_minMobility = 30f;                    // Minimal Rotation speed of the car
    public float m_maxMobility = 90f;                    // Maximum Rotation speed of the car

    public float m_speedInput;
    private float m_rotationInput;

		public float m_groundDrag = 3f;                 // Drag when the car is on the ground
    public float m_airDrag = 0.3f;                  // Drag when the car is in the air

		[Header("Ground Check")]
    public float m_gravityForce = 10f;              // The gravity force to the car
    public float m_checkGroundRayLength = 0.5f;     // Length of the ground Ray
    public Transform m_groundRayPos;                // Position of the ground Ray detection
    public LayerMask m_groundLayer;                 // What layer is considered ground
    public bool m_isGrounded;                       // Is the car grounded

    private void Start()
    {
        m_sphereBody.transform.parent = null;       // Disconnect the Sphere Collider from the Car 
    }

		private void Update()
    {
	GetForwardInput(Input.GetAxis("Vertical"));                             // Forward speed of the car uses the Input.GetAxis(Vertical)
        m_accel = m_speedInput < 0 ? m_accelBoost : m_maxForwardAccel;          // Change accel based on the current SpeedInput of the car 

        GetSidewaysInput(Input.GetAxis("Horizontal"));                          // Horizontal speed (AKA turning) uses the Input.GetAxis(Horizontal)

	// When the car is grounded it is allowed to turn and rotate
        if (IsCarGrounded())
        {
            m_rotationInput = Input.GetAxis("Horizontal");

            if (m_speedInput > 0)
            {
                transform.rotation = Quaternion.Euler(transform.rotation.eulerAngles + new Vector3(0f, m_rotationInput * m_mobility * Time.deltaTime * 1, 0f));
            }
            else if (m_speedInput < 0)
            {
                transform.rotation = Quaternion.Euler(transform.rotation.eulerAngles + new Vector3(0f, m_rotationInput * m_mobility * Time.deltaTime * -1, 0f));
            }
				}


	// Move the position of the Player to the body of the Sphere it is supposed to follow.
        transform.position = new Vector3(m_sphereBody.transform.position.x, m_sphereBody.transform.position.y + m_offsetToCenterSphere, m_sphereBody.transform.position.z);
		}
		
		// GetForwardInput(float verticalAxisDirection) will receive the Input.GetAxis("Vertical") to perform the forward movement for the car
		private void GetForwardInput(float verticalAxisDirection)
    {
        if (verticalAxisDirection > 0)
        {
            m_speedInput += verticalAxisDirection * m_accel * 100f;
        }
        else if (verticalAxisDirection < 0)
        {
            m_accel = m_reverseAccel;
            m_speedInput += verticalAxisDirection * m_accel * 100f;
        }
        else
        {
            m_speedInput = Mathf.Lerp(m_speedInput, 0f, Time.deltaTime * m_rollOffStrength);

            if (m_speedInput < 250)
            {
                m_speedInput = 0;
            }

            m_mobility = Mathf.Lerp(m_mobility, m_minMobility, Time.deltaTime * 5);

            if (m_mobility < m_minMobility + 1)
            {
                m_mobility = m_minMobility;
            }
        }

        m_speedInput = Mathf.Clamp(m_speedInput, (-reverseSpeed * 67.629750f) / 1.03f, (maxSpeed * 67.629750f) / 1.03f);                  // Clamp the speed  //1kmp 67,629750
    }		
		
		// GetSidewaysInput(float horizontalAxisDirection) will receive the Input.GetAxis("Horizontal") to perform the steering movement for the car
    private void GetSidewaysInput(float horizontalAxisDirection)
    {
        if (horizontalAxisDirection > 0)
        {
            m_mobility += horizontalAxisDirection * 2;
        }
        else if (horizontalAxisDirection < 0)
        {
            m_mobility -= horizontalAxisDirection * 2;
        }
        else
        {
            m_mobility = m_minMobility;
        }
		}

		private void FixedUpdate()
    {
				// If the car is on the ground it needs to receive a different amount of drag and you can drive
        if (IsCarGrounded())
        {
            m_sphereBody.drag = m_groundDrag;

            if (Mathf.Abs(m_speedInput) > 0)
            {
                m_sphereBody.AddForce(transform.forward * m_speedInput);
            }
        }
        else
        {
            m_sphereBody.drag = m_airDrag;
            m_sphereBody.AddForce(Vector3.up * -m_gravityForce * 100);
        }
		}

		// IsCarGrounded() will check if the groundRay is hitting a ground layer, if true return a true for m_isGrounded.
    // Else the car needs to stablize itself back to a "Nose Dive" rotation. 
    private bool IsCarGrounded()
    {
        m_isGrounded = false;

        RaycastHit hit;

        if (Physics.Raycast(m_groundRayPos.position, -transform.up, out hit, m_checkGroundRayLength, m_groundLayer))
        {
            m_isGrounded = true;
            transform.rotation = Quaternion.FromToRotation(transform.up, hit.normal) * transform.rotation;  // Set the rotation of the car to be that of the rotation of the face
        }
        else
        {
						// If the car is in the air, move to a "nose dive" rotation point
            transform.rotation = Quaternion.Slerp(transform.rotation, Quaternion.Euler(20, transform.eulerAngles.y, transform.eulerAngles.z), Time.deltaTime * m_restoreRotationSpeed);
        }

        return m_isGrounded;
    }
}
```
**Result:**

![ezgif-2-06511263e5](https://user-images.githubusercontent.com/78432932/161429042-4fa56a14-9dc5-40f3-8010-2b7887339bc2.gif)
#

## Car banking visual
The car also received a way to show banking when the player wanted to go left and right, this had also affect on the wheel facing direction

**CarControllerScript.cs**

```cs
// If the car is drifting turn the frontwheels
            if (!m_isDrifting)
            {
                m_frontWheelLeft.localEulerAngles = new Vector3(0, (Input.GetAxis("Horizontal") * m_minWheelRotation + 180), 0);
                m_frontWheelRight.localEulerAngles = new Vector3(0, (Input.GetAxis("Horizontal") * m_minWheelRotation) + 180, 0);

                m_carBodyModel.localEulerAngles = new Vector3(0, m_carBodyModel.localEulerAngles.y, Input.GetAxis("Horizontal") * m_minBanking);
            }
            else
            {
                m_frontWheelLeft.localEulerAngles = new Vector3(0, (Input.GetAxis("Horizontal") * m_maxWheelRotation) + 180, 0);
                m_frontWheelRight.localEulerAngles = new Vector3(0, (Input.GetAxis("Horizontal") * m_maxWheelRotation) + 180, 0);

                m_carBodyModel.localEulerAngles = new Vector3(0, m_carBodyModel.localEulerAngles.y, Input.GetAxis("Horizontal") * m_maxBanking);
            }
```
**Result:**

![ezgif-5-2295d3606c](https://user-images.githubusercontent.com/78432932/161429431-bcec42d5-37c0-4344-a85e-50df366afe83.gif)
#
