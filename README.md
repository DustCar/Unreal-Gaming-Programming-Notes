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

Components declared with a UPROPERTY() macro with expty parentheses would have its properties hidden in the Details panel when working on a blueprint derived from the C++ classes. In order to see the details of the blueprint's components then some setting, like below, are needed within the C++ class in order to see and edit the properties in the Details panel in the editor.
- VisibleAnywhere : is *visible* in the *bp editor* and *world editor*, but **cannot** be edited.
- EditAnywhere : is *visible* and can be *edited* in the *bp editor* and *world editor*.
- VisibleInstanceOnly : is **not visible** in the *bp editor*, but is *visible* in the *world editor* when an **INSTANCE** of the blueprint is dragged in.
- VisibleDefaultsOnly : is *visibile* in the *bp editor*, but **not visible** in the *world editor*.
- EditInstanceOnly : is **not visible/editable** in the *bp editor*, but is *visible/editable* in the *world editor* when an **INSTANCE** is dragged in.
- EditDefaultsOnly : is *visibile/editable* in the *bp editor*, but **not** in the *world editor*.

As for exposing components to the Event Graph of the blueprint editor, these parameters would be included within the parentheses:
- BlueprintReadWrite : gives access to two nodes in the event graph, which are a getter and a setter for the specified variable.
- BlueprintReadOnly : gives access to ONLY the GETTER node for the variable.

