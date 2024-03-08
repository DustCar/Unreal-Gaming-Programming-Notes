# Unreal-Gaming-Programming-Notes

Here will be all of the notes that will be important for me to retain information about gameplay and using Unreal Engine 5.2

## Unreal
### TSubClassOf<{class}>
`TSubClassOf<classType> SubclassVar`
Class pointer variable used to create new objects from. Editor's detail panel would list only classes derived from 'classType'. In blueprints, it would be a purple pin.
Source: https://forums.unrealengine.com/t/why-use-tsubclassof-and-not-just-the-class-itself/365690

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

// rest of code
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
After ensuring EIS is added and enabled, first task would be to set up the IMC within the character header and establishing SetupPlayerInputComponent in the cpp file.
 - add a UInputMappingContext* as a protected member in *Character.h
 - in the cpp file, set up SetupPlayerInputComponent. If anything is default set, other than `Super::SetupPlayerInputComponent`, then remove. We'll get back to it later on, for now is just setup.
   1. Get player controller
   2. Get local player subsystem from player controller. (remember to add scope ULocalPlayer when using GetSubsystem<>().
   3. Clear mappings then add the Input Mapping context from header file.

Next is to set up the IAs. This could be done individually by creating separate *UInputAction\** variables but would become tedious with more actions. 
For better management, use a config file to hold multiple actions. This config file would subclass **DataAsset** and can hold as many actions as wanted.

After creating the file, make sure to add `#include "InputAction.h"` in the header file. Then add all necessary/wanted *UInputActions* as **public** members with the preferred properties of *EditDefaultsOnly* and *BlueprintReadOnly*. 

*Make sure to add the config file as a protected member of the character class, just like the IMC from earlier.*

Now we go back to the *SetupPlayerInputComponent* function to bind functions to the actions listed in the header file.
To do so:
 1. Get the *EnhancedInputComponent* by casting it from the *PlayerInputComponent*
 2. Bind the actions to the functions that would be made soon using *BindAction*.

*Note: Make sure to add `#include "EnhancedInput/Public/EnhancedInputComponent.h"` and `#include "*ConfigData.h"` as headers.*

Finally, declare the input functions that were bounded to earlier in the header file and then create the functions in the cpp file.
When declaring the functions, it should have a ***const** FInputActionValue&* as a parameter and should be **void**.
Within the cpp file, make sure to add `#include "InputActionValue.h"` as a header.
