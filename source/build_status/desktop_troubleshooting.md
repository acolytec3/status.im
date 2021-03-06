---
id: desktop_troubleshooting
title: Troubleshooting
---

# Initial setup issues

## `re-natal` missing

Create a link:

``` bash
ln -sf node_modules/re-natal/index.js re-natal`
```

## Go problem

``` txt
panic: runloop has just unexpectedly stopped

goroutine 50 [running]:
github.com/status-im/status-go/vendor/github.com/rjeczalik/notify.init.0.func1()
        /path-to-status-react/desktop/modules/react-native-status/desktop/StatusGo/src/github.com/status-im/status-go/vendor/github.com/rjeczalik/notify/watcher_fsevents_cgo.go:69 +0x79
created by github.com/status-im/status-go/vendor/github.com/rjeczalik/notify.init.0
        /path-to-status-react/desktop/modules/react-native-status/desktop/StatusGo/src/github.com/status-im/status-go/vendor/github.com/rjeczalik/notify/watcher_fsevents_cgo.go:65 +0x4e
events.js:183
      throw er; // Unhandled 'error' event
```

Related to https://github.com/rjeczalik/notify/issues/139. Solution: re-run.

## App issues

## Node server crashing

`make desktop-server` log:

``` txt
DEBUG [status-im.utils.handlers:36] - Handling re-frame event:  :signal-event {"type":"node.crashed","event":{"error":"node is already running"}}
DEBUG [status-im.ui.screens.events:350] - :event-str {"type":"node.crashed","event":{"error":"node is already running"}}
DEBUG [status-im.utils.instabug:8] - Signal event: {"type":"node.crashed","event":{"error":"node is already running"}}
DEBUG [status-im.ui.screens.events:362] - Event  node.crashed  not handled
```

Solution: prevent starting Node when there is an instance already running.

### Reload JS - blank screen

Console log for `make run-desktop` shows error 533.
Solution: reload again. Still, might hang at `Signing you in...` step (due to node attempted to be restarted). Re-run Figwheel and `make run-desktop`

### ReactButton.qml non-existent property "elide" error upon startup

``` txt
qrc:/qml/ReactButton.qml:33: Error: Cannot assign to non-existent property "elide"
"Component for qrc:/qml/ReactWebView.qml is not loaded"
QQmlComponent: Component is not ready
"Unable to construct item from component qrc:/qml/ReactWebView.qml"
"Can't create QML item for componenet qrc:/qml/ReactWebView.qml"
"RCTWebViewView" has no view for inspecting!
```

Reloading JS does not help, restarting Figwheel/react-native might not as well. Restarting Metro bundler solved it for me.

### After login when several contacts are available: realm errors

1. `attempting to create an object of type 'chat'...`
2. `attempting to create an object of type 'transport'...`
3. Error text containing only the public key.
The realm stack trace follows.

### Error: spawn gnome-terminal ENOENT

In node server log:

``` txt
ignoring exception: Error: read ECONNRESET
```

In react-native log:

``` txt
./run-app.sh: line 72: 56660 Segmentation fault: 11  /path-to-status-react/desktop/bin/Status $args
events.js:183
      throw er; // Unhandled 'error' event
      ^

Error: spawn gnome-terminal ENOENT
    at _errnoException (util.js:992:11)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:190:19)
    at onErrorNT (internal/child_process.js:372:16)
    at _combinedTickCallback (internal/process/next_tick.js:138:11)
    at process._tickCallback (internal/process/next_tick.js:180:9)
```

or

``` txt
Status(7924,0x70000c1cd000) malloc: *** error for object 0x7f8b1539bd10: incorrect checksum for freed object - object was probably modified after being freed.
*** set a breakpoint in malloc_error_break to debug
./run-app.sh: line 72:  7924 Abort trap: 6           /path-to-status-react/desktop/bin/Status $args
events.js:183
      throw er; // Unhandled 'error' event
      ^

Error: spawn gnome-terminal ENOENT
    at _errnoException (util.js:992:11)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:190:19)
    at onErrorNT (internal/child_process.js:372:16)
    at _combinedTickCallback (internal/process/next_tick.js:138:11)
    at process._tickCallback (internal/process/next_tick.js:180:9)
```

### statusgo error during `make run-desktop`

``` txt
Command failed: build(.)sh -e "node_modules/react-native-i18n/desktop;node_modules/react-native-config/desktop;node_modules/react-native-fs/desktop;node_modules/react-native-http-bridge/desktop;node_modules/react-native-webview-bridge/desktop;modules/react-native-status/desktop"
# github.com/status-im/status-go/vendor/github.com/ethereum/go-ethereum/crypto/bn256
../vendor/github.com/ethereum/go-ethereum/crypto/bn256/bn256_fast.go:26: syntax error: unexpected = in type declaration
../vendor/github.com/ethereum/go-ethereum/crypto/bn256/bn256_fast.go:30: syntax error: unexpected = in type declaration
# github.com/status-im/status-go/vendor/github.com/ethereum/go-ethereum/crypto/bn256
vendor/github.com/ethereum/go-ethereum/crypto/bn256/bn256_fast.go:26: syntax error: unexpected = in type declaration
vendor/github.com/ethereum/go-ethereum/crypto/bn256/bn256_fast.go:30: syntax error: unexpected = in type declaration
make[3]: *** [statusgo-library] Error 2
make[2]: *** [modules/react-native-status/desktop/StatusGo/src/github.com/status-im/src/StatusGo_ep-stamp/StatusGo_ep-configure] Error 2
make[1]: *** [modules/react-native-status/desktop/CMakeFiles/StatusGo_ep(.)dir/all] Error 2
make: *** [all] Error 2
```

### inotify errors

upon running `make react-native-desktop` on linux, watchman may indicate: "The user limit on the total number of inotify watches was reached"

This can be fixed by running the below command. Note, changes will only be as valid as the current terminal session.

``` bash
echo 999999 | sudo tee -a /proc/sys/fs/inotify/max_user_watches && echo 999999 | sudo tee -a
/proc/sys/fs/inotify/max_queued_events && echo 999999 | sudo tee -a /proc/sys/fs/inotify/max_user_instances &&
watchman shutdown-server && sudo sysctl -p
```

To persist the changes on Linux edit `/etc/sysctl.conf` by adding the following lines

``` bash
fs.inotify.max_user_watches = 999999
fs.inotify.max_queued_events = 999999
fs.inotify.max_user_instances = 999999
```

then refresh the config with `sudo sysctl -p`

### metro bundler caching issue

From time to time metro bundler for some reasons doesn't recognize changes in JavaScript sources and returns old version of the files on react-native app request. Cleaning of cache should help to solve the issue (usually running `react-native start --reset-cache` from inside a Nix shell to start the bundler should work).

On Linux, the metro bundler saves its cache to the following directories (can be removed manually):
`/tmp/metro-cache-*`

### Where do I get Status Desktop?

Nightlies are available from [here](https://status.im/nightly/)

### How do I updgrade?

* On a Mac: quit Status, download a new nightly, and drag to the Applications Folder when prompted.
* On Linux: just download new nightly and make sure it’s named `Status.AppImage`.

### Status crashed and won’t Start. How to fix?

* On a Mac: open your Activity Monitor (Applications > Utilities > Activity Monitor), search for and kill any `ubuntu-server` process and then restart Status. Restarting your machine should also solve the issue.

### I see a blank screen on Startup. How do I fix it?

* On a Mac: quit Status, then go to `~/Library/Application Support/` and delete any Status directories. Delete the app in `~/Applications`. Then download from the nightlies page and reinstall.

* On linux: you should have the `gnome-keyring` package installed to make your keychain work correctly. `libgnome-keyring0.so` should be available. `gnome-keyring` also requires the package `libgnome-keyring0` to be installed on your system. As an alternative, we currently also support the `kWallet` keychain, but `gnome-keyring` has been tested more.

If you want to delete all Status data to start fresh, do (assuming StatusIm.AppImage is the filename you used to run the app):

``` bash
rm -rf ~/.local/share/Status/
rm -rf ~/.cache/Status/
```
