## Internal commands

### require
This command loads a plugin. It is only needed in non-interactive mode.
```aquashell
require "pluginname"; #Load plugin relative to the AquaShell plugin directory

require __ALL__; #Load all available plugins from the plugin directory
```

### exec
This commands executes another script file
```aquashell
exec "scriptfile.dnys";
```
You can also provide arguments:
```aquashell
exec "scriptfile.dnys" "my arg" 123 true; # In script file "my arg" will be %1, 123 will be %2 and true will be %3. 
# You can also access the amount of arguments via %argc and a specific argument via %argv[N].
```

### sys
This command passes an expression to the Windows console subsystem.
```aquashell
sys "pause"; #Runs the pause command of the Windows console subsystem

sys "set" storagevar; #Runs the set command (which will print all environment variables) and stores this output into the result string var
```

### run
This command launches a file. How the file is launched is determined by the Windows system settings
```aquashell
run "path/to/file" "args to file" "directory to be run in";
```

### cwd
This commands changes the current working directory.
```aquashell
cwd "path/to/directory";
```

### gwd
This commands stores the current working directory in the storage string var.
```aquashell
gwd "storage_var";
```

### getscriptpath
This command stores the full path to the executed script
```aquashell
getscriptpath "result var";
```

### getscriptname
This command stores the full path including the script file name
```aquashell
getscriptname "result var";
```

### debug
Prints debug text content using OutputDebugString
```aquashell
debug "Debug text";
```

### textview
This commands prints the text contents of a file to the current output
```aquashell
textview "path/to/file";
```

### random
This commands generates a random number from start to end (inclusive)
```aquashell
random start end "result var"; 
```

### sleep
Pauses the main thread for N milliseconds
```aquashell
sleep 1000; #Will pause the main thread for 1 second (= 1000 milliseconds)
```

### bitop
Performs either a bitwise OR, AND or XOR operation and stores it to the result var
```aquashell
bitop "or/and/xor" (10, 20, ...) resultIntVar;
```

### gettickcount
Stores the elapsed time in milliseconds since the operating system was started
```aquashell
gettickcount "result int var";
```

### timestamp
Stores the current system timestamp into the result var
```aquashell
timestamp resultIntVar;
```

### fmtdatetime
Creates a formatted datetime string and stores it into the result var
```aquashell
fmtdatetime "format string" (opt:timestamp) resultStringVar;
```

### getsystemerror
Stores the current system error number into the result int var
```aquashell
getsystemerror "result var";
```

### setsystemerror
Sets the current system error number
```aquashell
setsystemerror "number";
```

### threadfunc
Executes a function in a separate thread. Optionally an array of arguments can be passed
```aquashell
threadfunc "FunctionName" (opt:list of args);
```

### hideconsole
Hides the console window for the rest of the execution. Should only be used in scripts that don't need a console window
```aquashell
hideconsole;
```

### pause
Pauses the current script execution and waits for any key to be pressed
```aquashell
pause;
```

### listlibs
Useful in interactive mode. Lists all loaded plugins
```aquashell
listlibs;
```

### quit
Quits the shell. Useful in interactive mode. If you however want to exit the current script, you should use the command <strong>exit</strong> instead.
```aquashell
quit;
```

### Plugin commands

### Array
The array plugin provides some basic functionality to work with arrays. It differs between static and dynamic arrays.
```aquashell
array "array name" "data type" "initial size as positive number" (list of initial items) #Registers an array

array_item_get "name of array" "positive index of array item" "target var"; #Stores the array item expression to the given variable of same type

array_item_get "source variable" "array name" "positive index of array item to save to"; #Stores the expression of the variable to the specified array item

array_item_copy "source array name" "positive index" "target array name" "positive index"; #Copies one array value to another array item

array_resize "array name" "new array size"; #Resizes the array with new dimension. Use 0 to reset the array

array_item_insert "array name" "position to insert" "expression"; #Inserts the expression as a new item at the given position

array_item_append "array name" "expression"; #Appends the expression to the array

array_item_remove "array name" "index"; #Removes the item at the given position

array_clone "source array" "cloned array name"; # Clones the array into a new created array with the given name

array_free "array name"; #Removes the array and frees the memory
```

### Auto
The auto (automation) plugin can be used to interact with other programs, windows or controls.
```aquashell
aut_findwindow "window class name" "window title" "result var"; #Tries to find the window with given title and/or class name. If you want to skip one entity, use \0. Stores the found window handle in the specified result variable.

aut_iswindow "hwnd" "result var"; #Stores whether the given window is still valid in the result var of type bool.

aut_getfgwindow "result var"; #Stores the foreground window handle in the object variable if any found

aut_setfgwindow "hwnd" "result var"; #Tries to set the given window to foreground and stores the result in the result var of type bool

aut_setwndpos "hwnd" "xpos" "ypos" "result var"; #Tries to set the window position to the given coords. Stores the result in the result var of type bool

aut_getwndpos "hwnd" "result var x" "result var y" "operation result"; #Tries to get the window position and stores x and y in the variables. Also the operation result is stored in a result variable of type bool

aut_getwndsize "hwnd" "result var width" "result var height" "operation result"; #Tries to get the width and height of a window and stores the data. Also the operation result is stored in the result var of type bool

aut_setwndtext "hwnd" "new text" "result var"; #Tries to set the given windows text. Operation result is stored in result var of type bool

aut_getwndtext "hwnd" "result var"; #Stores the window text of the specified window in the result var of type string

aut_getclassname "hwnd" "result var"; #Stores the window class name of the specified window in the result var of type string

aut_showwnd "hwnd" "style" "result var"; #Tries to set the show style of a window. Stores the operation result in the result var

aut_getcursorpos "result var x" "result var y" "operation result"; #Get cursor position and store the values in the variables. Also stores the operation result in the op result var

aut_setcursorpos "x" "y" "operation result var"; #Sets current cursor position and stores operation result in the given var

aut_run "file/to/run" "params" "dir" "operation result var"; #Runs the given object. How the object is treated depends on Windows settings. You can also specify arguments, a directory of the file context and a result var that receives the status of the operation as bool

aut_iskeydown "vkey" "result var"; #Indicates if the given key is held down

aut_iskeyup "vkey" "result var"; #Indicates if the given key is released

aut_enumwindows "callback function name"; #Enumerates all available windows and calls a function with following schema: function EnumWindowsCallback bool(hwnd int, title string) {}; Returns indication value wether or not to continue in the enumeration process

aut_enumchildwindows "hwnd" "callback function name"; #Enumerates child windows of a parent window. Callback routine definition is same as above

aut_sendkeystrokes "strokes" "shall simulate ctrl" "shall simulate shift" "shall simulate alt" "operation result"; #Sends keystrokes to the system. You can specify if CTRL, Shift or Alt shall also be emulated. Stores operation result. Strokes can be characters, numbers but also special keys such as the return key, specified as \VK_RETURN; For instance to send the strokes 123+45= and return, you can use the token "123+45=\VK_RETURN;"

aut_sendmousestrokes "strokes" "shall simulate ctrl" "shall simulate shift" "shall simulate alt" "operation result"; #Sends mouse strokes to the system. Also here you can specify ctrl, shift and alt and also an operation result var. Mouse strokes are only special tokens such as "\VM_LEFTDOWN;\VM_LEFTUP;"

aut_keybdevent "virtual key code" "scan code" "flags"; #Uses keybd_event() WinAPI function to simulate key events

aut_mouseevent "flags" "x" "y" "data"; #Uses mouse_event() WinAPI function to simulate mouse events

aut_addkeyevent "name of function to call" "key number to hook into" "operation result"; #Add a keyboard hook. The function to be called is of following definition: function KeyboardEvent void(isdown bool, isshift bool, isctrl bool, isalt bool) {};

aut_addmouseevent "name of function to call" "mouse number to hook into" "operation result"; #Add a mouse hook. The function to be called is of following definition: function MouseEvent void(isshift bool, isctrl bool, isalt bool) {}; Note it is only called for pressed event, not down or up

aut_sendmessage "hwnd" "msg" "wparam" "lparam" "result var"; #Sends the given message with params to the given window and returns its result.

aut_postmessage "hwnd" "msg" "wparam" "lparam" "result var"; #Posts the given message with params to the given window. Will return true if the given message was put into the posting queue, not the actual result from the target window. 

aut_clpbsetstring "text"; #Stores the given expression as text in the clipboard

aut_clpbgetstring "result var"; #Stores the current clipboard data as text in the result var

aut_clpbclear; #Clears the clipboard data regardless of the content type

aut_addtimer "ident" "delay"; #Adds a timer where the callback function is executed each N milliseconds. The callback function is of following schema: function ident_OnElapsed bool() {}; Its return value indicates if the timer shall be removed or stay processed

aut_timerexists "timer ident" "result var"; #Indicates if a timer with the given ident exists.

aut_procevents; #Call this in your main loop in order to process all your hooks

aut_calctimers; #Call this in your main loop in order to process all timers
```

### EnvVars
This plugin provides all environment variables to the scripting system. It does not provide any further functions, but just makes
the variables available. For example you will then have the variable %windir or %USERNAME available. What variables actually are available
depends on your Windows system and settings

### Events
The events plugin allows you to register events and then trigger them on any desired occassion.
```aquashell
event.register "event name" "amount of arguments" "shall allow multiple handlers"; #Registers an event type. You need to specify the event name, a positive number that specifies the amount of arguments that a handler receives and also a bool flag if multiple handlers are allowed for that event

event.add "event name" "name of callback function"; #Associate an event handler function with a given registered event

event.exists "event name" "boolean result var"; #Stores a boolean value indicating whether the given event exists or not

event.raise "event name" (list, of, arguments) "result var to store result"; #Raise an event and call the handler. You can pass arguments to the handler. The amount must match the count specified when registered the event. You can specify the name of a variable that shall recieve the function result or just void if storage of a result is not intended.

event.release "event name"; #Removes the event and associated data
```

### FileIO
The FileIO plugin allows you to access files on a storage system (e.g. hard disk, USB drive, ...). 
```aquashell
fopen "file name", "shall_append" "result handle"; #Opens the given file for read/write operations and stores the file handle in the result var. You can check if the handle is valid via the constant %FIO_INVALID_HANDLE.

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

### Forms
The forms plugin can be used to create GUI systems consisting of windows with attached controls. This is a nice way to create more
userfriendly interfaces and visually configure the scripted task.
```aquashell
wnd_spawnform "form name" "form title text" x y width height "result var"; #Creates a new form and stores the handle in the result var of type int

wnd_setformpos "form handle" x y; #Sets the new form position

wnd_setformres "form handle" w h; #Sets the new form resolution

wnd_setformtitle "form handle" "new title"; #Updates the form title with the given text

wnd_setcomppos "form handle" "component type" "component name" x y; #Sets the new position of a form component

wnd_setcompres "form handle" "component type" "component name" w h; #Sets the new resolution of a form component

wnd_setcomptext "form handle" "component type" "component name" "new text"; #Sets the new text of a form component

wnd_setcompfont "form handle" "component type" "component name" "font name" w h bold italic underline strikeout; #Sets new component font data

wnd_getcomptext "form handle" "component type" "component name" "result var"; #Gets a component text

wnd_getcomphandle "form handle" "opt:component type" "opt:component name" "result var"; #Either get the handle of the form or an attached control to a form

wnd_setcompstyle "form handle" "component type" "component name" "new style" "result var"; #Sets new component styles and stores the old styles in the result variable

wnd_spawnlabel "form handle" "label name" x y w h "initial text"; #Adds a new label to the given form.

wnd_spawnbutton "form handle" "button name" x y w h "button text"; #Adds a new button to a form. It is assumed that a callback function with following definition exists: function ButtonName_OnClick void() {};

wnd_spawntextbox "form handle" "textbox name" x y w h "initial text"; #Adds a new textbox to the form. It is assumed that a callback function with following definition exists: function TextboxName_OnChange void(newText string) {};

wnd_settextboxmultiline "form handle" "textbox name"; #Sets a textbox component to be multiline

wnd_gettextboxtext "form handle" "textbox name" "result var"; #Stores the textbox text in the result var if found

wnd_settextboxtext "form handle" "textbox name" "new text"; #Sets the text content of the given textbox if exists

wnd_appendtextboxtext "form handle" "textbox name" "text to append"; Appends the text content to the given textbox in a new line

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

wnd_spawnimagebox "form handle" "imagebox name" x y w h "opt:initial image file"; #Adds a new imagebox to the form

wnd_setimageboximage "form handle" "imagebox name" "image file"; #Sets the current image of the imagebox

wnd_getimageboximage "form handle" "imagebox name" "result var"; #Stores the current image file name to the var

wnd_isformvalid "form handle" "result var"; #Indicates if the specified form is valid

wnd_process; #Processes all window events. Should be called in a main loop

wnd_getcurrentfocus "result var"; Get window handle of window or control that currently has focus

wnd_freeform "form handle"; #Frees the form and all attached controls
```

### Input
The InputBox plugin provides both a command and a function to graphically receive input expressions from the user.
```aquashell
input "target var" "A descriptive text"; #Prompts the user to input text that is stored into the var

inputbox "title of inputbox" "label/description text" "default expression" xpos ypos "name of result var"; #Prompts the user via GUI and returns the input to the script
```

### NetClient
The NetClient plugin can be used to perform basic network tasks. That means it can communicate with a network service on a specified port, be
it Internet or LAN.
```aquashell
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

### Speech
The speech plugin can be used to use Microsoft SAPI to perform text2speech.
```aquashell
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

### Strings
The strings plugin can be used to perform all importan operations on strings
```aquashell
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

### Ini
This plugin provides functionality to access INI configuration files
```aquashell
ini_read "path/to/file.ini" "section" "key" "result string variable"; #Stores the value of section.key into the result var

ini_write "path/to/file.ini" "section" "key" "expression"; #Writes the given expression into section.key. If it doesn't exist, then it will be created

ini_delete "path/to/file.ini" "section" "key"; #Removes the key from the section
```

### IRC
This plugin provides commands in order to connect and communicate with an IRC server
```aquashell
irc_spawn "identifier" "host" "port" "result var"; # Attempts to connect to an IRC server
irc_isvalid "identifier" "boolean result var"; # Indicates whether the identifier is linked to a valid (and connected) IRC object instance
irc_process "opt:identifier"; # Processes the IRC object. If no identifier is specified then it processes all existing IRC objects. 
irc_send "identifier" "message"; # Attempts to send a message to the server associated with the given object instance
irc_release "identifier"; # Releases the IRC object instance which results in closing the connection.
```

<p><hr/></p>

[Go back](index.md)