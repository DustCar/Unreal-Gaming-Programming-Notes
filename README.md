# Unreal-Gaming-Programming-Notes

Here will be all of the notes that will be important for me to retain information about gameplay and using Unreal Engine 5.2

## Unreal

---
### RootComponent (some general USceneComponent notes too)
Component that is part of any new actor. It is classified as a USceneComponent and can be replaced by other Components that also derives from USceneComponent. It includes a tranform and does not include a visual representation. Can attach other components to USceneComponents.

---
### Constructing Components
`CreateDefaultSubobject<{type}>(TEXT({name}))`

A Template function that constructs a component of the specified type in the angle brackets. The parameter in the parantheses would be the name of said subobject. Returns a pointer to the component set as specified type. 

*Ex: CreateDefaultSubobject<UCapsuleComponent>(TEXT("Capsule")) --> returns UCapsuleComponent\**

#### Forward Declaration
Useful when needing a specific class to declare a variable type without including a separate header file. Avoids having to copy loads of code into the file needing the class, but is limited in that it does not allow access to members, use of functions, and constructing objects of the class, making it an incomplete type. To access members/use functions/construct objects, then the header file is needed. 
```
#include ...

class {ClassName}
```
*Note: Best practice to avoid including a lot of header files in header files. Only include header files where needed, mostly in cpp files. Use Forward Declaration for header files for type declaring.*

#### Declaring variables for components in header file

Before creating the component, declare the variable in the header file and provide the macro UPROPERTY() before the declaration. In the instance that a component is not included as a part of the file's class then Forward Declaration is needed. For example, in an actor class, UCapsuleComponent needs to be forward declared, while UStaticMeshComponent does not. This is because UStaticMeshComponent is included in the actor class.

```
UPROPERTY()
(class) ComponentType* VarName;
```

Components declared with a UPROPERTY() macro with empty parentheses would have its properties hidden in the Details panel when working on a blueprint derived from the C++ classes. In order to see the details of the blueprint's components then some setting, like below, are needed within the C++ class in order to see and edit the properties in the Details panel in the editor.
- VisibleAnywhere : is *visible* in the *bp editor* and *world editor*, but **cannot** be edited.
- EditAnywhere : is *visible* and can be *edited* in the *bp editor* and *world editor*.
- VisibleInstanceOnly : is **not visible** in the *bp editor*, but is *visible* in the *world editor* when an **INSTANCE** of the blueprint is dragged in.
- VisibleDefaultsOnly : is *visibile* in the *bp editor*, but **not visible** in the *world editor*.
- EditInstanceOnly : is **not visible/editable** in the *bp editor*, but is *visible/editable* in the *world editor* when an **INSTANCE** is dragged in.
- EditDefaultsOnly : is *visibile/editable* in the *bp editor*, but **not** in the *world editor*.

  *Note: For component types, like UStaticMeshComponent, it is best to use VisibleAnywhere to assign a mesh since EditAnywhere would try to change the UStaticMeshComponent pointer itself rather than changing the static mesh only.*

As for exposing components to the Event Graph of the blueprint editor, these parameters would be included within the parentheses:
- BlueprintReadWrite : gives access to two nodes in the event graph, which are a getter and a setter for the specified variable.
- BlueprintReadOnly : gives access to ONLY the GETTER node for the variable.

*Note: Both parameters above **cannot** be used to expose private members.*

To expose private members for blueprints, use the `meta = (AllowPrivateAccess = "true")` in UPROPERTY().

To organize components within the Details tab, use the `Category = "{title}"` parameter.

#### Creating your own components for internal details (i.e. Health, Currency, Stats)
*Note: I'm sure the Gameplay Ability System can easily accomplish this too but this intentially skips GAS just to cover different cases.*
This best done using the *Actor Component* class, rather than *Scene Component*, as a base since there is no need for transform or attachments.

The way to setup this component in C++ is just like how it is stated in this section, using `CreateDefaultSubobject<>()` and declaring its definition.
Once the class is created, it automatically has a UCLASS macro that allows it to be used in Blueprints, so if needed then you can easily just add your new component to any blueprint, if you don't want to declare it in C++.

---
### Unreal Enhanced Input System (EIS)
Before starting, make sure that the Enhanced Input Plugin is enabled under the *Edit->Plugins* menu. After restarting editor, change the default input classes within *Edit->Project Settings*. Go to the input section under **Engine** and look for Default Classes. Set *Default Player Input Class* to *EnhancedPlayerInput* and set *Default Input Component Class* to *EnhancedInputComponent*.

Make sure that the plugin is included within the .uproject file (name is "EnhancedInput") and within the .Build.cs file (add "EnhancedInput" to PublicDependencyModuleNames line.

Useful Console Commands:
- showdebug enhancedinput

#### Main Concepts
- **Input Actions (IA)** : Link between EIS and project code. Separate from raw input and focuses on its current state, returning an input value based on three independent floating-point axes.
- **Input Mapping Contexts (IMC)** : Maps user inputs to IAs and can be dynamically added, removed, or prioritized for each user. Can be applied one or more times through a local player's Enhanced Input Local Player Subsystem.
- **Modifiers** : Adjusts the value of raw input coming from user's devices. IMCs can have any number of modifiers associated with each raw input for an IA. Examples: Dead zones, input smoothing. Custom modifiers can also be created.
- **Triggers** : Uses post-Modifier input values, or output magnitudes of other IAs, to determine whether an IA should activate. Any IA can have one or more Triggers for each input.

#### Using/Setting up EIS using C++
original article: <nightails.com/2022/10/16/unreal-engine-enhanced-input-system-in-c>

After ensuring EIS is added and enabled, first task would be to set up the IMC within the character header and establishing SetupPlayerInputComponent in the cpp file.
 - add a UInputMappingContext* as a protected member in *Character.h*
 - in the cpp file, set up SetupPlayerInputComponent. If anything is default set, other than `Super::SetupPlayerInputComponent`, then remove. We'll get back to it later on, for now is just setup.
   1. Get player controller using `GetController()`. Make sure to Cast to make sure you are retrieving an APlayerController object.
   2. Get enhanced input local player subsystem from player controller using `ULocalPlayer::GetSubsystem<>()`.
   3. Clear mappings using `ClearAllMappings()` then add the Input Mapping context from header file with `AddMappingContext()`.

Next is to set up the IAs. This could be done individually by creating separate *UInputAction\** variables but would become tedious with more actions. 
For better management, use a config file to hold multiple actions. This config file would subclass **DataAsset** and can hold as many actions as wanted.

After creating the **InputConfig** file, make sure to add `#include "InputAction.h"` in the header file. Then add all necessary/wanted *UInputActions* as **public** members with the preferred properties of *EditDefaultsOnly* and *BlueprintReadOnly*. 
Then, add the config file as a protected member of the Character class, just like the IMC from earlier.

Now we go back to the *SetupPlayerInputComponent* function to bind functions to the actions listed in the header file.
To do so:
 1. Get the *EnhancedInputComponent* by casting it from the *PlayerInputComponent*
 2. Bind the actions to the functions that would be made soon using *BindAction*.

*Note: Make sure to add `#include "EnhancedInput/Public/EnhancedInputComponent.h"` and `#include "*ConfigData.h"` as headers.*

Finally, declare the input functions that were bounded to earlier in the header file and then create the functions in the cpp file.
 - When declaring the functions, it should should be **void**.
 - For functions that require an input value, like for movement or anything vector based, it must have a ***const** FInputActionValue&* as a parameter
Within the cpp file, make sure to add `#include "InputActionValue.h"` as a header.

**The input functions that are bounded are the main core of functionality. Whatever is placed in those functions will run whenever the input is achieved.**

*Note: When using the Value parameter in the action functions, Value itself doesn't do much. It is a struct and has get methods that you can use to actually get an input action value.*

---
### General notes for writing code for Input Action functions
#### Pawns and movement
When adding inputs for *Pawns*, especially for movement, it is important to note that *Pawns* **DO NOT** have a movement component that automatically handle the input and move.

To move a *Pawn* with inputs, there are two ways (AFAIK on March 9th, 2024):
- Move the *Pawn* using its location:
  Simple (maybe naive) way:
  1. Within your move function, Create an *FVector* variable that holds a zero vector.
  2. Use `Value.Get<FVector2D>()` and a dot operator to grab the X (A and D) or Y (W and S) input action value.
  3. Assign the value to whatever direction you need the pawn to go to the variable you made on step 1. i.e. VectorVariable.X = step 2 var -> moves in X direction
  4. Use `AddActorLocalOffset()` and pass the vector variable in to move each tick. *Note: use local offset to move in the direction of the pawn and not with the world axis.*
     
  Using the above method creates very slow, and inconsistent, movement since the amount of movement is set with the value of the input action (1.0) and is based on framerate.
  To fix this, it is best to scale the value that is being assigned by a Speed variable and DeltaTime. i.e. `Value * Speed * DeltaTime`.
  - The Speed variable can be a private member that is a float
  - Since the function is not used within Tick then it is likely that it does not have access to DeltaTime yet. To access DeltaTime, use `UGameplayStatics::GetWorldDeltaSeconds(this)`.

- Add the *Floating Pawn Movement* to *Pawn* BP and use *AddMovementInput* as you would with *Characters*:
  1. Within the move function, use `Value.Get<FVector2D>()` and a dot operator to grab the X (A and D) or Y (W and S) input action value.
  2. Get the forward vector of the pawn using `GetActorForwardVector()`.
  3. Use `AddMovementInput()` and pass in the Forward vector, then the value.

---
### Projectiles
#### Projectile Movement
Three ways to implement projectile movement:
- Manually set location/rotation. Implemented using Tick.
- Add an impulse. Enable/Use physics/physics system.
- Attach a movement component. Unreal has an already built Projectile Movement Component.

**Using UProjectileMovementComponent**
documentation: <https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/GameFramework/UProjectileMovementComponent?application_version=5.2>

include: `#include "GameFramework/ProjectileMovementComponent.h"`

To add the component is the same as any other component:
1. Add pointer declaration in header with UPROPERTY() for editor usage.
2. Use CreateDefaultSubobject<>() in cpp to construct the component.
3. Set any variables/attachments.

---
### Delegates (Events)
*Occurs with UPrimitiveComponents (inherited classes too).*

#### Hit Events
Delegate `OnComponentHit`: Event called when a component hits (or is hit by) something solid.
- Returns a struct called `FComponentHitSignature`.
- Considered a **Multicast Delegate**.

`OnComponentHit.AddDynamic(User Object, Callback Function)`: used to add functions to the invocation list

Common callback function(s) used with hit events:
```
UFUNCTION()
void OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);
```
- Allows for access to the other actor/component for different on hit scenarios. Includes affect based on physics and information for hit result.

#### Damage Events
Generated by `UGampelayStatics::ApplyDamage`

Delegate `OnTakeAnyDamage`: Event called when the actor takes damage in any way.
- Returns a struct called `FTakeAnyDamageSignature`
- Considered a **Multicast Delegate**

Common callback function(s) used with damage events:
```
UFUNCTION()
void DamageTaken(AActor* DamagedActor, float Damage, const UDamageType* DamageType, AController* Instigator, AActor* DamageCauser);
```
- Applies damage to the actor getting damaged with damage value and type.

**Multicast Delegate**: A delegate where multiple functions can be bound to it and respond to that event. 
- Functions are added to an *Invocation List*. 
- *Broadcast* to functions in *Invocation List* when something happens for response.

---
### Template Functions
`TSubClassOf<type>`
- Used for type safety. Forces designers to use a derived UClass or subclass of set Class. It runs a check on compilation and would return a compile error if the *type* is not a subclass of the specified UClass type.

Why not use `UClass*`?
- With `UClass*`, it is possible to assign the `UClass*` with any UClass rather than only being a set type of UClass. In addition, when assigning the variable, it would do a check at runtime and return a *nullptr* on failure.

Extra information: <https://forums.unrealengine.com/t/why-use-tsubclassof-and-not-just-the-class-itself/365690>

`TSoftObjectPtr<type>`
- Used for referencing objects which might or might not be loaded via path. Can point to actors within a level even if they are not loaded. Can be loaded **ASYNCHRONOUSLY** and does **NOT** load its value into memory.

`UWorld::SpawnActor<type>(UClass, Location, Rotation)`
- Used to spawn actors into the world. Using a `TSubclassOf<>` UClass allows you to pass in a BP in for UClass to spawn in actors with all necessary components.
- To use, type `GetWorld()->SpawnActor<>()` or any other var/func that returns a UWorld object.

