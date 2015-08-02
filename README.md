# Process Palette
With Process Palette you can add custom entries to the command palette to run any command that you typically would from a terminal. This prevents you from having to switch to a terminal each time you need to run something and is especially useful for commands that you use regularly.

## Quick Start
To quickly get a feel for how it works:

1. Install process-palette.
2. Download the example [process-palette.json][2f6a8e37] configuration file and place it in the root of your project folder.
3. Load the new configuration file by choosing `Packages|Process Palette|Reload Configuration` from the menu or `Process Palette: Reload Configuration` from the command palette.

  [2f6a8e37]: https://github.com/hgmarais/process-palette/blob/master/examples/process-palette.json "process-palette.json"

The configuration file defines a single command that will list the files in the project folder by running `ls -lh`. It can be run by choosing `Process Palette: List` from the command palette. This will open the Process Palette panel and show the output. The panel can also be opened directly by pressing `Ctrl-Alt-P` or running `Process Palette: Toggle` from the command palette.

### Next Steps
1. Poke around in the configuration file a bit. Just remember to run the `Process Palette: Reload Configuration` command after making changes.
2. Read the rest of this document. Especially the **Properties** and **Variables** sections for extra flexibility.

## Configuration
Commands are specified with a configuration file in JSON format. The name of the file must be `process-palette.json` and should be in the root of your project folder. If you have multiple project folders, each with its own configuration file, then their configurations will be merged.

A `process-palette.json` file can also be placed in your `~/.atom` folder. If that is the case then it will be loaded first and any project specific files will be loaded afterwards.

### Basic Example
A `process-palette.json` configuration file contains an array called `commands`. The following is an example of an empty array:
```json
{
  "commands" : [
  ]
}
```
Each entry in the array is an object that describes one command. The most basic configuration simply specifies the command to run and associates it with an action. The following command will run [Ant](https://ant.apache.org/) without any arguments:
```json
{
  "commands" : [
    {
      "command" : "ant",
      "action"  : "Ant default"
    }
  ]
}
```

**Tip!** :
All `process-palette.json` configuration files can be reloaded by running the `Process Palette: Reload Configuration` command. It can be found in the `Command Palette` or in the `Packages|Process Palette` menu.


The new command will cause an entry to be added to the command palette called `Process Palette: Ant default`.

![Screenshot](https://github.com/hgmarais/process-palette/blob/master/resources/command-palette-basic.png?raw=true)

The working directory used when running a command is by default the project path, but it can also be configured. More on this in the Advanced Configuration section.

Command line arguments can also be specified in the form of an array of strings. The following example adds another command that causes the `clean` target to be executed by means of an argument:
```json
{
  "commands" : [
    {
      "action"  : "Ant default",
      "command" : "ant"
    },
    {
      "action"  : "Ant clean artifacts",
      "command" : "ant",
      "arguments" : ["clean"]
    }
  ]
}
```
Reloading the configuration will cause the command palette to now have two new entries:
- Process Palette: Ant default
- Process Palette: Ant clean artifacts

The namespace used for all commands is by default `Process Palette`. This is also configurable. One must just be careful to not override commands in existing packages.

Let's modify the previous two commands to use a namespace call `Ant`:
```json
{
  "commands" : [
    {
      "namespace" : "Ant",
      "action"    : "Default",
      "command"   : "ant"
    },
    {
      "namespace" : "Ant",
      "action"    : "Clean artifacts",
      "command"   : "ant",
      "arguments" : ["clean"]
    }
  ]
}
```
After reloading the configuration file the entries will be:
- Ant: Default
- Ant: Clean artifacts

### Shortcut Keys
Custom shortcut keys can also be associated with commands by adding a `keystroke` entry. Let's add the keystroke `Ctrl-Alt-A` to the `Ant: Default` command:
```json
{
  "namespace" : "Ant",
  "action"    : "Default",
  "command"   : "ant",
  "keystroke"  : "ctrl-alt-a"
}
```

![Screenshot](https://github.com/hgmarais/process-palette/blob/master/resources/command-palette-keystroke.png?raw=true)

After reloading the configuration the `Ant: Default` command can be run by pressing `Ctrl-Alt-A`.

## User Interface
### Process Palette Panel
Process Palette has a small panel that lists all the commands that are configured. It can be toggled by pressing `Ctrl-Alt-P` or from the menu `Packages|Process Palette|Toggle`.
From here one can see all the commands and even run them.

![Screenshot](https://github.com/hgmarais/process-palette/blob/master/resources/panel-basic.png?raw=true)

Pressing the down arrow in the top right corner will hide the panel. Clicking on the name of the command will cause the panel to switch to showing the output that was produced by the command when it was executed.

![Screenshot](https://github.com/hgmarais/process-palette/blob/master/resources/panel-output-basic.png?raw=true)

From here one can return to the list by pressing the button in the top left corner.

### Notifications
Each time a process is executed a message will be shown in the top right hand corner. A successful execution with an exit status code of 0 will show a success message. Anything other than 0 will show a warning. If the process could not be executed at all then a fatal message is shown. What these messages display can be configured or even disabled completely as will be seen in the Advanced Configuration section.

## Advanced Configuration
The `namespace`, `action`, `command` and `keystroke` aren't the only properties that can be configured. Of these only the `action` and `command` are required, some are optional and some have default values. Many of the properties can also be parameterized with variables from the environment. The following two sections describe the configurable properties and also the variables that can be used to parameterize them.

### Properties
Property|Description|Default
---|---|---
namespace|The namespace under which the command is categorized. This forms part of its identity in the Command Palette.|"Process Palette"
action (required)|The name of the action. This, together with the namespace, gives the command a unique identifier in the Command Palette.|null
command (required)|A string with the name of the command to execute.|null
arguments|An array of strings to pass as arguments to the command.|[ ]
cwd|The working directory from which to execute the command. It doesn't have a default value, but one is automatically determined when the command is executed. If a file is open then that file's project folder is used. If a file isn't open then the first project's folder is used. If there aren't any projects open then the folder of the `process-palette.json` file is used.|null
keystroke|A string describing the shortcut to associate with this command. It can be any combination of `ctrl`, `alt`, `shift` and `cmd` separated with `-` characters.|null

The following properties relate to the output produced by the process. The output can be redirected to a particular target. It can also be formatted depending on whether the process executed successfully or not. Giving any of these a value of `null` will prevent that output from being shown.

Property|Description|Default
---|---|---
outputTarget|Where the output produced by the process should be directed to. It can have one of the following  values: "panel", "editor", "clipboard", "console" or "void". If the value is overridden with `null` then it will default to "void". More on this below.|"panel"
successOutput|The format of the output when the process returned with an exit status of 0.|"{stdout}"
errorOutput|The format of the output when the process returned with a non-0 exit status.|"{stderr}"
fatalOutput|The format of the output when the command could not be executed at all.|"Failed to execute : {fullCommand}\n{stdout}\n{stderr}"

The following properties relate to the messages shown after a command is executed. Giving any of these a value of `null` will prevent that message from being shown.

Property|Description|Default
---|---|---
successMessage|The format of the message when the process returned with an exit status of 0.|"Executed : {fullCommand}"
errorMessage|The format of the message when the process returned with a non-0 exit status.|"Executed : {fullCommand}\nReturned with code {exitStatus}\n{stderr}"
fatalMessage|The format of the message when the command could not be executed at all.|"Failed to execute : {fullCommand}\n{stdout}\n{stderr}"

### Output Targets
The `outputTarget` property specifies where the output produced by the process should be directed to. The following are valid targets:

Target|Description
---|---
void|The output will not be captured at all.
panel|The output will be shown in the Process Palette's panel. Running a command that outputs to the panel will automatically open it.
clipboard|The output will be stored on the clipboard.
editor|The output will be inserted into the open editor at the current cursor position. If an editor is not open the output is lost.
console|The output will be appended to the developer console.

The default value of `outputTarget` is "panel". If it is overridden with `null` then it will default to "void".

### Variables
Some of the properties can be parameterized with variables. Variables are added by enclosing the name of the variable in braces : `{` and `}`. The default values of some of the properties are already parameterized as can be seen in the tables above. There are two types of variables : input and output. Input variables are available before the process executes and output variables are available after it has executed.

The following tables list the input and output variables:

**Input**

Variable|Description
---|---
clipboard | Text currently on clipboard.
fullCommand | The full command along with its arguments. Both the command and arguments will have their variables resolved.
configDirAbsPath | Absolute path of folder where the `process-palette.json` configuration file is that defines this command.
projectPath | Absolute path of project folder. If a file is open then that file's project folder will be used. If there isn't a file open then the first project's folder will be used. If there aren't any projects open then the path of the folder containing the `process-palette.json` file is used.

**Input from editor**

The following input variables are only available if an editor is open. Their values default to an empty string otherwise.

Variable|Description
---|---
fileExt | Extension of file.
fileName | Name of file without extension.
fileNameExt | Name of file with extension.
filePath | Path of file relative to project.
fileDirPath | Path of file's directory relative to project.
fileAbsPath | Absolute path of file.
fileDirAbsPath | Absolute path of file's directory.
selection | Currently selected text.
projectPath | Absolute path of file's project folder.

**Output**

These variables are only available after the process has executed. They can therefore typically be used in the output and message related properties.

Variable|Description
---|---
stdout | Standard output produced by process.
stderr | Standard error output produced by process.
exitStatus | Exit status code returned by process.

### Applying Variables To Properties
The table below shows which properties support input variables and/or output variables:

Property|Input|Output
---|---|---
cwd|yes|no
command|yes|no
arguments|yes|no
successOutput|yes|yes
errorOutput|yes|yes
fatalOutput|yes|yes
successMessage|yes|yes
errorMessage|yes|yes
fatalMessage|yes|yes

The `namespace`, `action` and `keystroke` properties do not support variables.

A useful way of seeing the values of the variables is to add them to one of the output properties and then executing the command. For instance :

```json
"successOutput" : "File path : {filePath}\nProject path : {projectPath}"
```

will show the values of `filePath` and `projectPath` respectively.

Keep in mind that the `arguments` property is an array of strings. Adding variables to arguments should therefore be done such as

```json
"arguments" : ["{fileNameExt}", "{selection}"]
```

in order to pass the file name and the currently selected text as arguments to the command.
