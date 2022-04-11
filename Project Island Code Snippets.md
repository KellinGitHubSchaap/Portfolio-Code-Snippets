# Project Island Code Snippet(s)


## Camera transitioning
The camera transitions are like the 2D Legend of Zelda games. The code is currently connected to a collider that get triggered when the player moves out of it. When this trigger occurs the position of the player is captured and from there it determines in what direction the camera needs to move.


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

## NPC interaction
Since this is an adventure game, I added in a dialogue system that gets activated when the player wants to talk with an NPC 

**PlayerInteractionScript.gd**

```py
extends Node2D

onready var m_interactionCast = get_node("RayCast2D") # The RayCast node object that will allow us to detect interactable objects
var m_interactionRayLength = 8 # Length of the RayCast
var m_playerFacingDir # The current direction the player is facing

var m_object # The object the player wants to interact with

export (NodePath) var m_dialoguePath # The path for dialogue that needs to be displayed on screen
onready var m_dialogueBoxScript = get_node(m_dialoguePath) # The dialogue that needs to be displayed on screen

onready var m_cameraScript = get_node("../../../Camera2D") # Get the script of the camera

func _ready():
	pass

func _process(delta):
	m_interactionCast.cast_to = GetPlayerFacingDir() # Shoot the ray to the point the player is currently facing
	
	if(m_interactionCast.is_colliding()):
		var collidedWith = m_interactionCast.get_collider() # What is the object we collided with
		
		# If the interaction is with a NPC > and the player presses ENTER > Go talk to that NPC
		if(get_tree().get_nodes_in_group("NPC").has(collidedWith) && Input.is_action_just_pressed("ui_accept")):
			PrepareDialogueScript(collidedWith)

func PrepareDialogueScript(var collidedWith):
	m_dialogueBoxScript.SetDialoguePath(collidedWith.m_dialogueSoundPitch, collidedWith.m_dialoguePath) # Handover the dialogue file that the NPC is holding
	GameManager.m_isInDialogue = true
	m_dialogueBoxScript.visible = true # Turn on the dialogue box
	m_dialogueBoxScript.ProcessInteractionInput() # See this as an Interaction input

# GetPlayerFacingDir will figure out in which direction the player is currently facing and also applies the Length
func GetPlayerFacingDir():
	m_playerFacingDir = Vector2(get_parent().m_moveDir.x, get_parent().m_moveDir.y) * m_interactionRayLength
	return m_playerFacingDir
```

**NPCDialogueHolder.gd**

```py
extends StaticBody2D

export var m_dialoguePath = ""         	       # Path to find the dialogue file of this NPC
export (float) var m_dialogueSoundPitch = 1    # Pitch of the beeping sound when talking to this character

```

**DialogueScript.gd**

```py
extends TextureRect

export var m_dialoguePath = ""         # File path to find the dialogue needed, given upon interacting with an NPC
export (float) var m_textSpeed = 0.03  # To determine how fast text is written to the dialogue box 

var m_dialogue    # Array to be filled with the dialogue

var m_phraseNum = 0   # Curren phrase the text is displaying
var m_finished = true   # Is the dialogue between characters done

onready var m_audioPlayer = get_node("Text/Timer/AudioStreamPlayer")	# Audio node that gets triggered each time a letter appears on screen

func _ready():
	pass

func ProcessInteractionInput():
	if(m_finished):
		m_dialogue = GetDialogue()
		assert(m_dialogue, "Dialogue not found")
		$Text/Timer.wait_time = m_textSpeed
	
		NextPhrase()
	else:
		$Text.visible_characters = len($Text.text)

# Get the NPC's dialogue file and turn this into a readable format
func GetDialogue() -> Array:
	var file = File.new()
	assert(file.file_exists(m_dialoguePath), "File path doesn't exist")

	file.open(m_dialoguePath, File.READ)
	var json = file.get_as_text()

	var output = parse_json(json)
	
	if (typeof(output) == TYPE_ARRAY):
		return output
	else:
		return []

# Get the next line in the current used file and make the dialogue box empty so that it can restart the process of writting everything down
func NextPhrase():
	if(m_phraseNum >= len(m_dialogue)):
		GameManager.m_isInDialogue = false
#		print(m_gameRef.m_isInDialogue)
		visible = false
		m_phraseNum = 0
		return

	m_finished = false
	
	$Text.bbcode_text = m_dialogue[m_phraseNum]["Text"]
	
	$Text.visible_characters = 0
	
	while( $Text.visible_characters < len($Text.text)):
		$Text.visible_characters += 1
		
		m_audioPlayer.play()
		$Text/Timer.start()
		yield($Text/Timer, "timeout")
		
	
	m_finished = true
	m_phraseNum += 1
	return

# This function gets triggered by PlayerInteractionScript.gd
func SetDialoguePath(var pitch, var diaPath = ""):
	m_dialoguePath = diaPath
	m_audioPlayer.pitch_scale = pitch
```

**Result:**

![ezgif-3-5ff4025792](https://user-images.githubusercontent.com/78432932/162701714-4da38ac6-215f-428a-91c0-1f5e90fd821a.gif)

#
