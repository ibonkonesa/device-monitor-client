# Device monitor client showcase

This example shows how to integrate with Device Monitor Daemon https://github.com/ibonkonesa/device-monitor-daemon
Based in Quasar Cordova project

## Install

Clone this repository

```sh
$ git clone https://github.com/ibonkonesa/device-monitor-client.git
```

Install Quasar dependencias

```sh
$ npm install
```

There is a bug with quasar cordova repos, execute this in order to reinstall cordova mode and keep cordova config (Seen here: https://forum.quasar-framework.org/topic/2073/restore-from-git/6)

```sh
$ cp -r src-cordova src-cordova-bak
$ quasar mode -r cordova
$ quasar mode -a cordova
$ rm -r src-cordova
$ mv src-cordova-bak src-cordova
```

Install cordova dependencies

```sh
$ cd src-cordova 
$ npm install 
$ cd ..

```

## Firebase

You have to create a google-services.json file if you want to compile to Android.
You have to create a GoogleService-Info.plist file if you want to compile to iOS.

Create Android or iOS in your Firebase Project Console and put this files at /src-cordova folder.

You must choose an uniq app identifier (ex: com.ibonkonersa.device-monitor) when you add both apps. Then, you have to update some files with this identifier:

- package.json: Update cordovaId property
- src-cordova/package.json: Update name property
- src-cordova/config.xml: Update widget id attribute

Finally, in order to access to the realtime database, go to Firebase Project Console and create a web app. A config JSON will be created. Just place this data at src/pages/index.vue (just below import Firebase line)


## Development

Once you have configured all stuf, just go to root path and execute

```sh
$ quasar dev -m cordova -T [ios|android]
```





