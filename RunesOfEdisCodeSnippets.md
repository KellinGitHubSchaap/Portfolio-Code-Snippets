# Runes Of Edis Code Snippets

Code for the "combo streak increase" animation. When this effect is activated it will show the player the current collected points and the combo streak, this effect continues as long as the player is chaining combos.

**HudManager.cs**
```cs
[Header("Combo Score Settings")]
public GameObject currentComboObject;           // Object in the hierarchy for the combo HUD element
public TextMeshProUGUI currentComboScoreText;   // The text that is displayed for the current streak the player has
private int tempComboScore = 0;                 // The current streak of combos created because of a long drift/score session
public TextMeshProUGUI comboHighScoreText;      // The text that is displayed for the combo streak
private int currentComboHighScore = 5;          // A highscore of a streak the player can beat when he/she is drifting about 

private void Start()
{
  StartHUDScreen();
}

// Set all value once the game has started
private void StartHUDScreen()
{
  comboHighScoreText.text = string.Format("Highest Combo: {0}X", currentComboHighScore);

  currentComboObject.SetActive(false);
  currentComboScoreText.text = string.Format("COMBO {0}X", tempComboScore);
}

// UpdateComboScore() will update the HUD element of the combo scores
public void UpdateComboScore()
{
  currentComboObject.SetActive(true);
  tempComboScore++;

  currentComboScoreText.text = string.Format("{0}X", tempComboScore);
	if (tempComboScore > currentComboHighScore)
	{
		  currentComboHighScore = tempComboScore;
		  comboHighScoreText.text = string.Format("Highest Combo: {0}X", currentComboHighScore);
	}
  StartCoroutine(ComboAnim());
}

// ResetComboScore() will be called when the player crashes into a wall
public void ResetComboScore()
{
  tempComboScore = 0;
  tempScore = 0;
  currentComboObject.SetActive(false);
  StopCoroutine("ComboAnim");
}

private int previousValue; // Hold the old value of the previous round, this will stop the While loop for a short period

private IEnumerator ComboAnim()   // ComboAnim() is a code based animation that will show the combo streak shrink if the player doesn't perform a new combo 
{
	int newValue = previousValue = Random.Range(0, 100000); // Generate a random value between to large points
	float timer = 0;

	Vector2 scale = Vector2.one; // Reset the scale size of the Object back to 1,1 on XY-axis

	while (timer < 1 && previousValue == newValue)
	{
		if (timer > 0.8f)
		{
			currentComboObject.SetActive(false);
			UpdateBankScore(tempScore * tempComboScore);

			tempComboScore = 0;
			tempScore = 0;

			yield return null;
		}

	float s = Mathf.Lerp(1, 0, timer);
	scale.x = s;
	scale.y = s;

	currentComboObject.transform.localScale = scale;

	timer += Time.deltaTime * 0.3f; // << SET Animtion Speed here 
	yield return new WaitForSeconds(0);
	}
}
```

**Result:**

![ezgif-2-cf95d5392e](https://user-images.githubusercontent.com/78432932/162633026-c6bbad91-17dc-4e0a-889d-42ba4dac89a6.gif)

#

For the sound manager I only needed to know how big the list of playable songs was and also let a randomizer pick a song to play out of the list

**SoundManager.cs**

```cs
[Header("Music Settings")]
public SongDataSO[] songData;	// Songs that can be played
public AudioClip currentSong;	// Current Song that is selected
public AudioSource musicOutput;	// Output source for the music

private void Start()
{
	StartCoroutine(UpdateSong());
}

private void Update()
{
	// Check if there is a song playing, if not choose a new song
  	if (!musicOutput.isPlaying)
  	{
	  	StartCoroutine(UpdateSong());
	}
}

// UpdateSong() is used by the game to change a song to something from a set Array
public IEnumerator UpdateSong()
{
	if (songData != null) // Get a new song from the List this mustn't be a song that is currently played and update the whole songNameText
  	{
	  	Debug.Log("Song has been updated");
    		int randomValue = Random.Range(0, songData.Length);

    		while (songData[randomValue].song == currentSong) // As long as the current picked song is the same as the only that just ended, pick a new one
    		{
	    		// If the randomValue is the same as the last songs position, the function will be restarted
      			randomValue = Random.Range(0, songData.Length);
    		}

    		currentSong = songData[randomValue].song;
    		HUDManager.instance.UpdateSongHUD();	// Update the HUD element that shows what song is currently being played

    		if (!musicOutput.isPlaying)
    		{
	    		musicOutput.clip = currentSong;
      			musicOutput.PlayOneShot(currentSong);
    		}
  	}

	yield return null;
 }
```

**HudManager.cs**
```cs
[Header("Song Settings")]
public TextMeshProUGUI currentSongText;
public Animator songTextAnim;

	// UpdateSongHUD() will recieve a song from the SoundManager which tells it to show the Player what kind of song is currently being played
	public void UpdateSongHUD()
	{
		songTextAnim.SetTrigger("GoTransition");
		currentSongText.text = string.Format("{0}", SoundManager.instance.currentSong.name);
	}
```

**Result:**

![ezgif-2-f76f30f6b9](https://user-images.githubusercontent.com/78432932/162633647-4b6d6ddf-2734-4b96-828f-126bbafd4848.gif)

#
