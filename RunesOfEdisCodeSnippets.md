# Runes Of Edis Code Snippets

## Code snippet(s) on the combo streak increase animation. When this effect is activated it will stimulate the player into continuing chaining combos, thereby getting a higher score outcome once the player stops making them.

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

#

##
