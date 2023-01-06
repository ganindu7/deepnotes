---
layout: default
title: setting up vscode
# nav_order: 3 
permalink: /topics/utils/vs_code
parent: Utilities
---

## Setting up [VSCode][VSCODE-SITE]

Goto the website and follow the instructions. **(It's much easier as of writing)**


For code completion via remote
add the line `set( CMAKE_EXPORT_COMPILE_COMMANDS ON ` to your CMakeLists.txt 
in the root of the project add a [`c_cpp_properties.json`][CPP-PROPERTIES-REF] and in there put ` "compileCommands": "${workspaceFolder}/build/compile_commands.json"` in the appropriate location.


Example setup in my remote arm target. 

```
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/include/**",
                "/usr/include/glib-2.0/",
                "/usr/include/gstreamer-1.0"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/aarch64-linux-gnu-g++",
            "cStandard": "c11",
            "cppStandard": "c++14",
            "intelliSenseMode": "linux-gcc-arm64",
            "configurationProvider": "ms-vscode.cmake-tools",
            "compileCommands": "${fileWorkspaceFolder}/build/compile_commands.json"  
        }
    ],
    "version": 4
}

```

You can also do something like this in the `CMakeLists.txt`

```cmake
IF(EXISTS "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json")
  EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E create_symlink "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json" "${PROJECT_SOURCE_DIR}/compile_commands.json")
  message(STATUS "******* symlinks generated *******")
else()
  message(WARNING "**** please rerun the cmake command to generare files for vscode autocomplete! ****") 
ENDIF()

```

*Note: You may need to rerun the `cmake` command (e.g. `cmake ../` from `build` directory) if you don't see the symlink in the project root.*


You can also set the intelliSense mode to clang-arm64 e.g. `"intelliSenseMode": "linux-clang-arm64"`

However with `compileCommands` correctly set and and intelliSense database set (reset the intelliSense database and see before major changes if you think it is somehow corrupted)


### Integrated Shell Settings for remote machines 

The settings file can be usually found on `~/.vscode-server/data/Machine/settings.json ` 


This is my custom profile (named gbash)

```
{
    "terminal.integrated.automationProfile.linux": {
    "path": "/bin/bash --login"
    },
    "terminal.integrated.profiles.linux": {
        "gbash": {
            "path": "/bin/bash --login -c",
            "icon": "terminal-bash"
        },

    },
    // "terminal.integrated.shell.linux": "/bin/bash --login -c",

    "terminal.integrated.defaultProfile.linux":"gbash"
}
```

```
{
    "terminal.integrated.automationProfile.linux": {
        "path": "/bin/bash --login -c"
    },
    "terminal.integrated.profiles.linux": {
        "gbash": {
            "path": "/bin/bash --login -c",
            "icon": "terminal-bash"
        }
    },
    "terminal.integrated.defaultProfile.linux": "gbash",
    "editor.fontLigatures": false,
    "terminal.integrated.defaultProfile.osx": ""
}

```

More info on this can be found on `defaultSettings.json` here is an excerpt 

```
    // The default profile used on Linux. This setting will currently be ignored if either `terminal.integrated.shell.linux` or `terminal.integrated.shellArgs.linux` are set.
    //  - null: Automatically detect the default
    //  - sh: $(terminal) sh
    // - path: /bin/sh
    //  - bash: $(terminal-bash) bash
    // - path: /usr/bin/bash
    //  - bash (2): $(terminal) bash (2)
    // - path: /usr/bin/bash
    //  - rbash: $(terminal) rbash
    // - path: /bin/rbash
    //  - rbash (2): $(terminal) rbash (2)
    // - path: /usr/bin/rbash
    //  - dash: $(terminal) dash
    // - path: /bin/dash
    //  - dash (2): $(terminal) dash (2)
    // - path: /usr/bin/dash
    //  - JavaScript Debug Terminal: $($(debug)) JavaScript Debug Terminal
    // - extensionIdentifier: ms-vscode.js-debug
    "terminal.integrated.defaultProfile.linux": null,

```

Basically this means that at the time of writing "terminal.integrated.shell.linux" has precedence. When examining the `default.Settings.json` we can see that there is no default setting  (`"terminal.integrated.defaultProfile.linux": null,`). Therefore we can create `"terminal.integrated.profiles.linux"` set a custom profile as `gbash` and then assign it with "terminal.integrated.defaultProfile.linux":"gbash". After doing this I did not get any errors when trying to run interactively. 

```
"terminal.integrated.shell.linux": "/bin/bash --login -c",
```

determines what the shell executable command is. 


#### Troubleshooting 

`The terminal process failed to launch: Path to shell executable "/bin/bash --login c" does not exist.`

This may happen even if you correctly set the settings file in the remote container this may be a bug) 

* goto: File > Preferences > Settings
* Click on Terminal 
* Scroll to Integrated > Default profile: Linux 
* set it to bash (usually default value is `None`)
* then again in remote settings you will notice that the integrated profile is now changed to `bash`

For me at this point the error goes away and If I have set the `gbash` profile definition as above I change the value from `bash to `gbash`. However I'm not sure if this makes any effect.    


click on Terminal 






Read [this article][VSCODE-INDEXER-ERRORS] for more info on debugging indexer errors by fixing IntelliSense bugs.


[VSCODE-SITE]: https://code.visualstudio.com/
[VSCODE-INDEXER-ERRORS]: https://code.visualstudio.com/docs/cpp/faq-cpp#:~:text=How%20do%20I%20get%20IntelliSense%20to%20work%20correctly%3F%23
[CPP-PROPERTIES-REF]: https://code.visualstudio.com/docs/cpp/c-cpp-properties-schema-reference
