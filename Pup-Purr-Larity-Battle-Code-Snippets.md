# Pup-purr-larity Battle Code Snippet(s)
 

**Code snippet on the rotational movement of a paper-like character, inspiration from the paper mario series. This code was written inside of a Coroutine so that it would only be played when necessary**
```cs
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
```
**Result:**

![ezgif-1-d049e6a1e6](https://user-images.githubusercontent.com/78432932/157834693-00760167-04a4-4b90-b303-9230b005c450.gif)
