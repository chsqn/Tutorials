# 2) Add a Character and Movement Mechanics TODO

Add a character to the scene.  Have him walk and jump, creating a basic platformer. TODO

TODO gif, demo build

## Configure the character sprite sheet

Add a sprite sheet for the character, slice it with a bottom pivot and set filter mode to point.  We are using [Kenney.nl's Platformer Characters](http://kenney.nl/assets/platformer-characters-1) PNG/Adventurer/adventurer_tilesheet.png.


<details><summary>How</summary>

 - Drag/drop the sprite sheet into Assets/Art.
 - Set 'Sprite Mode: Multiple'.
 - Click 'Sprite Editor':
   - Cell Count: 9 rows 3 columns
   - Pivot: Bottom
 - Set the 'Filter Mode: Point (no filter)'.

<img src="http://i.imgur.com/BuIsVWD.png" width=204 />

Note we won't be tiling the character sprite, so the default of Mesh Type: Tight is okay.

</details>
<details><summary>What's Pivot do?</summary>

A pivot point is the main anchor point for the sprite.  By default, pivot points are at the center of the sprite.  

For the character, we are moving the pivot point to the 'Bottom'.  This allows us to position and rotate the character starting at the feet.  

Here's an example showing a character with a default 'Center' pivot and one with the recommended 'Bottom' pivot.  They both have the same Y position.  Notice the the vertical position of each character as well as how the rotation centers around the different pivot points:

<img src="http://i.imgur.com/AQY4FOT.gif" width=320 />

The pivot point you select is going to impact how we create animations and implement movement mechanics.  The significance of this topic should become more clear later in the tutorial.

</details>



## Add Character to the Scene with a Walk Animation

Create a Character GameObject with a walk animation. Change Order in Layer to 2.  We are using adventurer_tilesheet_9 and adventurer_tilesheet_10.

<details><summary>How</summary>

 - Hold Ctrl and select 'adventurer_tilesheet_9' and 'adventurer_tilesheet_10' sprites from the sprite sheet 'adventurer_tilesheet'.
 - Drag them into the Hierarchy.
 - When prompted, save the animation as Assets/Animations/CharacterWalk.anim.
 - In the Inspector, set the SpriteRenderer's 'Order in Layer' to 2.
 - Rename the GameObject to "Character" (optional).

<img src="http://i.imgur.com/k7bSlCp.gif" width=50% />
 

This simple process created:
 - The character's GameObject.
 - A SpriteRenderer component on the GameObject defaulting to the first selected sprite.
 - An Animation representing those 2 sprites changing over time.
 - An Animator Controller for the character with a default state for the Walk animation.
 - An Animator component on the GameObject configured for the Animator Controller just created.

Click Play to test - your character should be walking (in place)! 

<img src="http://i.imgur.com/2bkJdtS.gif" width=100px />

<hr></details>
<details><summary>What's the difference between Animation and Animator?</summary>

An animat**ion** is a collection of sprites on a timeline, creating an animated effect similiar to a flip book.  Animations can also include transform changes, fire events for scripts to react to, etc to create any number of effects.

An animat**or** controls which animations should be played at any given time.  An animator uses an animator controller which is a state machine used to select animations.

A state machine is a common pattern in development where logic is split across several states.  The state machine selects one primary state which owns the experience until the state machine transitions to another state.  Each animator state has an associated animation to play.  When you transition from one state to another, Unity switches from one animation to the next.  

We will be diving into more detail about animations and animators later in the tutorial.  

<hr></details>


## Add a rigidbody and CapsuleCollider2D

Add Rigidbody2D and CapsuleCollider2D components to the character to enable gravity.  Adjust the collider size as needed.

<details><summary>How</summary>

 - Select the Character's GameObject.
 - In the Inspector, click Add Component and select 'Rigidbody2D'.
 - Click Add Component and select 'CapsuleCollider2D'.
 - Click Edit Collider in the Inspector, under the CapsuleCollider2D component you just added and adjust to fit the character.
   - Click and then hold Alt while adjusting the sides to pull both sides in evenly.

<img src="http://i.imgur.com/KFwBZeo.gif" width=100px />

Hit play and the character should now land on a platform... but might fall over:

<img src="http://i.imgur.com/T0fdwa1.gif" width=150px />

</details>

<details><summary>How do I know what size to make the collider?</summary>

The collider does not fit the character perfectly, and that's okay.  In order for the game to feel fair for the player we should lean in their favor.  When designing colliders for the character and enemies, we may prefer to make the colliders a little smaller than the sprite so that there are no collisions in game which may leave the player feeling cheated.

As the character animates, its limbs may be in different positions.  The collider won't always fit the character and for that reason we use a collider focused around the body.

In addition to killing the character when he comes in contact with an enemy, the collider is used to keep the character on top of platforms.  For this reason it's important that the bottom of the collider aligns with the sprite's feet.

</details>
<details><summary>Why not use a collider that outlines the character?</summary>

Bottom line, it's not worth the trouble.  Unity does not provide good tools for more accurate collisions on animating sprites.  Implementing this requires a lot of considerations and may be difficult to debug.

Most of the time the collisions in the game would not have been any different if more detailed colliders were used.  Typically 2D games use an approach similiar to what this tutorial recommends. It creates a good game feel and the simplifications taken have become industry standard.

</details>




## Freeze rotation

Freeze the character's rotation so he doesn't fall over.

Note: The character will stand straight up even on slanted platforms.  This will be addressed later in the tutorial.

<details><summary>How</summary>

 - Select the character.
 - In the Rigidbody2D component, expand 'Constraints'.
 - Check 'Freeze Rotation'.

<img src="http://i.imgur.com/uXxDSwD.png" width=128px />

</details>
<details><summary>Does freezing rotation mean the rotation can never change?</summary>

No.  Adding constraints to the rigidbody only limits the Unity physics engine. Freezing the rigidbody position or rotation means that even if you got hit by a bus, you would not move or rotate.  However you could have a custom component set the position or rotation at any time.

Later in the tutorial we will be writing a script to rotate entities so that they align with platforms (i.e. their feet sit flat on the floor).

We use constraints to remove capabilities from Unity, allowing us more control where we need it.  Specifically here that means our character is not going to ever fall flat on his face.

</details>


## Move left/right

Create a WalkMovement script to control the rigidbody and a  PlayerController script to feed user input to the WalkMovement component.

Note the character will always be looking right, even while walking left.  He can also walk off the screen and push the balls around.  This will all be addressed later in the tutorial.

<details><summary>How</summary>

 - Create a C# script "WalkMovement" under Assets/Code/Components/Movement.
 - Select the Character GameObject and add the WalkMovement component.
 - Paste in the following code:
 
```csharp
using UnityEngine;
using System;

/// <summary>
/// Controls the entity's walk movement.
/// 
/// Another component drives walk via desiredWalkDirection.
/// </summary>
[RequireComponent(typeof(Rigidbody2D))]
public class WalkMovement : MonoBehaviour
{
  /// <summary>
  /// Set by another component to inform this component 
  /// it should walk. Positive to walk right, negative to 
  /// walk left.  The magnitude is the walk speed.
  /// </summary>
  [NonSerialized]
  public float desiredWalkDirection;

  /// <summary>
  /// A multiple to increase/decrease how quickly
  /// the object walks.
  /// </summary>
  [SerializeField]
  float walkSpeed = 100;

  /// <summary>
  /// Used to control movement.
  /// </summary>
  Rigidbody2D myBody;
  
  /// <summary>
  /// A Unity event, called once before this GameObject
  /// is spawned in the world.
  /// </summary>
  protected void Awake()
  {
    myBody = GetComponent<Rigidbody2D>();

    Debug.Assert(myBody != null);
  }

  /// <summary>
  /// A Unity event, called every x ms of game time.
  /// 
  /// Adds velocity to the rigidbody to move it horizontally.
  /// </summary>
  /// <remarks>
  /// With this approach, forces may not be used to impact the 
  /// X on this entity.  E.g. if we wanted a fan which slowly 
  /// pushed characters to the left, the force added would be 
  /// lost here.  This matches the Unity example character asset 
  /// as enabling forces on both dimentions cause movement to 
  /// feel strange or leads to other experience problems which 
  /// quickly complicate the code (but possible of course, 
  /// just thing things through).
  /// </remarks>
  protected void FixedUpdate()
  {
    // Calculate the desired horizontal movement given 
    // the input desiredWalkDirection.
    float desiredXVelocity 
      = desiredWalkDirection * walkSpeed * Time.fixedDeltaTime;

    // Any y velocity is preserved, this allows gravity
    // to continue working.
    myBody.velocity = new Vector2(desiredXVelocity, myBody.velocity.y);
  }
}
```

 - Create a C# script "PlayerController" under Assets/Code/Components/Movement.
 - Select the Character GameObject and add the PlayerController component.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Wires up user input, allowing the user to 
/// control the player in game with a keyboard.
/// </summary>
[RequireComponent(typeof(WalkMovement))]
public class PlayerController : MonoBehaviour
{
  /// <summary>
  /// Used to cause the object to walk.
  /// </summary>
  WalkMovement walkMovement;

  /// <summary>
  /// A Unity event, called once before the GameObject
  /// is instantiated.
  /// </summary>
  protected void Awake()
  {
    walkMovement = GetComponent<WalkMovement>();
    
    Debug.Assert(walkMovement != null);
  }

  /// <summary>
  /// A Unity event, called every x ms of game time.
  /// 
  /// Consider moving.
  /// </summary>
  /// <remarks>
  /// Moving uses an input state, and therefore may be captured 
  /// on Update or FixedUpdate, we use FixedUpdate since physics 
  /// also runs on FixedUpdate, so trying to do this on update would
  /// require an extra cache (w/o benefit).
  /// </remarks>
  protected void FixedUpdate()
  {
    // Consider moving left/right based off keyboard input.
    walkMovement.desiredWalkDirection 
      = Input.GetAxis("Horizontal");
  }
}
```

The character should walk around, but there is clearly work to be done:

<img src="http://i.imgur.com/xOpivgJ.gif" />

</details>
<details><summary>What is an Input 'Axis' and how are they configured?</summary>

Unity offers several ways of detecting keyboard/mouse/controller input.  'Axis' is the recommended approach.  Each input Axis may be configured in the inspector:

 - Edit -> Project Settings -> Input.
 - In the 'Inspector', you will find a list of supported input types.

<img src="http://i.imgur.com/T2BJvBm.png" width=100px />

You can add, remove, rename, and configure the inputs for your game.  Inputs may also be reconfigured by the player at runtime.  For more information about the various options, see [Unity's description of the InputManager](https://docs.unity3d.com/Manual/class-InputManager.html).  We will be using the defaults for this tutorial.

To read / detect Input, Unity offers a few APIs including:

 - GetAxis: Gets the current state as a float.  E.g. horizontal may return 1 for right, -1 for left.  
 - GetButtonDown/GetButtonUp: Determines if a button was pressed or released this frame.
 - GetMouseButtonDown/GetMouseButtonUp: Same as above, but for mouse buttons.

There are a ton of options, check out the [complete list of Input APIs](https://docs.unity3d.com/ScriptReference/Input.html).

</details>

<details><summary>Why not use a bool or Enum for Left/Right instead of a float?</summary>

You could for the game we are making at the moment.  When playing with a keyboard, a button is down or it isn't.  

The nice thing about using a float here is it could be leveraged to allow players even more control over movement.  When playing with a controller, left and right are not simply on and off - the amount you move the joystick  by scales how quickly the character should walk.

The WalkMovement desiredWalkDirection should be set to something in the range of -1 to 1, where 1 represents the desire to walk at full speed towards the right.  From there the WalkMovement component will apply the walkSpeed, representing the fastest speed the entity should walk, and then update the rigidbody.

</details>
<details><summary>Why FixedUpdate instead of Update?</summary>

Update occurs once per rendered frame.  FixedUpdate occurs at a regular interval, every x ms of game time.  FixedUpdate may run 0 or more times each frame.

FixedUpdate is preferred for mechanics which require some level of consistency or apply changes incrementally.  Physics in Unity are processed in FixedUpdated.  So when manipulating physics for the game such as we are here by changing velocity on the rigidbody, we do this on FixedUpdate to match Unity's expectatations. 


</details>
<details><summary>Why multiple by Time.fixedDeltaTime?</summary>

It's optional. Anytime you make a change which includes some speed, such as walking, we multiply by the time elapsed so motion is smooth even when the frame rate may not be.  While using FixedUpdate, the time passed between calls is always the same - so Time.fixedDeltaTime is essentially a constant.  

If speed is being processed in an Update, you must multiply by Time.deltaTime for a smooth experience.  While in FixedUpdate, you could opt to not use Time.fixedDeltaTime, however leaving it out may lead to some confusion as fields which are configured for FixedUpdate may have a different order of magnitude than fields configured for use in Update.

Additionaly you may choose to adjust the time interval between FixedUpdate calls while optimizing your game.  By consistently multiplying by the delta time, you can adjust the interval for FixedUpdate without changing the game play.

</details>
<details><summary>Why set velocity instead of using AddForce?</summary>

AddForce is a way of impacting a rigidbody's velocity indirectly.  Anytime you interact with either AddForce or velocity, a similar mechanic could be made using the other.

Generally the game feel when using AddForce has more gradual changes and for many experiences that's great.  Although there are lots of options for tuning the forces experience, velocity simply gives you more direct control.

So that's to say you could use AddForce here instead.  Maybe give it a try and see how it feels.  We select velocity because we want the controls for moving left and right to feel crisp.  Later in the tutorial we will use AddForce, for the jump effect.

</details>
<details><summary>Why not combine these into a single class?</summary>

As discussed in chapter 1, Unity encourages a component based solution.  This means that we attempt to make each component focused on a single mechanic or feature.  Doing so simplifies debugging and enables reuse.  For example, we will be creating another enemy type which will use the same WalkMovement component created for the character above.

</details>


## Create a pattern for death effects

Create a pattern to use instead of destroying GameObjects directly, allowing an opportunity for objects to animate on death.

<details><summary>How</summary>

 - Create a C# script "DeathEffect" under Assets/Code/Components/Death.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Any/all component(s) on the gameObject that inherit from this to add
/// effects or animations on death, before the GameObject is destroyed.
/// </summary>
[RequireComponent(typeof(DeathEffectManager))]
public abstract class DeathEffect : MonoBehaviour
{
  /// <summary>
  /// How long we need to wait before destroying the
  /// GameObject to allow this effect to complete.
  /// </summary>
  public abstract float timeUntilObjectMayBeDestroyed
  {
    get;
  }

  /// <summary>
  /// Do not call directly.  Initiated only by the DeathEffectManager.
  /// 
  /// When we are ready to destroy a GameObject, PlayDeathEffects is called
  /// and then we wait for at least timeUntilObjectMayBeDestroyed
  /// before calling Destroy on the gameObject.
  /// </summary>
  public abstract void PlayDeathEffects();
}
```

 - Create a C# script "DeathEffectManager" under Assets/Code/Components/Death.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Manages playing multiple effects, and destroying
/// the gameObject when they all complete.
/// 
/// Required if the GameObject has any DeathEffects.
/// </summary>
public class DeathEffectManager : MonoBehaviour
{
  /// <summary>
  /// Called when an object that may have a death effect
  /// should be destroyed.
  /// 
  /// If no DeathEffects are found, the gameObject is 
  /// destroyed immediatally.
  /// </summary>
  /// <param name="gameObject">
  /// The gameObject to destroy.
  /// </param>
  public static void PlayDeathEffectsThenDestroy(
    GameObject gameObject)
  {
    DeathEffectManager deathEffectManager 
      = gameObject.GetComponent<DeathEffectManager>();

    if(deathEffectManager == null)
    {
      // If there is no DeathEffectManager on 
      // this gameObject, Destroy it now.
      Destroy(gameObject);
      return;
    }

    // Start death sequence, which ends with 
    // destroying the gameObject.
    deathEffectManager.PlayDeathEffectsThenDestroy();
  }

  /// <summary>
  /// Initiated only by PlayDeathEffectsThenDestroy.
  /// 
  /// Plays any DeathEffects then destroys this GameObject.
  /// </summary>
  void PlayDeathEffectsThenDestroy()
  {
    DeathEffect[] deathEffectList
      = gameObject.GetComponentsInChildren<DeathEffect>();

    float maxTimeUntilObjectMayBeDestroyed = 0;
    for(int i = 0; i < deathEffectList.Length; i++)
    {
      DeathEffect deathEffect = deathEffectList[i];
      maxTimeUntilObjectMayBeDestroyed = Mathf.Max(
        maxTimeUntilObjectMayBeDestroyed,
        deathEffect.timeUntilObjectMayBeDestroyed);

      // Start each individual DeathEffect to run in parallel.
      deathEffect.PlayDeathEffects();
    }

    // Wait until the slowest DeathEffect completes then Destroy.
    Destroy(gameObject, maxTimeUntilObjectMayBeDestroyed);
  }
}
```

</details>

<details><summary>Why not just play effects OnDestroy()?</summary>

OnDestroy is called anytime the object is destroyed, but we only want the death effects to trigger in certain circumstances.  For example, when we quit back to the main menu, we do not want explosions spawning for character being destroyed while closing level 1.

This pattern was selected because:

 - It gives us easy control over when DeathEffects should be considered, vs promptly destroying the object.
 - It gracefully falls back to Destroy when there are no DeathEffects to play.
 - It allows for several separate DeathEffects to be combined, creating a new kind of effect.

As always, there are probably a thousand different ways you could achieve similar results.

</details>
<details><summary>Why is there a public method that 'should not be called directly'?</summary>

PlayDeathEffects() in the DeathEffect class has a public method with a comment saying it 'should not be called directly'.  So why is it public?

In order to support multiple DeathEffects and to be able to fallback gracefully when an object does not have one, we always start effects by calling the public static method in DeathEffectManager, PlayDeathEffectsThenDestroy.  

Since DeathEffectManager is a class of its own, we would not be able to call a private or protected method in DeathEffect.

'internal' could be an option to consider, but typically when working in Unity you are working in a single project - therefore internal is effectively the same as public.

You might also consider using nested classes.  For simplicity in the tutorial, we're not using nested classes as they can be a bit confusing.  If you are familiar with this topic, briefly you could make DeathEffectsManager a class nested inside DeathEffect and then make PlayDeathEffects() private, and the rest pretty much works the same.

</details>

<details><summary>Why are you using Mathf and not System.Math?</summary>

Unity offers the UnityEngine.Mathf class to try and make some things a little easier.  Basically it's the same APIs which are offered from the standard System.Math class (which is also still available to use if you prefer).  The main difference is all of the APIs in Mathf are focused on the float data type, where the System.Math class often prefers double.  Most of the data you interact with in Unity is float.  

</details>



## Kill the player when he hits a ball

When the player comes in contact with a spiked ball, kill him!

<details><summary>How</summary>

 - Create a C# script "LayerMaskExtensions" under Assets/Code/Utils.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Provides additional convenience methods for Unity's LayerMask.
/// </summary>
public static class LayerMaskExtensions
{
  /// <summary>
  /// Determines if the layer is part of this layerMask.
  /// </summary>
  /// <param name="mask">
  /// The layer mask defining which layers should be included.
  /// </param>
  /// <param name="layer">
  /// The layer to check against the mask.
  /// </param>
  /// <returns>
  /// True if the layer is part of the layerMask.
  /// </returns>
  /// <remarks>
  /// This method is used to wrap the bit logic below as 
  /// it's not an intuitive read.
  /// </remarks>
  public static bool Includes(
    this LayerMask mask,
    int layer)
  {
    return (mask.value & 1 << layer) > 0;
  }
}
```

 - Create a C# script "KillOnContactWith" under Assets/Code/Components/Death.
 - Select the Spike Ball prefab and add the KillOnContactWith component.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Kills anything which collides with this GameObject
/// if the thing that hit us is included in the provided LayerMask.
/// </summary>
[RequireComponent(typeof(Collider2D))]
public class KillOnContactWith : MonoBehaviour
{
  /// <summary>
  /// Defines which layers will be killed on contact.
  /// </summary>
  [SerializeField]
  LayerMask layersToKill;

  /// <summary>
  /// A Unity event called anytime an object hits 
  /// this GameObject's collider.
  /// 
  /// Consider killing the thing we touched.
  /// </summary>
  /// <param name="collision">
  /// The thing we touched.
  /// </param>
  protected void OnCollisionEnter2D(
    Collision2D collision)
  {
    ConsiderKilling(collision.gameObject);
  }

  /// <summary>
  /// Checks if we should kill the object just touched, 
  /// if so Destroy that GameObject.
  /// </summary>
  /// <param name="gameObjectWeJustHit">
  /// The gameObject just touched.
  /// </param>
  void ConsiderKilling(
    GameObject gameObjectWeJustHit)
  {
    // Compare the GameObject's layer to the LayerMask
    if(layersToKill.Includes(gameObjectWeJustHit.layer) == false)
    { // This object gets to live.
      return;
    }

    // Kill it!
    DeathEffectManager.PlayDeathEffectsThenDestroy(gameObjectWeJustHit);
  }
}
```

 - Edit -> Project Settings -> Tags and Layers.
 - Create a custom Layer for 'Player'.
 - Select the Character GameObject and change its Layer to 'Player'.
 - Select the Spike Ball prefab, update 'Layers To Kill' to 'Player'.

<img src="http://i.imgur.com/wrkb3eJ.png" width=100px />

Hit play to watch the player die:

<img src="http://i.imgur.com/gKEl8wE.gif" width=200px />

For now, to test again stop and hit play again.  We'll respawn the player later in the tutorial.


</details>
<details><summary>What is a C# extension method and why use it?</summary>

Extension methods are a way of adding additional methods to a class or struct you don't own.  In this example, Unity has a struct 'LayerMask'.  That struct does not offer an easy way to determine if a layer is part of that LayerMask.  Using extensions, we are able to create an 'Includes' method that then can be used as if Unity had written it for us.

This allows us to focus on intent and forget the gory details.  For example this statement:

```csharp
if((layersToKill.value & 1 << gameObjectWeJustHit.layer) > 0) 
...
```

Can now be written like so, which should be easier for people to follow.

```csharp
if(layersToKill.Includes(gameObjectWeJustHit.layer)) 
...
```

</details>
<details><summary>What is this '& 1 <<' black magic?</summary>

Bitwise operations... which are beyond the scope of this tutorial.  More specifically, this is 'bitwise and' and 'bit shifting' if you would like to read more about this.  Here is a [Stackoverflow post on the topic](http://answers.unity3d.com/questions/8715/how-do-i-use-layermasks.html).

</details>

## Create an explosion prefab

Create an explosion prefab.  We are using a scaled Fireball from the standard assets' ExplosionMobile prefab.

<details><summary>How</summary>

 - In the Assets directory, right click -> Import Package -> ParticleSystems
 - You could import everything, but for this tutorial we only need the prefab 'ExplosionMobile'.

<img src="http://i.imgur.com/6lVqJtL.png" width=150px />

 - Drag the prefab from Assets/Standard Assets/ParticleSystems/Prefabs/ExplosionMobile into the scene.
 - Drag the 'Fireball' child GameObject out of the 'ExplosionMobile' GameObject, making Fireball stand alone.
 - Delete the 'ExplosionMobile' GameObject.
 - Rename Fireball to 'Explosion'.

<img src="http://i.imgur.com/IPPAzHG.gif" width=200px />

 - To preview the Explosion effect, select the GameObject.  In the 'Scene' tab a 'Particle effect' panel appears.  Click 'Stop' and then 'Simulate' to see the explosion.
 - Under the Particle Systems component, change the 'Scaling Mode' to 'Local'.
 - Change the Transform scale to about (.25, .25, .25).
 - Preview the Explosion effect again.

<img src="http://i.imgur.com/bOWigXy.gif" width=300px />

 - Drag/drop the Explosion GameObject to Asserts/Prefabs to create a prefab.
 - Delete the Explosion GameObject.

Note that by default Unity scales particle systems based on the viewport size.  This means that while previewing the effect in the 'Scene' tab, the size you see relative to the world may not be what's going to happen in game.  To limit impact from this, you can change the 'Max Particle Size' to 1 - but for best results always confirm your work in the 'Game' tab while playing.

<img src="http://i.imgur.com/h12U4xa.png" />

</details>
<details><summary>What's a particle / particle system?</summary>

A particle is a small 2D image managed by a particle system.  It's optimized to display a large number of similar particles at the same time, possible with different colors, sizes, etc.

A Particle System component animates a large numbers of particles to create effects such as fluid, smoke, and fire. Read more about [Particle Systems from Unity](https://docs.unity3d.com/Manual/class-ParticleSystem.html).

</details>
<details><summary>Why not use the entire MobileExplosion prefab?</summary>

You could, but for this tutorial we are creating WebGL builds of the game.  WebGL does not perform as well in general, and the performance tanks if you use the entire ExplosionMobile prefab.  Effects that would be perfectly fine in the Unity editor and as a desktop build may not work well in the browser. 

If you are not going to build for WebGL, go ahead and try using the ExplosionMobile prefab or other particle system you think looks good.

</details>

## Explosion self destruct

Create a script to destroy the explosion GameObject after the effect completes.

<details><summary>How</summary>

 - Create a C# script "SuicideIn" under Assets/Code/Components/Death.
 - Select the Fireball prefab, add the SuicideIn component.
 - Paste in the following code:
 
```csharp
using UnityEngine;

/// <summary>
/// Destroy's the GameObject after timeTillDeath seconds
/// since the object was instatiated have passed.
/// </summary>
public class SuicideIn : MonoBehaviour
{
  /// <summary>
  /// How long, in seconds, till this object should be destroyed.
  /// </summary>
  [SerializeField]
  float timeTillDeath = 5;
  
  /// <summary>
  /// A Unity event, called when this GameObject
  /// is instatiated.
  /// 
  /// Begin the countdown till Destroy.
  /// </summary>
  protected void Start()
  {
    Debug.Assert(timeTillDeath > 0,
      "timeTillDeath must be greater than 0");

    Destroy(gameObject, timeTillDeath);
  }
}
```

</details>
<details><summary>Why bother, the explosion is not visible after a few seconds?</summary>

Similar to how we destroyed balls which rolled off the bottom of the screen in chapter 1, we need to ensure the explosion GameObjects are destroyed at some point.

The explosion effect on screen only lasts for a few seconds, but Unity does not realize this on its own.  Destroying the GameObject prevents Unity from wasting resources on the old GameObjects which are never going to be visible again.

In other words, this script ensures that our explosions do not result in a memory leak.

</details>


## Explosion sound effect

Add a sound effect to the explosion prefab.  We are using a [shotgun blast from Sound Bible](http://soundbible.com/1919-Shotgun-Blast.html).

<details><summary>How</summary>

 - Drag/drop the .wav file into Assets/Art.
 - Drag/drop the Explosion prefab into the Hierarchy to create a GameObject.
 - Drag/drop the .wav file from Assets/Art onto the Explosion GameObject.

 This creates an AudioSource component on the GameObject.

 - Click 'Apply' prefab.

<img src="http://i.imgur.com/atFbwlK.png" width=100px />

 - Delete the Explosion GameObject from the Hierarchy.

Hit play, you should hear the sound when the player dies.

</details>
<details><summary>Why is Audio on a GameObject vs simply a clip which is played?</summary>

Audio playback in Unity is built to support 3D audio.  3D audio refers to the feature where the closer an object making noise is to your ear, the louder it is.  Additionally 3D sound is directional, so sounds to the players left would be loudest in the left speaker.

Your 'ear' is typically the camera itself.  This is managed by the AudioListener component which was placed on the Main Camera by default when the scene was created.  You could choose to move this component to the character instead, if appropriate.

To enable 3D audio, sounds need to originate at a position in the world.  We use the AudioSource component to play clips.  As a component, it must live an a GameObject which in turn must have a Transform -- the position we are looking for.

For consistency, 2D audio is played the same way.  2D means we don't have the features above, the clip sounds the same regardless of where it the world it was initiated from.  Note that audio is 2D by default.

Alternatively you could use the Unity API to play a clip as shown below.  This API will create an empty GameObject at the location provided, add an AudioSource component to it, configure that source to use the clip specified and have the AudioSource start playing.  After the clip completes, the GameObject will be destroyed automatically.  

```csharp
[SerializeField]
AudioClip clip;

protected void Start() 
{
  AudioSource.PlayClipAtPoint(clip, new Vector3(5, 1, 2));
}
```

</details>
<details><summary>How might you play mulitple clips at the same time?</summary>

Each AudioSource can be configured for one clip at a time.  To play multiple clips in parallel, you could use multiple AudioSources by placing multiple on a single GameObject or instantiating multiple GameObjects.  You can also use the following API to play a clip in parallel:

```csharp
GetComponent<AudioSource>().PlayOneShot(clip);
```

This will start playing another clip, re-using an existing AudioSource component (and its GameObject's position as well as the audio configuration options such as pitch).

</details>
<details><summary>Could you RNG select the clip to play?</summary>

Anything is possible.  Here's a little code sample that may help you get started.  

On a related note, you could also randomize the pitch to get some variation between each clip played.  e.g. this could be a nice addition to a rapidly firing gun.

```csharp
[SerializeField]
AudioClip clip1;
[SerializeField]
AudioClip clip2;

protected void OnEnable()
{
  AudioSource audioSource = GetComponent<AudioSource>();
  switch(UnityEngine.Random.Range(0, 2))
  {
    case 0:
    audioSource.clip = clip1;
    break;
    case 1:
    audioSource.clip = clip2;
    break;
  }
  audioSource.Play();
}
```

</details>



## Spawn explosion when the character dies

Add a script which spawns the explosion prefab when the character dies.

<details><summary>How</summary>

 - Create a C# script "DeathEffectSpawn" under Assets/Code/Components/Death.
 - Select the Character GameObject and add the DeathEffectSpawn component.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Spawns another GameObject before this GameObject is destroyed.
/// </summary>
[RequireComponent(typeof(Collider2D))]
public class DeathEffectSpawn : DeathEffect
{
  [SerializeField]
  GameObject gameObjectToSpawnOnDeath;

  public override float timeUntilObjectMayBeDestroyed
  {
    get
    {
      return 0;
    }
  }

  public override void PlayDeathEffects()
  {
    Debug.Assert(gameObjectToSpawnOnDeath != null,
      "gameObjectToSpawnOnDeath must not be null");

    Collider2D collider = GetComponent<Collider2D>();

    // Spawn the other GameObject at my current location
    Instantiate(
      gameObjectToSpawnOnDeath,
      collider.bounds.center, 
      Quaternion.identity);
  }
}
```

 - Select the Character GameObject and under DeathEffectSpawn, assign the Explosion prefab to 'Game Object To Spawn'.

Click play and an explosion should spawn when the player dies:

<img src="http://i.imgur.com/XhhkRpC.gif" width=200px />

</details>

<details><summary>Why not spawn the explosion at transform.position?</summary>

The character sprite was configured with Pivot 'Bottom'.  The transform.position refers to the location of this pivot point.  If we were to target tranform.position instead, the explosion would center around the character's feet.

This component could be reused on other GameObjects which may have a different pivot point. It will work correctly so long as the object has a collider.

We use the collider's bounds to determine where to spawn the explosion.  The [bounds struct](https://docs.unity3d.com/ScriptReference/Bounds.html) has a number of convienent methods for things like determining the center point of an object.

</details>

## Rotate the character when he walks the other way

Flip the character's sprite when he switches between walking left and walking right.

<details><summary>How</summary>

 - Create a C# script "RotateEntity" under Assets/Code/Components/Movement.
 - Select the character GameObject and add the RotateEntity component.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Rotates an entity based on it's current horizontal velocity.
/// 
/// This causes entities to face the direction they are walking.
/// </summary>
[RequireComponent(typeof(Rigidbody2D))]
public class RotateEntity : MonoBehaviour
{
  /// <summary>
  /// The rotation that's applied when looking left (vs right).
  /// </summary>
  /// <remarks>
  /// Cached here for performance.
  /// </remarks>
  static readonly Quaternion backwardsRotation = Quaternion.Euler(0, 180, 0);

  /// <summary>
  /// Used to control movement.
  /// </summary>
  /// <remarks>
  /// Cached here for performance.
  /// </remarks>
  Rigidbody2D myBody;

  /// <summary>
  /// The direction we are currently walking, 
  /// used to know when we turn around.
  /// </summary>
  /// <remarks>
  /// Defaults to true as our entities are configured facing right.
  /// </remarks>
  bool _isGoingRight = true;

  /// <summary>
  /// The direction we are currently walking.
  /// When changed, flips the rotation so the entity is facing forward.
  /// </summary>
  bool isGoingRight
  {
    get
    {
      return _isGoingRight;
    }
    set
    {
      if(isGoingRight == value)
      { // The value is not changing
        return;
      }

      // Flip the entity
      transform.rotation *= backwardsRotation;
      _isGoingRight = value;
    }
  }

  /// <summary>
  /// A Unity event, called before this GameObject is instantiated.
  /// </summary>
  protected void Awake()
  {
    myBody = GetComponent<Rigidbody2D>();
    Debug.Assert(myBody != null);
  }

  /// <summary>
  /// A Unity event, called each frame.
  /// 
  /// Updates the entities rotation.
  /// </summary>
  protected void Update()
  {
    float xVelocity = myBody.velocity.x;
    // If there is any horizontal movement
    if(Mathf.Abs(xVelocity) > 0.1)
    { 
      // Determine the current walk direction
      // This may rotate the sprite c/o
      // the smart property above.
      isGoingRight = xVelocity > 0;
    }
  }
}
```

</details>
<details><summary>What's a C# smart property?</summary>

In C#, data may be exposed as either a Field or a Property.  Fields are simply data as one would expect.  Properties are accessed in code like a field is, but they are capable of more.

In this example, when isGoingRight changes between true and false, the GameObject's transform is rotated so that the sprite faces the correct direction.  Leveraging the property changing to trigger the rotation change is an example of logic in the property making it 'smart'.

There are pros and cons to smart properties.  For example, one may argue that including the transform change when isGoingRight is modified hides the mechanic and makes the code harder to follow.  There are always alternatives if you prefer to not use smart properties.  For example:

```csharp
bool isGoingRightNow = xVelocity > 0;
if(isGoingRight != isGoingRightNow) 
{
  transform.rotation *= backwardsRotation;    
  isGoingRight = isGoingRightNow;
}
```

</details>

<details><summary>What's a Quaternion?</summary>

A Quaternion is how rotations are stored in a game engine.  They represent the rotation with (x, y, z, w) values, stored in this fashion because that it is an effecient way to do the necessary calculations when rendering on object on screen.

You could argue that this is overkill for a 2D game as in 2D the only rotation that may be applied is around the Z axis, and I would agree.  However remember that Unity is a 3D game engine.  When creating a 2D game, you are still in a 3D environment.  Therefore under the hood, Unity still optimizes its data for 3D.

Quaternions are not easy for people to understand.  When we think of rotations, we typically think in terms of 'Euler' (pronounced oil-er) rotations.  Euler rotations are degrees of rotation around each axis, e.g. (0, 0, 30) means rotate the object by 30 degrees around the Z axis.

In the inspector, modifying a Transform's rotation is done in Euler.  In code, you can either work with Quatenions directly or use Euler and then convert it back to Quatenion for storage.

Given a Quatenion, you can calculate the Euler value like so:

```csharp
Quaternion myRotationInQuaternion = transform.rotation;
Vector3 myRotationInEuler = myRotationInQuaternion.eulerAngles;
```

Given an Euler value, you can calculate the Quatenion:

```csharp
Quaternion rotationOfZ30Degrees = Quaternion.Euler(0, 0, 30);
```

Quaternions may be combined using Quaternion multiplication:

```csharp
Quaternion rotationOfZ60Degrees 
  = rotationOfZ30Degrees * rotationOfZ30Degrees;
```

</details>

<details><summary>Why not compare to 0 when checking if there is no movement?</summary>

In Unity, numbers are represented with the float data type.  Float is a way of representing decimal numbers but is a not precise representation like you may expect.  When you set a float to some value, internally it may be rounded ever so slightly.

The rounding that happens with floats allows operations on floats to be executed very quickly.  However it means we should never look for exact values when comparing floats, as a tiny rounding issue may lead to the numbers not being equal.

In the example above, as the velocity approaches zero, the significance of if the value is positive or negative, is lost.  It's possible that if we were to compare to 0 that at times the float may oscilate between a tiny negative value and a tiny positive value causing the sprite to flip back and forth.

</details>

## Restrict movement to stay on screen

Create a script which ensures the character can not walk off screen.

<details><summary>How</summary>

 - Create a C# script "KeepWalkMovementOnScreen" under Assets/Code/Components/Movement.
 - Select the Character GameObject and add the KeepWalkMovementOnScreen component.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Ensures that the entity stays on the screen. 
/// It will flip the current walk direction automatically 
/// (which has no impact on the Player but causes enemies to bounce).
/// </summary>
[RequireComponent(typeof(Rigidbody2D))]
public class KeepWalkMovementOnScreen : MonoBehaviour
{
  #region Data
  /// <summary>
  /// Used to determine if we are currently moving.
  /// </summary>
  Rigidbody2D myBody;

  /// <summary>
  /// Used to cause the entity to start walking the 
  /// opposite direction when it hits the edge of the screen.
  /// 
  /// This is not required and may be null.
  /// </summary>
  WalkMovement walkMovement;

  /// <summary>
  /// The area of the world the camera can see.
  /// 
  /// This is calculated on Awake, and therefore
  /// does not consider effects like screen shake.
  /// </summary>
  Bounds screenBounds;
  #endregion

  #region Init
  /// <summary>
  /// A Unity event, called once before this GameObject
  /// is spawned in the world.
  /// </summary>
  protected void Awake()
  {
    myBody = GetComponent<Rigidbody2D>();
    walkMovement = GetComponent<WalkMovement>();

    Camera camera = Camera.main;
    Vector2 screenSize = new Vector2(
      (float)Screen.width / Screen.height, 
      1);
    screenSize *= camera.orthographicSize * 2;

    screenBounds = new Bounds(
      (Vector2)camera.transform.position, 
      screenSize);

    Debug.Assert(myBody != null);
  }
  #endregion

  #region Events
  /// <summary>
  /// A Unity event, called each frame.
  /// 
  /// If the entity is off screen, pop it back 
  /// and flip the walk direction.
  /// </summary>
  protected void Update()
  {
    // Check if the entity is off screen
    if(screenBounds.Contains(transform.position) == false)
    { 
      // Move the entity back to the edge of the screen
      transform.position =
        screenBounds.ClosestPoint(transform.position);
      if(walkMovement != null)
      {
        // Flip the walk direction
        walkMovement.desiredWalkDirection 
          = -walkMovement.desiredWalkDirection;
      }
    }
  }
  #endregion
}
```

</details>

<details><summary>Why use bounds for these checks?</summary>

There are a few ways you could check for an entity walking off the edge of the screen.  I choose to use the Unity bounds struct because it has methods which make the rest of this component easy.  Specifically:

 - Contains: Check if the current position is on the screen.
 - ClosestPoint: Return the closest point on screen for the character, used when he is off-screen to teleport him back.

</details>

<details><summary>What does flipping the walk direction do?</summary>

Each frame the PlayerController sets the walk direction without consider the previous value.  So flipping the walk direction here is promptly overwritten by the PlayerController - resulting in little or no impact to movement in the game.

We included this logic because not all controllers are going to work the same way.  Later in the tutorial we will be adding another entity that uses WalkMovement by only setting desiredWalkDirection periodically.  For that entity, flipping the direction will cause the entity to bounce off the side of the screen and walk the other way.

This logic doesn't impact the character but it's not harmful either and it fits with the theme of this component, enabling reuse.

</details>

<details><summary>What's the different between setting transform.position and using myBody.MovePosition?</summary>

Updates to the Transform directly will teleport your character immediatelly and bypass all physics logic.  

Using the rigidbody.MovePosition method will smoothly transition the object to its new postion.  It's very fast, but if you try this and watch closely, MovePosition is animating a few frames on the way to the target position instead of going there immediatelly.

We are not suggesting one approach should always be used over the other - consider the use case and how you want your game to feel, sometimes teleporting is exactly the feature you're looking for.  

Be careful when you change position using either of these methods as opposed to using forces on the rigidbody.  It's possible that you teleport right into the middle of another object.  The next frame, Unity will try to react to that collision state and this may result in objects popping out in strange ways.

In this component we are setting transform.position for the teleport effect.  If rigidbody.MovePosition was used instead, occasionally issues would arrise as MovePosition competes with other forces on the object.

</details>

## Jump

Add the ability for the character to jump.  Add a sound effect as well, we are using 'phaseJump1' from [Kenney.nl's Digital Audio pack](http://kenney.nl/assets/digital-audio).

Note the character will be able to double jump / fly, this will be addressed later in the tutorial.

<details><summary>How</summary>

 - Create a C# script "JumpMovement" under Assets/Code/Components/Movement.
 - Select the Character GameObject and add the JumpMovement component.
 - Paste in the following code:

```csharp
using UnityEngine;

/// <summary>
/// Controls the entity's jump.  
/// 
/// Another component drives when to jump via Jump().
/// </summary>
[RequireComponent(typeof(Rigidbody2D))]
[RequireComponent(typeof(AudioSource))]
public class JumpMovement : MonoBehaviour
{
  /// <summary>
  /// The sound to play when the character starts their jump.
  /// </summary>
  [SerializeField]
  AudioClip jumpSound;

  /// <summary>
  /// How much force to apply on jump.
  /// </summary>
  [SerializeField]
  float jumpSpeed = 6.5f;

  /// <summary>
  /// Used to add force on jump.
  /// </summary>
  Rigidbody2D myBody;

  /// <summary>
  /// Used to play sound effects.
  /// </summary>
  AudioSource audioSource;

  /// <summary>
  /// A Unity event, called once before this GameObject
  /// is spawned in the world.
  /// </summary>
  protected void Awake()
  {
    myBody = GetComponent<Rigidbody2D>();
    audioSource = GetComponent<AudioSource>();

    Debug.Assert(myBody != null);
    Debug.Assert(audioSource != null);
  }

  /// <summary>
  /// Adds force to the body to make the entity jump.
  /// </summary>
  public void Jump()
  {
    Debug.Assert(jumpSpeed >= 0,
      "jumpSpeed must not be negative");
    
    // Jump!
    myBody.AddForce(
      (Vector2)transform.up * jumpSpeed, 
      ForceMode2D.Impulse);

    // Play the sound effect
    audioSource.PlayOneShot(jumpSound);
  }
}
```

 - Drag/drop the jump sound effect into Assets/Art.
 - Select the Character GameObject and add an 'AudioSource' component if it does not already have one.
 - Under the JumpMovement component for the character, assign the jummp sound.

<img src="http://i.imgur.com/q4LEETw.gif" width=150px />


 - Edit the existing C# script "PlayerController" under Assets/Code/Components/Movement.
 - Add the following (or [click here for the full file to copy/paste from](TODO file link):

<details><summary>Existing code</summary>

```csharp
using UnityEngine;

/// <summary>
/// Wires up user input, allowing the user to 
/// control the player in game with a keyboard.
/// </summary>
[RequireComponent(typeof(WalkMovement))]
```

</details>

```csharp
[RequireComponent(typeof(JumpMovement))] 
```

<details><summary>Existing code</summary>


```csharp
public class PlayerController : MonoBehaviour
{
  /// <summary>
  /// Used to cause the object to walk.
  /// </summary>
  WalkMovement walkMovement;
```

</details>

```csharp
  /// <summary>
  /// Used to cause the object to jump.
  /// </summary>
  JumpMovement jumpMovement; 
```

<details><summary>Existing code</summary>


```csharp
  /// <summary>
  /// A Unity event, called once before the GameObject
  /// is instantiated.
  /// </summary>
  protected void Awake()
  {
    walkMovement = GetComponent<WalkMovement>();
    Debug.Assert(walkMovement != null);
```

</details>

```csharp
    jumpMovement = GetComponent<JumpMovement>(); 
    Debug.Assert(jumpMovement != null); 
```

<details><summary>Existing code</summary>


```csharp
  }

  /// <summary>
  /// A Unity event, called every x ms of game time.
  /// 
  /// Consider moving.
  /// </summary>
  /// <remarks>
  /// Moving uses an input state, and therefore may be captured 
  /// on Update or FixedUpdate, we use FixedUpdate since physics 
  /// also runs on FixedUpdate, so trying to do this on update would
  /// require an extra cache (w/o benefit).
  /// </remarks>
  protected void FixedUpdate()
  {
    // Consider moving left/right based off keyboard input.
    walkMovement.desiredWalkDirection 
      = Input.GetAxis("Horizontal");
  }
```

</details>

```csharp
  /// <summary>
  /// A Unity event, called once per frame.
  /// 
  /// Consider jumping.
  /// </summary>
  /// <remarks>
  /// Jumping uses an input event, and therefore must be
  /// captured on Update.
  /// </remarks>
  protected void Update()
  {
    if(Input.GetButtonDown("Jump"))
    {
      jumpMovement.Jump();
    }
  }
```

<details><summary>Existing code</summary>


```csharp
}
```

</details>



Click play, you can now jump around.  But you can hold onto the side of a platform while falling and spam the space bar to fly away:

<img src="http://i.imgur.com/RRpRio5.gif" width=250px />
</details>
<details><summary>Why AddForce here instead and what's 'Impulse'?</summary>

As discussed above when creating the WalkMovement component, you could always create mechanics using either AddForce or by modifying the velocity.

We are using AddForce to jump in this component.  Using velocity here instead would have actually created the same basic jump experience we are looking for.  

Using AddForce for the jump may provide a better experience for some corner cases or future mechanics.  For example, if we wanted to support double jump in this game, initiating the second jump while in the air would feel much different.

What is ForceMode2D.Impulse and how is it different from ForceMode2D.Force?

These options appear to have the same effect on effects on objects, the difference is only the scale.  The unit for Impulse is defined as force per FixedUpdate.  The unit for Force is defined as force per second.  In the end, it means when configured your speed you may need a much larger value when using Force than while using Impulse.

</details>
<details><summary>How do you know when to use Update vs FixedUpdate for Input and rigidbodies?</summary>

Unity recommends always using FixedUpdate when interacting with a rigidbody as physics is processed in FixedUpdate. 

There is nothing blocking you from changing the rigidbody in an Update loop.  You could, for example, AddForce every Update.  This is not recommended and may lead to inconsistent experiences.

For Input:

 - When reading the current Input state (e.g. using Input.GetAxis), either FixedUpdate or Update is fine.  For example if you are checking the current position of the joystick, you'll get the same information in FixedUpdate and Update. 
  - If you need to modify a rigidbody based on current Input state, I recommend reading Input in FixedUpdate to keep it simple.
 - When checking for an Input event (e.g. using Input.GetButtonDown), you must use Update.  Input is polled in the Update loop.  Since it's possible for two Updates to happen before a FixedUpdate, some events may be missed when only checking in FixedUpdate.  
   - Always read events in Update.  Unity will not block or warn you when checking for an event in FixedUpdate, and most of the time it will work - but occasional bugs will arrise.

</details>


## Add Platformer Effect to platforms

Add the PlatformerEffect2D component to each platform, allowing the character to jump through them.

Note collisions with the sides of the platforms may feel off, we'll update this in the next section.

<details><summary>How</summary>

 - Select all of the Platform GameObjects.
 - Add Component: PlatformEffector2D.
 - Under the BoxCollider2D, select 'Use by Effector'.

<img src="http://i.imgur.com/55YiY3N.gif" width=200px />

Click play to test it out.  You may need to increase the character's Jump Speed to really see how platformer effect works:

<img src="http://i.imgur.com/hRe7CEJ.gif" width=200px />

</details>
<details><summary>Wow that was easy, what else like this can Unity do for 'free'?</summary>

Effectors in Unity are easy ways to add various mechanics to the game.  The one-way collision effect we are using here happens to be a very common mechanic for 2D games, so Unity has this component ready to drop in.  

Unity is not doing anything with these components that you technically could not have built yourself in a custom script, but that said adding the one-way effect the PlatformerEffector2D creates would not be easy to do.

Read more about the [various 2d effectors in Unity](https://docs.unity3d.com/Manual/Effectors2D.html) including a conveyor belt, repulsion, and floating effects.

</details>


## Update the platforms' surface arc

Reduce the PlatformerEffector2D Surface Arc to about 135.

<details><summary>How</summary>

 - Select all of the Platform GameObjects.
 - Change the 'Surface Arc' to '135' under the Platform Effector 2D compenent.

Play, now the character should not be able to stick to the sides while falling:

<img src="http://i.imgur.com/GGzbkdp.gif" width=200px />

</details>



## Test!

Chapter 2, complete!  Your game should now look a lot like the gif at the top.  You can compare to our  [demo build](https://hardlydifficult.com/PlatformerTutorialPart2/index.html) and review the [Unity Project / Source Code for Chapter 2](https://github.com/hardlydifficult/Unity2DPlatformerTutorial/tree/Part2). TODO links

Additionally to review, you may want to:
 - Consider sorting components on the character GameObject, as it's starting to look a little cluttered.
 - Try adjusting the jump speed for the character.
 - Maybe try different particle systems for the explosion death effect.
 - Cut a test build and try it outside of the Unity editor environment.

<details><summary>How do you sort components on a GameObject?</summary>

The order does not impact anything.  So why bother?  Just tidyness really.   As the number of components grows it may be nice to have them presented in an order you find more intuative.

Start by collapsing everything.
To sort, select the GameObject and in the Inspector

Transform has to be first. Then Unity stuff.  Then scripts.

Unity grouping logically similar components, eg. Rigidbody near Collider

Scripts in order where possible, like DeathEffectManager before any DeathEffects.

<img src="http://i.imgur.com/ElAr8xt.gif" width=150px />

On a related note, order does matter when for some scripts in terms of which compoment executes before another.  To ma... Script execution order

TODO

</details>

