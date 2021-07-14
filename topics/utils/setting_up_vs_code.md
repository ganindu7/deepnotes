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
            "intelliSenseMode": "linux-gcc-arm6",
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

Read [this article][VSCODE-INDEXER-ERRORS] for more info on debugging indexer errors by fixing IntelliSense bugs.


[VSCODE-SITE]: https://code.visualstudio.com/
[VSCODE-INDEXER-ERRORS]: https://code.visualstudio.com/docs/cpp/faq-cpp#:~:text=How%20do%20I%20get%20IntelliSense%20to%20work%20correctly%3F%23
[CPP-PROPERTIES-REF]: https://code.visualstudio.com/docs/cpp/c-cpp-properties-schema-reference
