# Documentation

Welcome to the official AquaShell Scripting documentation. AquaShell is a scripting shell based on the dnyScript scripting language. The shell allows you to create automation tasks via small scripts as well as develop complex applications.

In order to make a build get the source code and project files here: https://github.com/danielbrendel/dnyAquaShell

Syntax highlighting is available for both <a href="https://marketplace.visualstudio.com/items?itemName=danielbrendel.dnyscript-syntax">Visual Studio Code</a> and <a href="res/dnyScript_npp_highlighting.xml">Notepad++</a>.

## Table of contents:
- [Shell documentation](#shell)
- [Scripting documentation](#scripting)
- [Command Reference](#command-reference)

## Shell

This product is an extendable scripting shell for Windows using dnyScriptInterpreter.
The shell can be used either in interactive mode or just as a host for running
dnyScript script files. The functionality of the shell can be extended via native
DLL plugins. There are also useful command line options available.

### Example code
```
# Example script code
# Demonstrate recursive function calls

const MAX_COUNT int <= 10;

function recursive void(count int)
{
  if (%count, -ls, %MAX_COUNT) {
    ++ count;
    print "Count value: %count";
    call recursive(%count) => void;
  };
};

call recursive(0) => void;

print "Done.";

pause;
```

### Basic commands:
* require (library) # Attempts to load a plugin library. Only needed when not running in interactive mode
* exec (scriptfile) (opt:args) # Executes a script file. In the script use %argc for the argument count and %argv[n] for the actual argument value. Using %n, where n is the argument index value, is also possible.
* sys (string) (opt:storage var) # Attempts to run the expression as a windows command. If a result var is provided the output is stored in it, else the output gets echoed.
* run (file) (args) (dir) # Attempts to launch a specified file
* cwd (dir) # Changes the current working directory
* pause # Pause current script execution
* listlibs # Lists all shell plugin libraries
* quit # Exists the shell

### Helper constants and variables
* DNYAS_IS_INTERACTIVE_MODE: Boolean value that indicates whether or not the current script code runs within the interactive mode
* CR: Represents a carriage return character
* LF: Represents a line feed character
* void: Can be used with commands or functions if you want to dismiss the result value.

### Helper commands:
* getscriptpath (result) # Stores the full path of the current executed script in the result var
* getscriptname (result) # Stores the full file name of the current executed script in the result var

### Multiline support:
* \< # Opens the editor in multiline mode
* (code)
* \> # Closes the multiline mode and executes the script code

### Init and unload script:
* You can place a script named 'init.dnys' in the 'scripts' directory of the base directory
  which will get executed when the shell is in loading progress. There you can place
  initialization code
* You can place a script named 'unload.dnys' in the 'scripts' directory of the base directory
  which will get executed when the shell gets unloaded. There you can place cleanup code

### Command line arguments

The following command line arguments exist:

* -v: Prints out the version information
* -e "path/to/a/script.dnys" [opt:args]: If an existing script file is provided, then the shell will try to execute it.
* -c "script code": Execute script code that is provided as an argument.

You can also add the shell path to your PATH environment variable using the following command line system command:
* "add_path" "-u|-m": Adds the shell path to your PATH environment variable. Use -u for current user or -m for local machine.

If you have added the shell path to your environment PATH variable, then you can also perform actions as follows:
```
aquashell [args]
```

### Plugin API:
Plugins must be written in compatibility with the shell application. 

A plugin needs to export the functions `dnyAS_PluginLoad` and `dnyAS_PluginUnload`.

```cpp
bool dnyAS_PluginLoad(dnyVersionInfo version, IShellPluginAPI* pInterfaceData, plugininfo_s* pPluginInfos);
void dnyAS_PluginUnload(void);
```
The first one is called when the plugin gets loaded. There you must implement all loading stuff. The function recieves the current shell interface version, a pointer to the plugin API class instance and a pointer to a plugin information structure where the plugin should save its information strings. If everything goes well then the plugin must return true, otherwise false.

Here is an example of a plugin info struct object.

```cpp
plugininfo_s g_sPluginInfos = {
	L"Plugin name",
	L"1.0",
	L"Author name",
	L"Contact info",
	L"Plugin description"
};
```

The latter one is called when the plugin gets unloaded. There you can implement all cleanup stuff. 

Please refer to the demo plugin sourcecode in order to view a full documented example.

## Scripting

### Feature overview:
* Scripting Engine for application-embedded scripting
* User-friendly scripting syntax
* Interface between script engine and engine user
* Default internal commands
* Sticks to C++ STD in order to provide compatibility
 
### Syntax:
The script language is a command-based language. This makes the syntax be as follows:
```
(cmd_ident) (expression2) (expression3) (expression n...);
```
Now 'cmd_ident' is the identifier of a specific command. 'expression2' - 'expression n'
are considered arguments to a specific command. Each command must be ended with a semicolon (";"). 
In order to allow complex scripts it is neccessary to summarize/collect expressions with collector characters. 
These are quotation marks ("") and brackets ("()" and "{}"). This allows you to format your command call
as follows:
```
myCommandIdent "argument1" (argument2a, arguments3a, ...) { complex argument ("test1", "test2");};
```
Each command or empty line can be noted with a '#(comment)' to comment your scripts.
When writing your scripts you should pay attention to a good coding-style which offers the reader
to have a better glance at your scripts. Some commands require an argument expression and a code expression.
Altough not neccessarly required you should put argument expressions in () brackets and code expressions
in {} brackets (note: it depends on the command provider whether argument or code expressions are required. 
Most of the times you will just encounter parameterized commands). See the example below:
```
#The following function implementation works, but is bad style
function myFunction void "arg1 string, arg2 string" "print %arg1%arg2;";
```
You can see that you couldn't include spacing between the argument accessor code. A better example would be:
```
function myFunction void(arg1 string, arg2 string)
{
	print "%arg1 %arg2"; #This is a comment
};
```
As you can see some commands may require arguments provided as an array list, separated by a commas,
in a single expression.

### Examples:

1) Using comments
Comments are indicated by a #-character. Everything what follows
a comment is ignored by the parser. 

2) Declaring global and local variables

Declare a global variable
```
global name type;
```
There are the following system types available:
- bool -> A boolean value (true and false)
- int -> A signed integer which is 64 bits in size
- float -> A signed float which is 64 bits in size
- string -> A string
Additionally custom variable data types can be used.
A declaration then is applied as follows:
```
global name customtypename;
```
Local variables are declared as follows:
```
local name datatype;
```
Local function variables are automatically removed after the function has finished.
If you have a local and a global variable with the same name then the local variable
is prefered. As a result of that clean naming is advised.

You can also initially declare a variable when setting a value. The context is then automatically determined:
```
function testfunc void()
{
	set context_local string <= "I am a local variable";
};

set context_global string <= "I am a global variable";
```

3) Assigning values to variables
```
global name1 int;
set name1 <= 100;
local name2 string;
set name2 <= "This is a test";
```

4) Creating functions
```
function myFunction type(arguments)
{
	# function implementation here
	result 0;
};
```
Functions can either return a value or not. Also providing arguments is optional.
The function arguments can handle the same types as shown in the variables section.
If a function shall not return a value then the returntype shall be set as 'void'.
```
function myVoidFunction void()
{
};

function myFloatFunction float()
{
	result 0.1;
};

function myComplexFunction int(arg1 string, arg2 float, arg3 bool)
{
	result 0;
};
```
The functions are called as follows:
```
call myVoidFunction();
call myFloatFunction() => myFloatVar;
call myComplexFunction("test1", 0.1, true) => myIntVar;
```
You can also use local variables in functions:
```
function myMoreComplexFunction float(arg1 float)
{
	local temp float;
	set temp <= %arg1;
	*= temp %arg1;
	result %temp;
};
```

5) Defining classes
Sometimes tasks should be bound to objects. dnyScript provides a basic class structure functionality.
```
class ClassName {
	method myTestMethod void()
	{
	};
	
	member myMember float;
};
```
You can define methods and variables similar to the global scope. Instead of using the keyword 'function' you use 
'method'. And for member variables you use the keyword 'member'. Member variables are automatically allocated upon
class instantiation and freed after the class instance gets removed from memory.
You can also define a constructor and a destructor.
```
class MyTestClass {
	method construct void()
	{
		print "Constructor called";
	};
	
	method destruct void()
	{
		print "Destructor called";
	};
};
```
To access methods or members inside the class context you can use %this.
When accessing the class instance from outside you must prefix the class 
instance identifier with an @ character except when accessing member variables.
```
global classInstance class;
set @classInstance <= MyTestClass;
call @classInstance.SomeMethod() => void;
set someVariable <= "%classInstance.MyVariable";
unset @classInstance;
```

6) Commands
The script user can implement own script commands which can retrieve arguments
and also return a value. 
#A void command:
```
myCommand "stringarg"
```
#A value-returning command:
```
global myStorageVar type;
myRetValCommand "stringarg" myStorageVar;
```
For void-commands you need to implement the IVoidCommandInterface class and for
value-returning commands you need to implement the IResultCommandInterface<type> class.
Note that both type of commands can return a boolean value to indicate errors. These
errors are handled solely by the script engine and you should only indicate errors
if the script execution is considered uncontinueable. If you want to return values which 
do not affect further script execution and which shall be handled by the script code then 
you should stick to the result-command interface. It supports all dny-data types.

### Internal command reference:
```
const constname consttype <= value; //Registers a constant with the given name, type and value\
global varname vartype; //Registers a global variable with the associated type\
set varname <= value; //Assigns a value to a global/local variable\
unset varname; //Removes a global variable\
function name rettype(paramters) {code} //Defines a function\
local varname vartype; //Registers a local function variable inside a function with the associated type\
result value; //Sets the result value of the associated function\
call funcname(arguments) => resultvar; //Calls a function with paramters (if required) and stores the result (if required)\
if (cond1, operation, cond2) {code}; <elseif (cond1, operation, cond2) {code}> <else {code}> //Performs an if-elseif-else evaluation. Elseif's and else's are optional. 'operation' can either be -eq(equal), -nt(not equal), -ls(less than), -lse(less than or equal), -gr(greater than) and -gre(greater than or equal).\
for (varname, startvalue, endvalue, step, opt:param) {code}; //Performs a for-loop with positive or negative values (use -inc for 1 and -dec for -1 or a step value. Param can be -eq if the loop var shall be inclusive to the loop end value, default behaviour is non-inclusive)
while (cond1, operation, cond2) {code} //Performs a while-loop. 'operation' can either be -eq(equal), -nt(not equal), -ls(less than), -lse(less than or equal), -gr(greater than) and -gre(greater than or equal).\
class name { code }; //Define a class structure
method name rettype(parameters) { code }; //Implement a class method 
member name vartype; //Define a class member variable
execute scriptfile; //Executes a script file relative to the base directory\
print text; //Prints text to the standard output if set by script user\
exit; //Aborts script execution\
++ var; //Increments a variable\
-- var; //Decrements a variable\
+= var value; //Adds the value to the variable\
-= var value; //Subtracts the value from the variable\
*= var value; //Multiplicates the value to the variable\
/= var value; //Divides the value from the variable\
+ var value1 value2; //Stores the sum to the variable\
- var value1 value2; //Stores the difference to the variable\
* var value1 value2; //Stores the product to the variable\
/ var value1 value2; //Stores the quotient to the variable\
! boolvar; //Reverses the boolean value of the variable
```
 
### Usage:
* Instantiate an object of CScriptingInterface object
* Set a script base directory with SetScriptBaseDirectory() method if required
* Set a standard output routine with SetStandardOutput() method if required
* Register custom data types with RegisterDataType() if required
* Register commands with RegisterCommand() method or variables with RegisterCVar() method 
  (both kind of objects can also be unregistered if not needed anymore). Implement classes
  derived from IVoidCommandInterface interface for void-commands and IResultCommandInterface<(
  dnyDataType)> interface for commands that return a value.
* Execute scripts via ExecuteScript() method or code via ExecuteCode() method
* Use the methods BeginFunctionCall(), ExecuteFunction() and EndFunctionCall() to call a 
  script function. If required you can push arguments via the PushFunctionParam() methods
  and query function result data via the QueryFunctionResultAs*() methods.
* Possible operation status values can be obtained via GetErrorInformation() method. It returns
  a class object instance which can be used to query the operation status code and an additional
  error message if the status was set to an error indicating value

### System requirements:
* ISO-conform C++ compiler

## Command reference

### Internal commands

#### require
This command loads a plugin. It is only needed in non-interactive mode.
```
require "pluginname"; #Load plugin relative to the AquaShell plugin directory
```

#### exec
This commands executes another script file
```
exec "scriptfile.dnys";
```
You can also provide arguments:
```
exec "scriptfile.dnys" "my arg" 123 true; # In script file "my arg" will be %1, 123 will be %2 and true will be %3
```

#### sys
This command passes an expression to the Windows console subsystem.
```
sys "pause"; #Runs the pause command of the Windows console subsystem
```

#### run
This command launches a file. How the file is launched is determined by the Windows system settings
```
run "path/to/file" "args to file" "directory to be run in";
```

#### cwd
This commands changes the current working directory. Useful when you are in interactive mode and want to change the directory.
```
cwd "path/to/directory";
```

#### pause
Pauses the current script execution and waits for any key to be pressed
```
pause;
```

#### listlibs
Useful in interactive mode. Lists all loaded plugins
```
listlibs;
```

#### quit
Useful in interactive mode. Quits the shell
```
quit;
```

### Plugin commands

#### Array
The array plugin provides some basic functionality to work with arrays. It differs between static and dynamic arrays.
```
static_array "name" "data type" (list, of, items); #Creates a static array with the given items. Datatype may be one of the default dnys data types

store_array_item_s (array name, index var) "target var"; #Stores an array item in any registered var with the same data type

free_array_s "array name"; #Removes the array and frees the memory

dynamic_array "array name" "data type" "initial size as positive number" (list of initial items) #Registers a dynamic array

store_array_item_d "name of array" "positive index of array item" "target var"; #Stores the array item expression to the given variable of same type

store_item_to_array_d "source variable" "array name" "positive index of array item to save to"; #Stores the expression of the variable to the specified array item

copy_array_item_d "source array name" "positive index" "target array name" "positive index"; #Copies one array value to another array item

resize_array "array name" "new array size"; #Resizes the array with new dimension

item_insert "array name" "position to insert" "expression"; #Inserts the expression as a new item at the given position

item_append "array name" "expression"; #Appends the expression to the array

free_array_d "array name"; #Removes the array and frees the memory

asetindex "index value"; #Sets the common array index value

foreach (array name, iterator variable name) { code }; #Iterates through the array and executes the given code for each iteration
```

#### Auto
The auto (automation) plugin can be used to interact with other programs, windows or controls.
```
HWND obj_name; #Registers an object from type HWND for window management

aut_findwindow "hwnd object" "window title" "window class name"; #Tries to find the window with given title and/or class name. If you want to skip one entity, use \0. Stores the found window handle in the specified object variable.

aut_iswindow "hwnd object" "result var"; #Stores whether the given window is still valid in the result var of type bool.

aut_getfgwindow "hwnd object"; #Stores the foreground window handle in the object variable if any found

aut_setfgwindow "hwnd object" "result var"; #Tries to set the given window to foreground and stores the result in the result var of type bool

aut_setwndpos "hwnd object" "xpos" "ypos" "result var"; #Tries to set the window position to the given coords. Stores the result in the result var of type bool

aut_getwndpos "hwnd object" "result var x" "result var y" "operation result"; #Tries to get the window position and stores x and y in the variables. Also the operation result is stored in a result variable of type bool

aut_getwndsize "hwnd object" "result var width" "result var height" "operation result"; #Tries to get the width and height of a window and stores the data. Also the operation result is stored in the result var of type bool

aut_setwndtext "hwnd object" "new text" "result var"; #Tries to set the given windows text. Operation result is stored in result var of type bool

aut_getwndtext "hwnd object" "result var"; #Stores the window text of the specified window in the result var of type string

aut_getclassname "hwnd object" "result var"; #Stores the window class name of the specified window in the result var of type string

aut_showwnd "hwnd object" "style" "result var"; #Tries to set the show style of a window. Stores the operation result in the result var

aut_getcursorpos "result var x" "result var y" "operation result"; #Get cursor position and store the values in the variables. Also stores the operation result in the op result var

aut_setcursorpos "x" "y" "operation result var"; #Sets current cursor position and stores operation result in the given var

aut_run "file/to/run" "params" "dir" "operation result var"; #Runs the given object. How the object is treated depends on Windows settings. You can also specify arguments, a directory of the file context and a result var that receives the status of the operation as bool

aut_iskeydown "vkey" "result var"; #Indicates if the given key is held down

aut_iskeyup "vkey" "result var"; #Indicates if the given key is released

aut_enumwindows "callback function name"; #Enumerates all available windows and calls a function with following schema: function EnumWindowsCallback bool(hwnd int, title string) {}; Returns indication value wether or not to continue in the enumeration process

aut_enumchildwindows "hwnd" "callback function name"; #Enumerates child windows of a parent window. Callback routine definition is same as above

aut_sendkeystrokes "strokes" "shall simulate ctrl" "shall simulate shift" "shall simulate alt" "operation result"; #Sends keystrokes to the system. You can specify if CTRL, Shift or Alt shall also be emulated. Stores operation result. Strokes can be characters, numbers but also special keys such as the return key, specified as \VK_RETURN; For instance to send the strokes 123+45= and return, you can use the token "123+45=\VK_RETURN;"

aut_sendmousestrokes "strokes" "shall simulate ctrl" "shall simulate shift" "shall simulate alt" "operation result"; #Sends mouse strokes to the system. Also here you can specify ctrl, shift and alt and also an operation result var. Mouse strokes are only special tokens such as "\VM_LEFTDOWN;\VM_LEFTUP;"

aut_addkeyevent "name of function to call" "key number to hook into" "operation result"; #Add a keyboard hook. The function to be called is of following definition: function KeyboardEvent void(isdown bool, isshift bool, isctrl bool, isalt bool) {};

aut_addmouseevent "name of function to call" "mouse number to hook into" "operation result"; #Add a mouse hook. The function to be called is of following definition: function MouseEvent void(isshift bool, isctrl bool, isalt bool) {}; Note it is only called for pressed event, not down or up

aut_procevents; #Call this in your main loop in order to process all your hooks
```

#### EnvVars
This plugin provides all environment variables to the scripting system. It does not provide any further functions, but just makes
the variables available. For example you will then have the variable %windir or %USERNAME available. What variables actually are available
depends on your Windows system and settings

#### Events
The events plugin allows you to register events and then trigger them on any desired occassion.
```
events.register "event name" "amount of arguments" "shall allow multiple handlers"; #Registers an event type. You need to specify the event name, a positive number that specifies the amount of arguments that a handler receives and also a bool flag if multiple handlers are allowed for that event

events.add "event name" "name of callback function"; #Associate an event handler function with a given registered event

events.raise "event name" (list, of, arguments) "result var to store result"; #Raise an event and call the handler. You can pass arguments to the handler. The amount must match the count specified when registered the event. You can specify the name of a variable that shall recieve the function result or just void if storage of a result is not intended.
```

#### FileIO
The FileIO plugin allows you to access files on a storage system (e.g. hard disk, USB drive, ...). 
```
fopen("var name to store handle", "file name", shall_append) { code to execute }; #Opens a file and optionally executes the code that handles the file operation

fisopen "file handle" "result var"; #Indicates if a file has been opened

fateof "file handle" "result var"; #Indicates if file pointer is at end of file

fwritetext "file handle" "text to write"; #Writes text to file at current position

fwriteline "file handle" "line to write"; #Writes a line to the file at current position

freadline "file handle" "result var"; #Reads the current line of the file and stores it in the result var

fclose "file handle"; #Closes the file

dcreate "folder name" "result var"; #Attempts to create a folder stores the result var

dremove "folder name" "result var"; #Attempts to delete the specified folder and stores the result var

fremove "file name" "result var"; #Attempts to delete the given file and stores the result var

denum "base folder" "filter" "callback function name" "result"; #Enumerates all subfolders in a given folder. The callback function must be of following definition: function CallbackEnumFolders int(baseFolder string, subfolder string) {};

dgetcurrent "result var"; #Stores the current working directory in the result var

fgetsize "file name" "result var"; #Stores the file size in bytes in the result var if it has been found

fdisdir "object name" "result var"; #Indicates if the given object name is a folder.

fexists "file name" "result var"; #Indicates if the given file exists

dexists "folder name" "result var"; #Indicates if the given folder exists

fcopy "source" "destination" "result var"; #Tries to copy the file from source to destination and stores the result

fmove "source" "destination" "result var"; #Tries to move the file from source to destination and stores the result
```

#### Forms
The forms plugin can be used to create GUI systems consisting of windows with attached controls. This is a nice way to create more
userfriendly interfaces and visually configure the scripted task.
```
wnd_spawnform "form name" "form title text" x y width height "result var"; #Creates a new form and stores the handle in the result var of type int

wnd_setformpos "form handle" x y; #Sets the new form position

wnd_setformres "form handle" w h; #Sets the new form resolution

wnd_setcomppos "form handle" "component type" "component name" x y; #Sets the new position of a form component

wnd_setcompres "form handle" "component type" "component name" w h; #Sets the new resolution of a form component

wnd_setcomptext "form handle" "component type" "component name" "new text"; #Sets the new text of a form component

wnd_setcompfont "form handle" "component type" "component name" "font name" w h bold italic underline strikeout; #Sets new component font data

wnd_getcomptext "form handle" "component type" "component name" "result var"; #Gets a component text

wnd_spawnlabel "form handle" "label name" x y w h "initial text"; #Adds a new label to the given form.

wnd_spawnbutton "form handle" "button name" x y w h "button text"; #Adds a new button to a form. It is assumed that a callback function with following definition exists: function ButtonName_OnClick void() {};

wnd_spawntextbox "form handle" "textbox name" x y w h "initial text"; #Adds a new textbox to the form. It is assumed that a callback function with following definition exists: function TextboxName_OnChange void(newText string) {};

wnd_gettextboxtext "form handle" "textbox name" "result var"; #Stores the textbox text in the result var if found

wnd_settextboxtext "form handle" "textbox name" "new text"; #Sets the text content of the given textbox if exists

wnd_spawncheckbox "form handle" "checkbox name" x y w h "initial text" "is checked"; #Adds a checkbox to the form

wnd_cbischecked "form handle" "checkbox name" "result var"; #Indicates if the checkbox is currently checked

wnd_cbsetvalue "form handle" "checkbox name" "check value"; #Sets the checked status of the given checkbox

wnd_spawnlistbox "form handle" "listbox name" x y w h; #Adds a new listbox to the form. The following callback functions are assumed: function ListboxName_OnSelectionChange void(newItem int) {}; and function ListboxName_OnDoubleClick void(item int) {};

wnd_lbadditem "form handle" "listbox name" "new text item"; #Adds a new text item to the listbox

wnd_lbinsertitem "form handle" "listbox name" "index of position where to insert" "new text item"; #Inserts an item at the given position

wnd_lbremoveitem "form handle" "listbox name" "index of item which to remove"; #Removes the item from the listbox

wnd_lbupdateitem "form handle" "listbox name" "index of item to update" "new text content"; #Updates the given item with new content

wnd_lbselectitem "form handle" "listbox name" "index of item to select"; #Selects the given item of the listbox

wnd_lbgetcount "form handle" "listbox name" "result var"; #Stores the amount of items in the result var

wnd_lbgetselection "form handle" "listbox name" "result var"; #Stores the index value of the selected item in the result var

wnd_lbgettext "form handle" "listbox name" "index of item" "result var"; #Stores the text content of a given item in the result var

wnd_spawncombobox "form handle" "combobox name" x y w h; #Adds a combobox to the form. It is assumed that the following callback function exists: function ComboboxName_OnSelect void(newItem string) {};

wnd_cbadditem "form handle" "combobox name" "new text item"; #Adds a new text item to the combobox

wnd_cbremoveitem "form handle" "combobox name" "index of item"; #Removes the given item by index

wnd_cbgetcount "form handle" "combobox name" "result var"; #Stores the amount of items in the result var of type int

wnd_cbselectitem "form handle" "combobox name" "index of item"; #Selects the given item by index

wnd_cbgetselection "form handle" "combobox name" "result var"; #Stores the content of the selected item in the result var

wnd_cbgetselectionid "form handle" "combobox name" "result var"; #Stores the index value of the selected item into the result var of type int

wnd_cbgetitemtext "form handle" "combobox name" "index value" "result var"; #Stores the text content of an item by index value in the result var of type string

wnd_spawnlistview "form handle" "listview name" x y w h; #Adds a listview control to the form. The following callback functions are assumed: function ListviewName_OnSelect void(item int) {}; and function ListviewName_OnDoubleClick void(item int) {};

wnd_lvaddcategory "form handle" "listview name" "category name" "width" "is left aligned"; #Adds a new category to the listview

wnd_lvgetsubitemcount "form handle" "listview name" "result var"; #Stores the sub item count in the result var

wnd_lvgetitemcount "form handle" "listview name" "result var"; #Stores the item count in the result var

wnd_lvsetitemtext "form handle" "listview name" "item text" "item num" "result var"; #Adds a listview item

wnd_lvsetsubitemtext "form handle" "listview name" "item num" "subitem text" "subitem num" "result var"; #Adds a subitem to the listview

wnd_lvgetitemtext "form handle" "listview name" "item num" "subitem num" "buffer count" "result var"; #Gets an item/subitem text content and stores it

wnd_lvgetselection "form handle" "listview name" "result var"; #Gets the current selection

wnd_lvdeleteitem "form handle" "listview name" "index of item"; #Deletes the given item

wnd_spawnprogressbar "form handle" "bar name" x y w h max start; #Adds a new progress bar to the form

wnd_setpbrange "form handle" "bar name" "max value"; #Sets the new max value of the progress bar

wnd_setpbposition "form handle" "bar name" "position"; #Sets the new position value of the progress bar

wnd_getpbrange "form handle" "bar name" "result var"; #Stores the max value in the result var

wnd_getpbposition "form handle" "bar name" "result var"; #Stores the current position value in the result var

wnd_spawnimagebox "form handle" "imagebox name" x y w h "image file"; #Adds a new imagebox to the form

wnd_setimageboximage "form handle" "imagebox name" "image file"; #Sets the current image of the imagebox

wnd_getimageboximage "form handle" "imagebox name" "result var"; #Stores the current image file name to the var

wnd_isformvalid "form handle" "result var"; #Indicates if the specified form is valid

wnd_process; #Processes all window events. Should be called in a main loop

wnd_freeform "form handle"; #Frees the form and all attached controls
```

#### InputBox
The InputBox plugin provides both a command and a function to graphically receive input expressions from the user.
```
inputbox "title of inputbox" "label/description text" "default expression" xpos ypos "name of result var"; #Prompts the user and returns the input to the script

#Or call the wrapper function: 
call inputbox("title", "label", "default", x, y) => result_var;
```

#### MiscUtils
This plugin provides various miscellaneous helper commands.
```
addtimer "timer ident" "integer duration in milliseconds"; #Adds a timer where the callback function is executed each N milliseconds. The callback function is of following schema: function YourIdent_OnElapsed bool() {}; Its return value indicates if the timer shall be removed or stay processed

timerexists "timer ident" "result var"; #Checks if the given timer exists and stores the result

calctimers; #Processes all timers. This should be run from your main loop

textview "file/to/view"; #This is just a command to print the whole content of a text file to the output

random "start" "max" "result var"; #Generates a random number between 'start' and 'max' (inclusive). Stores the result in the result variable

sleep "time in milliseconds"; #Let's the script execution pause for the given amount of milliseconds

clpb_setstring "text content"; #Writes the given string expression to clipboard

clpb_getstring "result var"; #Writes the contents of the clipboard (if it is text) to the result variable

clpb_clear; #Clears the clipboard content

gettickcount "result var"; #Gets the system tick count and writes it to the given result variable
```

#### NetClient
The NetClient plugin can be used to perform basic network tasks. That means it can communicate with a network service on a specified port, be
it Internet or LAN.
```
net_spawnclient "client ident" "target service address" "target service port" ("socket type tcp/udp", "data type ansi/unicode"); #Spawns a network client to communicate with a network service. For each spawned client the following event functions must exist:
function ClientIdent_OnConnected void() {}; #Connected to service
function ClientIdent_OnDisconnected void() {}; #Disconnected from service
function ClientIdent_OnRecieve void(content string) {}; #Recieve text contents from service
function ClientIdent_OnError void(error_message string) {}; #Called for occurred errors 

net_isvalid "client ident" "result var"; #Checks if a client with that ident exists and stores the result in the var

net_process; #Processes all clients and their sockets. This should be called in a main loop

net_sendbuffer "client ident" "text content"; #Tries to send the text content through the client channel

net_releaseclient "client ident"; #Releases the client, frees the memory and removes it
```

#### Speech
The speech plugin can be used to use Microsoft SAPI to perform text2speech.
```
spk_setvoice "voice ident"; #Sets the current available voice to use for speech

spk_setpitch "pitch integer"; #Sets the current voice pitch

spk_setvolume "volume integer"; #Sets the current volume

spk_setspeed "speed integer"; #The speed of how fast the text shall be spoken

spk_getvoice "result var"; #Gets current voice ident

spk_getpitch "result var"; #Gets current pitch value

spk_getvolume "result var"; #Gets current volume

spk_getspeed "result var"; #Gets current speed

spk_speak "text"; #Speaks the given text content. The script continues after the text has been spoken

spk_speakasync "text"; #Speaks the given text content asynchronously. The script continues immediatelly and the text is spoken parallely
```

#### Strings
The strings plugin can be used to perform all importan operations on strings
```
s_getlen "string" "result var"; #Stores the length of the string in the result var

s_getchar "string" "index" "result var"; #Stores the character at the given position of the string in the result var

s_append "string var" "expression"; #Appends the expression to the given string variable and stores it into said variable

s_find "string" "substring" "result var"; #Attempts to find the position of the substring in the string and stores it into the result var

s_substr "string" "start" "end" "result var"; #Gets the substring from 'start' to 'end' (inclusive) of the string and writes it to the result var

s_replace "result var" "old string" "new string"; #Tries to find all occurences of 'old string' in the variable and replaces it with 'new string'. Result is stored in the same variable

s_tokenize "string to tokenize" "split char" "var ident"; #Tokenizes the given string into separate variables For example the tokenization of "This is just fine" using " " and ident myIdent would create the following objects:
myIdent.count=4
myIdent[0]="This"
myIdent[1]="is"
myIdent[2]="just"
myIdent[3]="fine"

s_settokenindex "index number"; #Set current token index value

s_cleartokens "ident"; #Clears all associated token variables of that given token

s_upper "string variable"; #Converts all lower characters to upper characters of that variable

s_lower "string variable"; #Converts all upper characters to lower characters of that variable

s_rtrim "string variable"; #Performs a right-trim operation on that variable

s_ltrim "string variable"; #Performs a left-trim operation on that variable

s_fmtescseq "string token" "result var"; #Writes the actual Unicode character in hexadecimal representation of the token to the result var. Example: "\53BD;" would convert the hexadecimal representation as character and store it
```

#### DateTime
This plugin provides some date and time handling commands
```
timestamp "result int var"; # Stores the current system timestamp into the result var

fmtdatetime "format string" "opt:timestamp" "result string var"; Creates a formatted datetime string and stores it into the result var. Optionally you can provide a timestamp to perform the operation on.
```

#### IRC
This plugin provides commands in order to connect and communicate with an IRC server
```
irc_spawn "identifier" "host" "port" "result var"; # Attempts to connect to an IRC server
irc_isvalid "identifier" "boolean result var"; # Indicates whether the identifier is linked to a valid (and connected) IRC object instance
irc_process "opt:identifier"; # Processes the IRC object. If no identifier is specified then it processes all existing IRC objects. 
irc_send "identifier" "message"; # Attempts to send a message to the server associated with the given object instance
irc_release "identifier"; # Releases the IRC object instance which results in closing the connection.
```

#### TextInput
This plugin can be used to read text input from the command line.
```
input "target var" "A descriptive text"; #Prompts the user to input text that is stored into the var

setinput "var name" "A descriptive text"; #Same as 'input' with the exception that the variable will be registered if it not already exists
```