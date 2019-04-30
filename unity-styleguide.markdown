Unity3D Style Guide
==============================

There's an exception to every rule -- this guide is a starting point for consistency / interoperability and can absolutely be changed in the future. 

Project Organization
--------------------

* Avoid spaces in file and folder names

### Assets Folder Structure

Projects can and will overflow with new versions of art assets, half-baked ideas, and scripts no longer used. Using version control can help to cut down on extra text-based files, and nicely organized folders can help to manage all other kinds of assets. 

As a starting point, we recommend the following folders:

Assets/
... Scripts/ 				(can be prefixed with an underscore if you want it at the top of the list)

... Editor/					(for editor scripts, has to be named "Editor")

... Prefabs/				

... Scenes/

... Graphics/  				(Or: Graphics2D and Graphics3D, Or: Materials and Textures)

... Imports/ 				(New assets that haven't been further organized)

Organizing graphics and all of their associated materials and etc is very important as they will probably be coming from a variety of different sources. Try to keep them as compartmentalized as possible. 


### Revisions, updates

C\# Coding Guidelines
---------------------

### Spacing

- Space between operators
```
x = 3 * 5; // nice
x=3*5; // harder to read when looking for something specific
```

- Space between function arguments
```
void Highlight(bool highlightOn, float duration){ ... }
```
as opposed to 

```
void Highlight(bool highlightOn,float duration){ ... } 
```

### Naming

* Function names should begin with a captial letter. 
* Class names should begin with a capital letter. 
* Parameter names should begin with a lowercase letter. 

* Don't be afraid to change a name that is found to be actively confusing. Find + replace is your friend!


### Braces

Visual Studio and general C\# standards call for braces to have their own line:
```
int Foo(int x)
{
	return x + 1;
}
```

That said, just try to stick to one style or the other (same line or separate line)

* Attempt to **always** include braces around the body of `if` statements, instead of a single statement with no braces:

```
if (condition)
	doAThing();
```

as this can easily be revised later to:

```
if (condition)
	doAThing();
	doSomethingElse(); // BUG: will be executed regardless of value of (condition).
```

Better:
```
	if (condition)
	{
		doAThing();
		doSomethingElse();
	}
```

### Constants



Unity Specific
---------------

### Versions

* Use Unity Hub to open projects to ensure you use a consistent version of Unity.

* Do not update a project to a new version without both a compelling reason **and** a hard backup of the project folder.

* Coordinate version decisions across the entire team prior to starting serious work on a project with multiple developers. 

### Reusable Scripts vs Bespoke Scripts

If a script is meant to be re-used in another project, or used in multiple contexts in the same project, additional care should be used to organize its file for later reading / maintaining. 

Try to order function definitions so that the script's public interface is toward the top, and private implementation details are written below.

Always be sure to keep Unity Lifecycle functions (`Awake()`, `Start()`, `Update()`, etc) toward the top of the file as well. 

Extremely short, 'bespoke' scripts that do *one* thing, or are used in one very specific context can be more relaxed in terms of style.

### Debugging

**Log statements**

For simple issues, a `Debug.Log()` here and there will help you diagnose what's going on. Just be careful not to leave tons of extraneous logs running in your program. Either remove them when no longer needed, comment them out, or have a DEBUG bool variable that controls whether the log statements are executed. 

Consider using `Debug.LogWarning()` and `Debug.LogAssertion()` for situations that warrant the extra attention. They will pop up as the yellow triangle and red circle logs, respectively. Warnings are at the level of "this *could* be bad", and assertions are for issues that seriously need to be resolved. 

**VS Code or other debuggers**

For more complicated situations, consider using debugging tools to step through your code, set breakpoints and all the rest of it. This will keep extraneous `Debug.Log()` statements from accumulating and will allow you to inspect more of the program state when the program pauses at whichever breakpoint. 

### Editor Scripts

- If you have a component with lots of public state that you want you or another dev / designer to be able to change on the fly and there are some serious stipulations for *how* the parameters all mesh together, it's time to write an editor script! You can write components that behave just like built-in components do which keep you from making incoherent choices in the inspector. (For example, hiding an option until a checkbox activating it is checked).

- Give editor scripts (themselves .cs files) a suffix of \_Editor.

Here's a simple example, it just makes a button available to call a function at runtime without having to clutter up an existing script with debug code.

```
using UnityEngine;
using UnityEditor;
using UnityEngine.Events;

[CustomEditor(typeof(DEBUG_UTIL))]  
public class DEBUG_UTIL_Editor : Editor
{
 	// the main function you need to define to tell the editor how to draw this type
    public override void OnInspectorGUI()  	
    {
    	// draws whatever would ordinarily be drawn w/o a custom Editor script.
        DrawDefaultInspector(); 
        DEBUG_UTIL debug = (DEBUG_UTIL)target;

        // Functions in GUILayout will allow you to include different labels and buttons and etc in your 
        // custom editor. In this case we have a button
        if (GUILayout.Button("LAUNCH CALLBACKS")) {
            debug.debug_event.Invoke();
        }
    }
}
```

### Component Access

Avoid heavy use of `GetComponent<>`, especially in `Update()`. Prefer to cache component references at the start of the program. 

```
// bad news: lots of garbage generated every frame:

	void Update()
	{
		GetComponent<MeshRenderer>().material.SetColor("_Color", Color.green); 
	}

// better:
	
	MeshRenderer mesh;

	void Awake()
	{
		mesh = GetComponent<MeshRenderer>(); // cache component
	}

	void Update()
	{
		mesh.material.SetColor("_Color", Color.green);
	}
```

(For what its worth in the above example, it is better to also cache the numerical id for the shader's Color field instead of looking it up every frame from a string!)

### Lifecycle Functions in MonoBehaviours


`Awake()`: Called once for each script instance before any other callback. Use it to set up references between objects. All MonoBehaviours will have their `Awake()` function called *before any* `Start()` function is called. 

`Start()`: Also called exaclty once. Can be used for initialization.

`Update()`: Called every frame (so, as fast as possible).

`FixedUpdate()`: Called at a fixed interval, making it more suitable for physics-related updates or anything else that needs updating on a more regular (but still fast) schedule.

`OnEnable()` and `OnDisable()`: Called when the *GameObject* is enabled or disabled. Useful for resetting things when a GameObject is toggled on or off. 

`OnGUI()`: Essentially the equivalent of `Update()` for GUI related things.

`OnDestroy()`: Called when a scene ends (unless an object is marked to not be destroyed), when the game ends, or (around) when an object is manually destroyed via the `Destroy()` function. 

### Coroutines

If you find yourself writing an `Update()` function with hundreds of lines, coroutines may help you break it up into more manageable chunks.

A coroutine is a function whose execution can be suspended somewhere in the middle, allowing the program to do something else and pick up where it left off later. They allow one to simulate parallel processing without actually having to worry about managing threads.

Coroutines are useful for functionality that runs on a set timer, on a schedule not covered by the normal Update functions, or else to break up a monolithic Update function into a few subsections which may or may not be active at the same time. Coroutines can even be set to continue indefinitely (in a deliberate infinite loop). 


**Ending Coroutines Prematurely**

Sometimes you might find it useful to stop a coroutine before it has a chance to complete according to whichever criteria it is set to. In order to stop a coroutine, you need to retain a reference to it in a `Coroutine` variable.

```
Coroutine routineRef;

...

void Foo(){
	routineRef = StartCoroutine(MondoRoutine());
}

IEnumerator MondoRoutine(){
	// do something immediately
	yield return new WaitForSeconds(3f);  // returns control to other parts of the program
	// do something after timer
}

void Stop(){
	if (routineRef != null)
	{
		StopCoroutine(routineRef); // ends coroutine prematurely
	}
}

```


### Public member variables

It can be tempting to make nearly every single variable in a project public in order to make it easier to pass around references to scripts and to easily access their data. 

Excessive use of public variables causes some of the following problems:

1.) Confusion for reader about which variables refer to internal or external state.
2.) Open-ended questions about *where* public variables can be altered (in this file? somewhere else?). 

Fortunately, there are some workarounds to get some of the benefits of public variables without all of the mess.

- If you want a variable to be accessible in the inspector, use the [SerializeField] decorator before it, and it will show up in the inspector even if it is `protected` or `private`. (This will always work for built-in types like `int` and `float` and `string`, as well as `Vector3`). 

- If you want other objects to be able to reference the variable, but don't want them to be able to change it, consider making the variable a Field and declaring a public getter method:
```
public float Volume {get; internal set;}
```

As we see above, Volume (capitalized per c\# convention) is accessible from outside the class it is defined in, but cannot be changed from outside the class it was defined in. Here we see *default* implementations of the getter and setter methods of a Field. It is possible to define your own getter and setter for a field.

### Referring to other GameObjects

- public GameObject reference

- Using the scene hierarchy

- The Find functions 

### Variable Declarations with Default Values

When writing a script, one often hard-codes some variable values out of convenience. e.g. `bool displayed = true;`. It is not strictly necessary to do an inline intialization of built-in types, as C# will initialize numeric types to 0, strings to the empty string, etc. 

There can be some trouble when supplying a initial value for a `public` (or private and serialized to inspector) variable that can be altered in the inspector. 

Say we had a script called MusicController.cs with the following declaration:

```
public float volume = 20f; // starting volume low
```

Changing this variable in the inspector will **not** change the starting value that we set in our script, and may confuse a reader who comes along later thinking mistakenly that the volume will always start at 20f. 

A clearer approach would be to set the default value in `MonoBehaviour.Reset()`

### Floating Point Precision

Unity uses floats (32 bit floating point numbers) basically everwhere where a real number is required (like in Vector3's). This is done because computation on doubles (64 bit floating point numbers) is much slower.

Most of what you have to worry about is remembering to write an 'f' after all real-number literals.
```
float f = 1.0; // compiler might get mad at you for double to float converstion

float f1 = 1.0f // all good
```

Generally speaking, try to keep your scene closer to the origin. As you move several thousands of units away, the amount of precision (representable values between each whole number) decreases notably. 

### Prefabs

Prefabs are essentially recipes for GameObjects (with whatever collection of components attached). 

You can update a prefab and have all changes be applied to each *instance* of the prefab that is in a scene. In this way, it is the equivalent of a *class*, which can have various instances. 

Prefabs can be instantiated at run-time through custom scripts or by dragging a prefab into a scene in the inspector. Be careful about instantiating either a tremendous number of new GameObjects or instantiating and destroying them at an extreme frequency (i.e. on a per-frame basis regardless of context). 

Storing a customized GameObject as a prefab can be a helpful way to indicate your design intentions (as it stores information on all the components of the object in a .prefab file which is kept in the assets folder), even if the object is meant to be used only one time per scene. 

### ScriptableObjects


### Scenes

- It can be nice to make separate scenes to try out new ideas. 
- Newer versions of Unity store scenes as YAML files, where older ones had binary .scene files. This makes tracking changes in version control systems *easier* but still not foolproof. When making serious changes, duplicating a scene can be helpful. 

### Scene Hierarchy

- If a 2D object is lower than another in the scene hierarchy, then it is drawn on top of that object in z-depth of a canvas. 
- Same goes for children in a scene hierarchy - if they are in a canvas, they will be drawn on top of their parent.

- Keep the scene organized. If you need to create dummy GameObjects to group related GameObjects together, do so. 

- Attach global or wide-reaching scripts in easy to find locations.
	- A favorite is to attach them to the scene's main camera. 

### Events and UnityEvents

Events are powerful tools for de-coupling the request for some action to occur and the doing of whatever work. One piece of code conceputally yells out "hey, someone clicked the red button", and doesn't have to concern itself with all of the other pieces of code that may or may not respond to that message. 

Your garden-variety functions like `Update()` and `Start()` are conceptually events - they are just called on a very particular schedule that is set in stone by the game engine. Other events that you define can be called on any kind of schedule or preconditions that you can think of. 

**UnityEvents**

The most visible example of a UnityEvent is the `onClick()` function of the Unity `Button` class. When you edit a `Button` in a scene, you'll see that there is an expandable list of things that can be set to happen when the button is clicked. These are pairs of {object, function}. (Functions in this case include setter functions to set the value of a variable). 

Most UnityEvents are simple function calls with parameters set at compile-time. It *is* possible, however, to have an argument passed in at run-time. A basic example is the `Slider` class, which has a UnityEvent called `OnValueChanged(float value)` which has a float parameter that can be supplied at runtime by the `Slider` class.

It is absolutely possible to add listeners (functions to be called) and to `Invoke()` all listeners attached to a UnityEvent all within scripts. 


**C\# Events**

C\# events are more versatile than UnityEvents, but also easier to misuse. They can't be serialized to the inspector, so all of their effects are confined to custom scripts. 

- The first step of using c\# events is creating a *delegate*. A delegate is the equivalent of a type-safe *function pointer* if you've ever programmed in C. A delegate establishes the allowable parameter types and return type for a function that you might want to use *as an object* - e.g. as a parameter for another function.

	- A tiny example: passing an operation to a generic function that does something to two ints and return the result.

```
	public delegate int MathDelegate(int, int);
```

With the above definition we could define a generic function that uses a `MathDelegate` and functions we could pass it that comply with this type signature.

```
int GenericOperation(int lhs, int rhs, MathDelegate fn)
{
	return fn(lhs, rhs);
}

int Add(int lhs, int rhs)
{
	return lhs + rhs;
}

int Mult(int lhs, int rhs)
{
	return lhs * rhs;
}
```

We could then call `GenericOperation` as follows:

```
int x = GenericOperation(1, 3, Add); // x == 4

int y = GenericOperation(6, 4, Mult); // y == 24
```

Now that we've gone over *delegates* in isolation, let's move on to *events*.

- Part of the declaration of an event is an associated delegate type that specifies which kind of functions can be associated with the event
	- {public/protected/private} event {delegate_type} {event_name}

```
public delegate OnCarSelected();

public event OnCarSelected CarSelectedEvent;
```







### UI Topics

- Anchor it! -> We often have to deploy to multiple resolutions. Test how your components will adjust.


Exporting models to .FBX
------------------------


