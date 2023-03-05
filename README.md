# C# and Unity Portfolio

Hi! This is my C# and Unity portfolio.

Here are a few code samples from the project. The goal is to always keep everything modular with the least amount dependencies, but also to treat data as a "single source of truth" that lives in one place.

## Inventory Setup

```cs
using UnityEngine;
using UnityEngine.UI;

public class ItemData : MonoBehaviour
{
    public int itemId;
    public string itemName;
    public string itemDescription;
    public Sprite itemSprite;
    public int maxQuantity;
    public int damage;
    public int armorClass;
    public Image itemImage;

    public void SetInfo(int id, string name, string description, Sprite sprite, int quantity, int dmg, int ac)
    {
        itemId = id;
        itemName = name;
        itemDescription = description;
        itemSprite = sprite;
        maxQuantity = quantity;
        damage = dmg;
        armorClass = ac;
        itemImage.sprite = itemSprite;
    }
}
```

```cs
using UnityEngine;
using UnityEngine.UI;

public class ItemSlotData : MonoBehaviour
{
    public int slotNumber;
    public int itemId;
    public string itemName;
    public Sprite itemIcon;
    public int quantity;
    public Image image;
    public bool isAvailable;
    
    public ItemSlotData(int itemId, string itemName, Sprite itemIcon, int quantity, Image image, bool isAvailable, int slotNumber)
    {
        this.itemId = itemId;
        this.itemName = itemName;
        this.itemIcon = itemIcon;
        this.quantity = quantity;
        this.image = image;
        this.isAvailable = isAvailable;
        this.slotNumber = slotNumber;
    }
}
```

## Combat AI

Still a work in progress, but the goal is to set up a modular character controller that can be attached as a component to any character.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public static class CharacterSettings
{
    public static float normalAgentSpeed = 2.0f;
    public static float agentPursueRunSpeed = 3.5f;
    public static Quaternion targetRotation;
    public static float pursueDistance = 37.0f;
    public static float pursueRadius = 37.0f;
    public static float lowerBoundSpeedIncrease = 2.0f;
    public static float upperBoundSpeedIncrease = 4.0f;
    public static float coolDownLowerBound = 5.0f;
    public static float coolDownUpperBound = 20.0f;
    public static float cooldownTime; // this gets set by AttackingState, which sets things once (the only state that runs once on frame), and then immediately passes to DoneAttacking which needs this static value to check, since DoneAttacking runs continously
}

public class CharacterController : MonoBehaviour
{
    public Transform playerTransform;
    
    [SerializeField] float pursueDistance = 37.0f;
    [SerializeField] float pursueRadius = 37.0f;
    [SerializeField] StateText stateText;
    [SerializeField] NavMeshAgent navMeshAgent;
    [SerializeField] FieldOfView fieldOfView;
    [SerializeField] CharacterAnimationAPI characterAnimationAPI;
    [SerializeField] Pathfinding pathfinding;

    public CharacterState characterState;

    void Start()
    {
        TransitionToState(CharacterState.State.Idle);
        stateText.SetText1("Idle");
    }

    void Update()
    {
        characterState.UpdateState();
        UpdatePlayerDistanceText();
    }

    void UpdatePlayerStateText()
    {
        stateText.SetText2(characterState.ToString());
    }

    void UpdatePlayerDistanceText()
    {
        stateText.SetText1(Mathf.Round(Vector3.Distance(transform.position, playerTransform.position)).ToString() + "m");
    }

    public CharacterState GetState()
    {
        return characterState;
    }

    public void TransitionToState(CharacterState.State state)
    {
        switch (state)
        {
            case CharacterState.State.Idle:
                characterState = new IdleState( this,
                                                playerTransform,
                                                navMeshAgent,
                                                characterAnimationAPI,
                                                fieldOfView);
                stateText.SetText2("Idle");
                break;
            case CharacterState.State.Pursue:

                characterState = new PursueState( this,
                                                playerTransform,
                                                navMeshAgent,
                                                characterAnimationAPI,
                                                fieldOfView);
                stateText.SetText2("Pursue");
                pathfinding.disablePathfinding = true;
                break;
            case CharacterState.State.Attacking:

                characterState = new AttackingState( this,
                                                playerTransform,
                                                navMeshAgent,
                                                characterAnimationAPI,
                                                fieldOfView);
                stateText.SetText2("Attacking");
                break;
            case CharacterState.State.DoneAttacking:
                stateText.SetText2("DoneAttacking");
                characterState = new DoneAttackingState(this, playerTransform, navMeshAgent, characterAnimationAPI, fieldOfView);
                break;
        }
    }
}

public abstract class CharacterState
{
    protected CharacterController characterController;
    protected Transform playerTransform;
    protected NavMeshAgent navMeshAgent;
    protected CharacterAnimationAPI characterAnimationAPI;
    protected FieldOfView fieldOfView;

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

    public CharacterState(  CharacterController characterController,
                            Transform playerTransform,
                            NavMeshAgent navMeshAgent,
                            CharacterAnimationAPI characterAnimationAPI,
                            FieldOfView fieldOfView)
    {
                            this.characterController = characterController;
                            this.playerTransform = playerTransform;
                            this.navMeshAgent = navMeshAgent;
                            this.characterAnimationAPI = characterAnimationAPI;
                            this.fieldOfView = fieldOfView;
    }

    public abstract void UpdateState();

    protected void TransitionToState(State state)
    {
        characterController.TransitionToState(state);
    }
}

public class IdleState : CharacterState
{
    public IdleState(   CharacterController characterController,
                        Transform playerTransform,
                        NavMeshAgent navMeshAgent,
                        CharacterAnimationAPI characterAnimationAPI,
                        FieldOfView fieldOfView)

             : base (   characterController,
                        playerTransform,
                        navMeshAgent,
                        characterAnimationAPI,
                        fieldOfView)
    {
        
    }

    public override void UpdateState()
    {
        if (Vector3.Distance(characterController.transform.position, playerTransform.position) < CharacterSettings.pursueDistance
            && fieldOfView.IsPlayerVisible())
        {
            TransitionToState(State.Pursue);
        }
        else
        {
            TransitionToState(State.Idle);
        }
    }
}


public class PursueState : CharacterState
{
    NavMeshPath path;
    float attackDistance = 5.0f;
    float timer = 0.0f;

    public PursueState( CharacterController characterController,
                        Transform playerTransform,
                        NavMeshAgent navMeshAgent,
                        CharacterAnimationAPI characterAnimationAPI,
                        FieldOfView fieldOfView)

             : base(    characterController,
                        playerTransform,
                        navMeshAgent,
                        characterAnimationAPI,
                        fieldOfView)
    {
        Initialize();
    }

    void Initialize()
    {
        navMeshAgent.isStopped = false;
        path = new NavMeshPath();
        navMeshAgent.speed = CharacterSettings.agentPursueRunSpeed; // we will also modify this in FollowPlayer(), but start with this
        characterAnimationAPI.PlayAnimation("run", 0.2f);
        timer = 0.0f;
    }

    bool IsPlayerWithinPursuitRadius()
    {
        return Vector3.Distance(characterController.transform.position, playerTransform.position) <= ArachnidSettings.pursueRadius;
    }

    bool IsPlayerWithinAttackDistance()
    {
        return Vector3.Distance(characterController.transform.position, playerTransform.position) < attackDistance;
    }

    void FollowPlayer()
    {
        float distanceToPlayer = Vector3.Distance(characterController.transform.position, playerTransform.position);

        if (distanceToPlayer >= 8f && distanceToPlayer <= 12f)
        {
            // here we can run to the player
        }
        else
        {
            // here walk to the player
        }
        navMeshAgent.CalculatePath(playerTransform.position, path);
        navMeshAgent.SetPath(path);
    }

    void IncreaseAnimationSpeed(float speedIncrease)
    {
        characterAnimationAPI.SetAnimationSpeed(1.0f + speedIncrease);
    }

    void IncreaseAgentSpeed(float speedIncrease)
    {
        navMeshAgent.speed = CharacterSettings.normalAgentSpeed * (1.0f + speedIncrease);
    }

    public override void UpdateState()
    {
        if (!IsPlayerWithinPursuitRadius())
        {
            TransitionToState(State.Idle);
        }

        else if (IsPlayerWithinAttackDistance())
        {
            TransitionToState(State.Attacking);
        }
        
        else
        {
            FollowPlayer();
        }
    }
}


public class AttackingState : CharacterState
{
    private float attackDistance = 5.0f;
    private float dashDistance = 3.0f;
    private float dashDuration = 0.25f;
    private float timer = 0.0f;
    private float cooldown = 5.0f;
    private float cooldownTimer = 0.0f;
    private Vector3 dashStartPosition;
    private bool isDashing;
    bool cooldownTimerDone = false;


    public AttackingState(CharacterController characterController, Transform playerTransform, NavMeshAgent navMeshAgent, CharacterAnimationAPI characterAnimationAPI, FieldOfView fieldOfView) : base(characterController, playerTransform, navMeshAgent, characterAnimationAPI, fieldOfView)
    {
        navMeshAgent.isStopped = true;
        isDashing = true;
        cooldownTimer = cooldown;
        Initialize();
    }

    void Initialize()
    {

        // determine cool down first, since we run this once,
        // and we don't want to keep determining cool down in DoneAttacking, which runs on frame, so will quickly reach any number
        // so let's set this once here, and then pass down

        CharacterSettings.cooldownTime = Random.Range(CharacterSettings.coolDownLowerBound, CharacterSettings.coolDownUpperBound);

        float rand = Random.value;

        if (rand < 0.8f)
        {
            characterAnimationAPI.PlayAnimation("attack1", 0.5f); // 80% chance
        }
        
        else if (rand < 0.91f)
        {
            characterAnimationAPI.SetAnimationSpeed(1.0f + 10.0f);
            characterAnimationAPI.PlayAnimation("attack2", 0.5f); // 11% chance
        }
        
        else
        {
            characterAnimationAPI.PlayAnimation("attack3", 0.5f); // 9% chance
        }

        ResetAgentSpeed();
        ResetAnimationSpeed();
        DetermineWhetherToAttackOrRest();
    }

    void DetermineWhetherToAttackOrRest()
    {
        int roll = Random.Range(1, 7);
        if (roll > 2)
        {
            // Attack();
        }
        else
        {
            // Rest();
        }
    }

    void Rest()
    {
        TransitionToState(State.Pause);
    }

    void ResetAnimationSpeed()
    {
        characterAnimationAPI.SetAnimationSpeed(1.0f);
    }

    void ResetAgentSpeed()
    {
        navMeshAgent.speed = CharacterSettings.normalAgentSpeed;
    }

    public override void UpdateState()
    {
        if (Vector3.Distance(characterController.transform.position, playerTransform.position) > attackDistance)
        {
            // i f the player is out of range, go back to the Pursue state

            navMeshAgent.isStopped = false;
            navMeshAgent.SetDestination(playerTransform.position);
            cooldownTimer = 0.0f;
            TransitionToState(State.Pursue);
        }

        // if the player is still within the radius, attack again

        if (Vector3.Distance(characterController.transform.position, playerTransform.position) < attackDistance)
        {
            // done attacking pauses and handles cooldown time,
            // afterwards calling State.Attack again so we can attack again (or do a randomized series of attack actions later)

            // we set the rotation once,
            // which we then carry out over time in DoneAttacking
            // that way we play fair with the player,
            // instead of the creature continuously turning to face the player

            SetRotation(); 
            TransitionToState(State.DoneAttacking);
        }
    }

    void SetRotation()
    {
        Vector3 directionToPlayer = (playerTransform.position - characterController.transform.position).normalized;
        
        directionToPlayer.y = 0f;

        if (directionToPlayer != Vector3.zero)
        {
            CharacterSettings.targetRotation = Quaternion.LookRotation(directionToPlayer);
        }
    }
}

public class DoneAttackingState : CharacterState
{
    private float timer = 0.0f;
    private float cooldownTime;
    private float pursuitRadius = 7.0f;


    private float attackDistance = 5.0f;
    private float rotationSpeed = 5.0f;

    float dashDuration = 1.0f;
    float dashDistance = 5.0f; // in front of the player, so 1 means 1 unit in front of the player
    float dashSpeed = 5.0f;

    public DoneAttackingState(  CharacterController characterController,
                                Transform playerTransform,
                                NavMeshAgent navMeshAgent,
                                CharacterAnimationAPI characterAnimationAPI,
                                FieldOfView fieldOfView) :
                                
                        base(   characterController,
                                playerTransform,
                                navMeshAgent,
                                characterAnimationAPI,
                                fieldOfView)
    {
        timer = 0.0f;
    }

    public override void UpdateState()
    {
        Debug.Log("Done attacking state");

        timer += Time.deltaTime;

        if (Vector3.Distance(characterController.transform.position, playerTransform.position) > attackDistance)
        {
            MoveToPlayer();
            TransitionToState(State.Pursue);
        }

        else if (Vector3.Distance(characterController.transform.position, playerTransform.position) < attackDistance)
        {
            RotateToPlayer();

            // starts cool down is what ends up triggering the attack again by going into State.Attacking

            // if we are within the attack distance, then let's lunge toward the player

            DashToPlayer();
            StartCooldown();
        }
    }

    void DashToPlayer()
    {
        if (timer < dashDuration)
        {
            Vector3 targetPosition = playerTransform.position + (characterController.transform.position - playerTransform.position).normalized * dashDistance;
            characterController.transform.position = Vector3.Lerp(characterController.transform.position, targetPosition, Time.deltaTime * dashSpeed);
        }
    }

    void MoveToPlayer()
    {
        navMeshAgent.isStopped = false;
        navMeshAgent.SetDestination(playerTransform.position);
    }

    void RotateToPlayer()
    {
        characterController.transform.rotation = Quaternion.Slerp(characterController.transform.rotation, ArachnidSettings.targetRotation, Time.deltaTime * rotationSpeed);
    }

    // main to restart attack after cool down, so this is the transition from State.DoneAttacking to State.Attacking

    void StartCooldown()
    {
        navMeshAgent.isStopped = true;

        // this cooldownTime is set every once in a while when AttackingState runs once

        if (timer > 2.0f)
        {
            TransitionToState(State.Attacking);
            timer = 0.0f;
        }
    }
}
```



RPG vertical slice:

![](https://github.com/pjkw/Portfolio/blob/main/gifs/Menu%20gif.gif)

https://imgur.com/a/WNYbRQp

![](https://github.com/pjkw/Portfolio/blob/main/gifs/Port%20Intro%20gif.gif)

![](https://github.com/pjkw/Portfolio/blob/main/gifs/Ship%20gif.gif)

https://imgur.com/a/0yIFrTF

![](https://github.com/pjkw/Portfolio/blob/main/gifs/camp.gif)

https://imgur.com/a/joe5BFu

Environment design of the camp: https://imgur.com/a/nEKwRzF


![](https://github.com/pjkw/Portfolio/blob/main/gifs/Dungeon%20intro%20gif.gif)

![](https://github.com/pjkw/Portfolio/blob/main/gifs/Dungeon%20combat%202.gif)

https://imgur.com/a/6XOoBB3

![](https://github.com/pjkw/Portfolio/blob/main/gifs/Beach%20Monster.gif)

https://imgur.com/a/0yIFrTF

![](https://github.com/pjkw/Portfolio/blob/main/images/Port%203.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Port%201.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/town%20square.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/House.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Castle%20Horizon.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Hilltop.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/terrain%202.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Terrain%203.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Port%20Blockout.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Ramp.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Supports.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Supports%20Ocean%20Floor.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/Dock%20Platform.png)

![](https://github.com/pjkw/Portfolio/blob/main/gifs/Dock%20Blocked%20Out.gif)

![](https://github.com/pjkw/Portfolio/blob/main/images/World%20Design%201.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/World%20Design%202.png)

![](https://github.com/pjkw/Portfolio/blob/main/images/World%20Design%203.png)

![](https://github.com/pjkw/Portfolio/blob/main/gifs/World%20Gif%20Optimized.gif)

https://imgur.com/a/WX9sHFf

# Older Portfolio

Here is my older portfolio: https://imgur.com/a/BSXfZ7Z
