## cordova-macos-test

Minimal Cordova macOS project to reproduce a failure in [cordova-osx@6.0.0](https://github.com/apache/cordova-osx/blob/master/RELEASENOTES.md#600-jun-15-2020) and [cordova@10.0.0](https://github.com/apache/cordova-cli/blob/master/RELEASENOTES.md#1000-jul-31-2020) to restore plugins from `package.json`.

### Reproduction steps

The Cordova project is configured to install and run [cordova-plugin-device](https://github.com/apache/cordova-plugin-device) in a macOS app.

```bash
yarn install
yarn cordova platform add osx
yarn cordova run
```

### Expected behavior
The app restores cordova-plugin-device from `package.json` and displays the platform on which it runs.

### Actual behavior
The app correctly installs cordova-plugin-device into the native macOS (Xcode) project, but fails to make it accessible to the WebView. The app builds and runs successfully, but JS and native plugin code fail to run.

Output logs
```
** BUILD SUCCEEDED **

Starting: cordova-osx-test/platforms/osx/build/cordova-macos-test.app/Contents/MacOS/cordova-macos-test
2020-09-10 13:14:02.368 cordova-macos-test[75571:1896761] navigating to file:///cordova-osx-test/platforms/osx/build/cordova-macos-test.app/Contents/Resources/www/index.html
2020-09-10 13:14:02.374 cordova-macos-test[75571:1896761] WebStoragePath is '~/Library/Application Support/org.macos.test', modify in config.xml.
2020-09-10 13:14:02.671 cordova-macos-test[75571:1896761] ERROR: Plugin 'Device' not found, or is not a CDVPlugin. Check your plugin mapping in config.xml.
2020-09-10 13:14:07.624 cordova-macos-test[75571:1896761] deviceready has not fired after 5 seconds.
2020-09-10 13:14:07.624 cordova-macos-test[75571:1896761] Channel not fired: onCordovaInfoReady
```

### Observations
* Manually removing and adding cordova-device-plugin successfully configures and runs the plugin code:
  ```bash
  yarn cordova plugin rm cordova-plugin-device
  yarn cordova plugin add cordova-plugin-device
  yarn cordova run
  ```

    Output logs:
    ```
    ** BUILD SUCCEEDED **
    Starting: cordova-osx-test/platforms/osx/build/cordova-macos-test.app/Contents/MacOS/cordova-macos-test
    2020-09-10 13:25:27.845 cordova-macos-test[76539:1918183] navigating to file:///cordova-osx-test/platforms/osx/build/cordova-macos-test.app/Contents/Resources/www/index.html
    2020-09-10 13:25:27.847 cordova-macos-test[76539:1918183] WebStoragePath is '~/Library/Application Support/org.macos.test', modify in config.xml.
    2020-09-10 13:25:28.057 cordova-macos-test[76539:1918183] Running cordova-osx@6.0.0
    __2020-09-10 13:25:28.057 cordova-macos-test[76539:1918183] DEVICE INFO Mac OS X__
    ```


* Restoring plugins from `package.json` vs manually adding them results in several configuration differences in:
  - `platforms/osx/cordova-macos-test/config.xml` is missing plugin declarations, i.e.
     ```diff
    - <feature name="Device">
    -   <param name="ios-package" value="CDVDevice" />
    - </feature>
    ```
  - `platforms/osx/osx.json` is missing plugin declarations:
    ```diff
    "config_munge": {
    +    "files": {}
    -    "files": {
    -      "config.xml": {
    -        "parents": {
    -          "/*": [
    -            {
    -              "xml": "<feature name=\"Device\"><param name=\"ios-package\" value=\"CDVDevice\" /></feature>",
    -              "count": 1
    -            }
    -          ]
    -        }
    -      }
    -    }
       },
    -  "installed_plugins": {},
    -  "dependent_plugins": {}
    -  "installed_plugins": {
    -    "cordova-plugin-device": {}
    -  },
    -  "dependent_plugins": {},
    -  "modules": [
    -    {
    -      "id": "cordova-plugin-device.device",
    -      "file": "plugins/cordova-plugin-device/www/device.js",
    -      "pluginId": "cordova-plugin-device",
    -      "clobbers": [
    -        "device"
    -      ]
    -    }
    -  ],
    -  "plugin_metadata": {
    -    "cordova-plugin-device": "2.0.3"
    -  }
     }
    ```
* When multiple plugins are restored from `package.json`, the last installed plugin overrides the configuration in `platforms/osx/www/cordova_plugins.js`.
