# Portfolio

Welcome to my portfolio! I love working with Unity and C#! Originally I started off as an FOSS developer, working with Linux and open source tools. However after moving to Unity, I fell in love with C#. The language is really awesome, and gives the benefit of strongly typed languages with a lot of ease of convience features.

I started my first Unity project in 2018, and still remember jumping into the editor and being flooded with many cool new features. The journey was long, but I have consistently worked in the Unity editor ever since, and hope to share here my portfolio and some details of the process.

# Contents

I. Level Blockout

II. Code Samples

# RPG Game

I wanted to make a vertical slice for an RPG game. Wtih a vertical slice I could experiment with systems, while efficiently moving forward. I've on RPG games before and this time I wanted to put to use the lessons I've learned from the past.

# Player Start

I wanted the player to start in a small closed space, as many old FPS games have done. I also wanted the player to start in a comfortable space, so they can gently ease into the game. To make the player more comforable I worked on a fire torch, including the comforting sounds of fire.


# Vertical Slice

The goal was to make a vertical slice of an RPG game. 

## Port Blockout

![](https://github.com/pjkw/Portfolio/blob/main/images/Port%20Blockout.png)

In designing this space I wanted to place emphasis on functional spaces that would be useful for a port.

For example, a cargo unloading area, close to the ship, so the crew can unload goods with a short distance from the ship:

![](https://github.com/pjkw/Portfolio/blob/main/images/Unloading%20Area.png)

A ramp, to roll heavy cargo up:

![](https://github.com/pjkw/Portfolio/blob/main/images/Ramp.png)

And a highrise area, so the high tide does not wash up on the port:

![](https://github.com/pjkw/Portfolio/blob/main/images/Highrise.png)

Also, I wanted to avoid a flat terrain, and frame the landscape with cliffs and mountains, surrounding the port:

![](https://github.com/pjkw/Portfolio/blob/main/images/Port%20Landscape.png)

## Port Town

To contrast the start of the level, with a heavy storm, rain, and lighting, I wanted to create a cozy tavern, where the player can find refugee from the storm, and also talk to an NPC to set up the first quest.

# Developing an AI Character

In this section we will go over the process of developing an AI character.

## Scene set up

We start off by creating a new empty scene. In this scene we will include our core functionality, such as the player character, the game manager, and other systems. (Note that we would like some of these to be prefabs but since we are still continually reworking these components we will turn them into prefabs once they are closer to completion.)


![Scene Setup](https://github.com/pjkw/Portfolio/blob/main/images/Scene%20Setup.png)

To start off we create a plane with a checkered material to keep things simple and focus just on our new character.

![Empty Scene](https://github.com/pjkw/Portfolio/blob/main/images/Empty%20Scene.png)

## Asset import

For the dragon, we will be using the Unka dragon asset from the Unity asset store: https://assetstore.unity.com/packages/3d/characters/creatures/unka-the-dragon-84283

Even with turning the skybox back on, the dragon is too dark in the shadows and we are losing details.

![Dragon](https://github.com/pjkw/Portfolio/blob/main/images/Dragon.png)

## Lighting

By default, Unity comes with the ambient light set to a middle gray, which can be a little flat. We will turn off ambient light by setting the color values to dark, effectively removing ambient light, and letting us set up our own lighting.

![Ambient Light](https://github.com/pjkw/Portfolio/blob/main/images/Ambient%20Light.png)

Now we will add three point lighting to the scene to light up the details in the shadows, and from different sides. Essentially we will have a main light, a rim light, and a key light.

![3 Point Lighting](https://github.com/pjkw/Portfolio/blob/main/images/Dragon%202.png)

After adding three point lighting, we are now getting much more details in the texture. However, from far away the details can blend since the dragon is black. In a dark area, such as a dungeon, we can add a moving light on top of the dragon to get extra detail, while still having our scene be dark. I noticed they did this in the Dark Souls games, where if you're running around a dungeon you can tell there is an orb of light around the character. We can add this later when building out the level.

# Finite State Machine

Now that we have the scene and the dragon set up we can begin creating the AI. Our strategy as usual will be to make the system as modular as possible so that we can work efficiently.

We will start by creating the CharacterState class:

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CharacterState : MonoBehaviour
{
    [SerializeField]
    public enum State
    {
        None,
        Idle,
        Pursue,
        Attacking,
        DoneAttacking,
        Pause,
        TookDamage,
        BattleStance,
        PlayerDefeated, 
        Death,
        PostDeath
    }

    [SerializeField]
    public State state = State.None;
}
```

Next we will create a DragonStateManager, which will be the brains behind the AI dragon:

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CharacterStateManager : MonoBehaviour
{
    // this is the custom script for this script
    // this will be the only script we want to rewrite

    [SerializeField] CharacterLineOfSight characterLineOfSight;
    [SerializeField] CharacterCombat characterCombat;
    [SerializeField] CharacterState characterState;
    [SerializeField] CharacterStateText characterStateText;
    [SerializeField] CharacterMovementController characterMovementController;
    [SerializeField] CharacterAnimation characterAnimation;
    [SerializeField] CharacterAnimationEvents characterAnimationEvents;
    [SerializeField] Timer timer;
    [SerializeField] Animator animator;
    [SerializeField] CharacterNavMesh characterNavMesh;
    [SerializeField] Transform characterTransform;
    [SerializeField] Transform playerTransform;
    [SerializeField] CharacterKnockback characterKnockback;


    public GameObject healthBar;
    public GameObject character;

    public GameObject stateText;

    float time = 0.0f;
    bool runTimer = false;
    bool alreadyAddedToNumberOfAttackingEnemies = false;

    void Start()
    {
        GameManager.OnPlayerDeathEvent += OnPlayerDeathEvent;
    }

    void Update()
    {
        SwitchState();
        UpdateStateText();
        RunTimer();
    }

    void StartTimer()
    {
        runTimer = true;
    }

    void StopTimer()
    {
        runTimer = false;
    }

    void ResetTimer()
    {
        time = 0.0f;
    }

    void RunTimer()
    {
        if (runTimer)
        {
            time += Time.deltaTime;
        }
    }

    void UpdateStateText()
    {
        characterStateText.UpdateText(characterState.state.ToString());
    }

    void OnPlayerDeathEvent()
    {
        characterState.state = CharacterState.State.PlayerDefeated;
    }

    void OnSpottedPlayer()
    {
        if (characterLineOfSight.spotted)
        {
            // we will need some sort of state machine check here so that we don't revert back to the pursue state
            characterState.state = CharacterState.State.Pursue;
        }
    }

    bool CheckIfWithinAttackingDistance()
    {
        return characterMovementController.withinAttackingDistance;
    }

    void ChooseRandomAttack()
    {
        if (!CheckIfDoneAttacking()) return;

        int dice = RollDice(1, 3);

        if (dice == 1)
        {
            characterAnimation.Attack1();
        }

        else if (dice == 2)
        {
            characterAnimation.Attack2();
        }
    }

    bool CheckIfDoneAttacking()
    {
        return characterAnimationEvents.CheckIfDoneAttacking();
    }

    void ReturnToOriginalPosition()
    {
        characterMovementController.ReturnToOriginalPosition();
    }

    void SwitchState()
    {
        switch(characterState.state)
        {
            case CharacterState.State.None:

                OnSpottedPlayer();
                break;

            case CharacterState.State.Pursue:
                // this should be a separate state

                GameState.instance.combatState = GameState.CombatState.InCombat;

                characterMovementController.WalkTowardPlayer();


                if (CheckIfWithinAttackingDistance())
                {
                    characterState.state = CharacterState.State.Attacking;
                }

                if (!alreadyAddedToNumberOfAttackingEnemies)
                {
                    alreadyAddedToNumberOfAttackingEnemies = true;

                    GameState.instance.numberOfAttackingEnemies += 1;
                }

            

                /*
                characterCombat.Engage();

                if (CheckIfWithinAttackingDistance())
                {
                    characterState.state = CharacterState.State.Attacking;
                }


                */

                break;

            case CharacterState.State.Attacking:

                characterAnimation.BattleStance();
                characterNavMesh.StopAgent();
                characterTransform.LookAt(playerTransform);

                ChooseRandomAttack();
                
                // check if done attacking to transition to the done attacking state

                if (CheckIfDoneAttacking())
                {
                    characterState.state = CharacterState.State.DoneAttacking;
                }

                // if player left radius go back to pursue

                if (!CheckIfWithinAttackingDistance())
                {
                    characterState.state = CharacterState.State.Pursue;
                }

                break;

            case CharacterState.State.DoneAttacking:
                // so here we are going to attack again when done attacking,
                // but we will need to check whether the player is still
                // within the attack radius
                // we want some sort of pause so we don't keep spamming attacks,
                // and give the player a chance to attack

                // last todo here: create a pause between attacks

                if (CheckIfWithinAttackingDistance())
                {
                    // characterAnimation.ResetAnimation();
                    ChooseRandomAttack();

                    // characterState.state = CharacterState.State.Attacking;
                }

                else
                {
                    if (!animator.GetBool("attackDone")) return;

                    // pause

                    characterState.state = CharacterState.State.Pause;

                    // previous state transition, now we will transition from pause to pursue

                    // characterState.state = CharacterState.State.Pursue;
                }
                
                break;

            case CharacterState.State.Pause:

                StartTimer();

                if (time > RollDecimalDice(0.0f, 3.0f))
                {

                    ResetTimer();

                    characterState.state = CharacterState.State.Pursue;
                }

                break;


            case CharacterState.State.TookDamage:

                // pause when took damage

                characterNavMesh.StopAgent();

                characterKnockback.PerformKnockback();

                StartTimer();

                if (time > RollDecimalDice(1.0f, 3.0f))
                {
                    ResetTimer();

                    characterState.state = CharacterState.State.Pursue;
                }

                break;

            case CharacterState.State.PlayerDefeated:

                ReturnToOriginalPosition();

                break;

            case CharacterState.State.Death:

                CleanUp();

                characterNavMesh.StopAgent();

                // ensures we just run the Death state once

                GameState.instance.numberOfAttackingEnemies -= 1;
                

                characterState.state = CharacterState.State.PostDeath;

                break;

            case CharacterState.State.PostDeath:

                break;
        }
    }

    void CleanUp()
    {
        // remove health bars

        healthBar.SetActive(false);

        // remove colliders

        foreach (Collider collider in character.GetComponents<Collider>())
        {
            collider.enabled = false;
        }

        // clear state text if turned on 

        stateText.SetActive(false);
    }

    float RollDecimalDice(float start, float end)
    {
        float result = UnityEngine.Random.Range(start, end);;
        // Debug.Log(result);

        return result;
    }

    int RollDice(int start, int end)
    {
        return UnityEngine.Mathf.RoundToInt(UnityEngine.Random.Range(start, end));
    }
}

```

# Input System

The game supports Playstation, Xbox, and other generic gamepad controls.

# Version Control




I use Github and git for version in 90% of all cases. However, with this open world game I moved over to Plastic SCM.

I did try LFS with Git, and while the solution can work for large projects, I decided to give Plastic SCM a try, the version control company Unity recently purchased.

I will continue using Github for the majority of projects, but with checking very large files for an open world game Plastic SCM seems like a more dedicated solution. In the future I would also like to try Perforce.

Plastic SCM is built right into Unity, so the workflow is fast. However for smaller projects, I will continue to use git, because git works well for 99% of cases. The only reason I would consider using Plastic SCM is when working with extremely large file sizes, which git was just simply not built for.


![Plastic SCM](https://github.com/pjkw/Portfolio/blob/main/images/Plastic%20SCM.png)


