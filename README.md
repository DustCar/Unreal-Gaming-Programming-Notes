# Unreal-Gaming-Programming-Notes

Here will be all of the notes that will be important for me to retain information about gameplay and using Unreal Engine 5.2

## Unreal
### TSubClassOf<{class}>
`TSubClassOf<classType> SubclassVar`
Class pointer variable used to create new objects from. Editor's detail panel would list only classes derived from 'classType'. In blueprints, it would be a purple pin.
Source: https://forums.unrealengine.com/t/why-use-tsubclassof-and-not-just-the-class-itself/365690

### RootComponent (some general USceneComponent notes too)
Component that is part of any new actor. It is classified as a USceneComponent and can be replaced by other Components that also derives from USceneComponent. It includes a tranform and does not include a visual representation. Can attach other components to USceneComponents.

### Forward Declaration
Useful when needing a specific class to declare a variable type without including a separate header file. Avoids having to copy loads of code into the file needing the class, but is limited in that it does not allow access to members, use of functions, and constructing objects of the class, making it an incomplete type. To access members/use functions/construct objects, then the header file is needed. 
```
#include ...

class {ClassName}

// rest of code
```
*Note: Best practice to avoid including a lot of header files in header files. Only include header files where needed, mostly in cpp files. Use Forward Declaration for header files for type declaring.*

### Constructing Components
`CreateDefaultSubobject<{type}>(TEXT({name}))`
A Template function that constructs a component of the specified type in the angle brackets. The parameter in the parantheses would be the name of said subobject. Returns a pointer to the component set as specified type. *Ex: CreateDefaultSubobject<UCapsuleComponent>(TEXT("Capsule")) --> returns UCapsuleComponent\**

