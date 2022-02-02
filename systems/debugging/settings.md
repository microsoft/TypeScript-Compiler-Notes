Here are some settings that sandersn uses

## Hard-coded file:

This starts much faster than a full test run. I update the compiler flags each time I debug something new.

    {
      "name": "welove.ts",
      "program": "${workspaceRoot}/built/local/tsc.js",
      "request": "launch",
      "skipFiles": [
        "<node_internals>/**"
      ],
      "sourceMaps": true,
      "smartStep": true,
      "args": ["--noEmit", "--skipLibCheck", "--strict", "--exactOptionalPropertyTypes", "/home/nathansa/src/test/welove.ts"],
      "env": {
        "NODE_ENV": "testing"
      },
      "type": "pwa-node",
      "console": "integratedTerminal",
      "outFiles": [ "${workspaceRoot}/built/local/tsc.js"]
    },

I also have a Javascript and JSX target that are nearly identical.

## Currently opened test

This debugs the current test file that's open in VS Code.

    {
      "type": "pwa-node",
      "protocol": "inspector",
      "request": "launch",
      "name": "Mocha Tests (currently opened test)",
      "runtimeArgs": ["--nolazy"],
      "program": "${workspaceRoot}/node_modules/mocha/bin/_mocha",
      "args": [
        "-u",
        "bdd",
        "--no-timeouts",
        "--colors",
        "built/local/run.js",
        "-f",
        // You can change this to be the name of a specific test file (without the file extension)
        // to consistently launch the same test
        "${fileBasenameNoExtension}",
      ],
      "env": {
        "NODE_ENV": "testing"
      },
      "sourceMaps": true,
      "smartStep": true,
      "preLaunchTask": "gulp: tests",
      "console": "integratedTerminal",
      "outFiles": [
        "${workspaceRoot}/built/local/run.js"
      ]
    },

I also made a hard-coded variant, as suggested by the comment.

## Attach to running tsc

This is sometimes useful when I want to build a complex project with `tsc -b` and then debug it.

    {
      "name": "9229",
      "port": 9229,
      "request": "attach",
      "skipFiles": [
        "<node_internals>/**"
      ],
      "type": "pwa-node",
      "env": {
        "NODE_ENV": "testing"
      },
      "sourceMaps": true,
      "outFiles": [ "${workspaceRoot}/built/local/tsc.js"]
    },
