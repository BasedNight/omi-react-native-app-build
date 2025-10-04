# Omi React Native Mobile App Build Instructions

More of a visual learner? Watch the accompanying [tutorial video](https://www.youtube.com/watch?v=Ju6DTeXqivc) instead.

## Setup build tools

### [EAS-like Local Builder](https://github.com/erayalakese/eas-like-local-builder)

1. Pull the Docker image: `docker pull erayalakese/eas-like-local-builder`.
<!-- the following steps are optional, but if you do not perform them, you will have to log in to Expo each time you build the mobile app -->
2. Create a Docker volume to persist Expo authentication: `docker volume create expo-data`
3. Login to Expo: `docker container run -v expo-data:/root/.expo -it erayalakese/eas-like-local-builder eas login`

### [Bundletool](https://github.com/google/bundletool)
1. Clone the [bundletool-docker repository](https://github.com/BasedNight/bundletool-docker): `git clone https://github.com/BasedNight/bundletool-docker.git`
2. Navigate to the project directory: `cd bundletool-docker`
3. Build the Dockerfile: `docker build -t bundletool:1.18.2 --build-arg BUNDLETOOL_VERSION=1.18.2 .`
4. Make sure the container works correctly: `docker run --rm bundletool:1.18.2 version`

## Building the app

1. Clone the official [Omi repository](https://github.com/BasedHardware/omi): `git clone https://github.com/BasedHardware/omi.git`
2. Navigate to example app directory: `cd omi/sdks/react-native/example`
<!-- changes so that the project will work with expo/eas -->
3. Create .gitignore and add the android directory to it: `echo "android/" >> .gitignore`
4. In `package.json`, upgrade the `react-native` dependency from version "0.73.2" -> "0.73.6"
<!-- the following change must be made so that the build will pull in omi-react-native from npm rather than from the local filesystem, which is not exposed to the Docker build -->
5. In `package.json`, change the `@omiai/omi-react-native` dependency from "file:.." to "1.0.2"
6. Install the project dependencies: `npm install`
7. Initialize the Expo project: `docker container run -e EAS_NO_VCS=1 -e PROFILE=production -v expo-data:/root/.expo -v "$PWD":/app -w /app -it erayalakese/eas-like-local-builder eas init`
<!-- when prompted, create a new Expo project and link it to the existing project folder -->
8. Build the app: `docker container run -e EAS_NO_VCS=1 -e PROFILE=production -v expo-data:/root/.expo -v "$PWD":/app -w /app -it erayalakese/eas-like-local-builder`
<!-- when prompted, generate a new Android keystore. We will use this later to produce a signed APK that can be installed on an Android device. -->

## Use bundletool to produce a signed APK from the built AAB file
Dependencies: make, unzip

1. Copy the build output (ex. build-1759368339874.aab) from the example app directory to the bundletool-docker directory: `cp build-1759430424374.aab ~/omi-react-native-app-build/bundletool-docker/`
2. Navigate to the bundletool-docker directory: `cd ~/omi-react-native-app-build/bundletool-docker/`
3. Rename the build output to something distinct and easy to remember: `mv build-1759430424374.aab example.aab`
4. In your browser, log in to Expo.dev.
5. Navigate to your newly-created project (ex. omi-react-native-example)
6. In the sidebar, under "Project Settings," click "Credentials"
7. Click on your application identifier.
8. Under "Android upload keystore," click the … next to the credentials and click "Download".
9. Save the credentials ZIP in the bundletool-docker directory.
10. Unzip the credentials, creating two files: an Android keystore file (.jks) and a Markdown file (.md) containing your credential values for signing the APK.
11. Rename the Android keystore file from `@<username>…-keystore.bak.jks` to `keystore.bak.jks`.
12. Create a .env file for storing your signing credentials: `touch .env`
13. Copy the credential values from `…-credentials.md` to the appropriate environment variables in `.env`, like so:

```
KEYSTORE_PASS=3b8763bf… (Android upload keystore password)
KEYSTORE_KEY_ALIAS=32854ce8… (Android key alias)
KEY_PASS=700213b00… (Android key password)
```

**CHECKPOINT**: Before running the Makefile, you should have the following files in your bundletool-docker directory:
```
example.aab (build output)
.env (signing credentials stored as environment variables)
keystore.bak.jks (Android upload keystore)
```
14. Run the Makefile on the build output and clean up build artifacts: `make example.apk && make clean`
