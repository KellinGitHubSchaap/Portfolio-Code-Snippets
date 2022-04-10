# Project Island Code Snippet(s)

## Code snippet(s) on the camera transitions like the 2D Legend of Zelda games. The code is currently connected to a collider that get triggered when the player moves out of it. When this trigger occurs the position of the player is captured and from there it determines in what direction the camera needs to move.


**CameraMovementScript.gd**

```py
extends Camera2D

# New position of the Camera2D
var m_newPosition = Vector2()

onready var m_player = get_node("../Sprites/Player") # Get the player Sprite

export (int) var m_maxTransitionSpeed = 2 # How fast can the camera travel between screen points
var m_transitionSpeedX  # Speed on the X axis 
var m_transitionSpeedY  # Speed on the Y axis

func _ready():
  print(m_projectResolution)
  pass

func _process(_delta):
	TransitionCameraToPoint()

# Keep Transitioning the camera until its position is the same as the new position 
func TransitionCameraToPoint():
	if(GameManager.m_isTransitioning && position != m_newPosition):
		position.x += m_transitionSpeedX
		position.y += m_transitionSpeedY
	else:
		position = m_newPosition
		GameManager.m_isTransitioning = false
	
# If the player moves out of the Camera sight signal a transition
func _on_Player_exited(_body):
	var playerPosition = get_parent().get_node("Sprites/Player").get_global_transform_with_canvas().get_origin() # What is the position of the player based on screen canvas

# Check the direction of the players exit of screen
	if(!GameManager.m_isTransitioning):
		if(playerPosition.x < 4):
			SetCameraTransition(Vector2(position.x - GameManager.m_projectResolution.x, position.y), Vector2(-m_maxTransitionSpeed, 0), Vector2(-8, 0))
		elif(playerPosition.x > GameManager.m_projectResolution.x - 4):
			SetCameraTransition(Vector2(position.x + GameManager.m_projectResolution.x, position.y), Vector2(m_maxTransitionSpeed, 0), Vector2(8, 0))
		elif(playerPosition.y < 4):
			SetCameraTransition(Vector2(position.x, position.y - GameManager.m_projectResolution.y), Vector2(0, -m_maxTransitionSpeed), Vector2(0, -8))
		elif(playerPosition.y > GameManager.m_projectResolution.y - 4):
			SetCameraTransition(Vector2(position.x, position.y + GameManager.m_projectResolution.y), Vector2(0, m_maxTransitionSpeed), Vector2(0, 8))

# Give the camera a new position to focus on if the player left the screen, positon is given in the On_Player_Exited(body):
func SetCameraTransition(var newPos, var transitionSpeed, var newPlayerPos):
	GameManager.m_isTransitioning = true
	m_newPosition = newPos
	m_transitionSpeedX = transitionSpeed.x
	m_transitionSpeedY = transitionSpeed.y
	
	m_player.position.x += newPlayerPos.x 
	m_player.position.y += newPlayerPos.y

```

**Result:**

![ezgif-2-e021d5f9a1](https://user-images.githubusercontent.com/78432932/162635038-24f67cf0-b501-4f78-8645-b427d734a397.gif)

#

## New mechanic

etc
