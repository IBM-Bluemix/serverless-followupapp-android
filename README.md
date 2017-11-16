# serverless-followupapp-android

:warning: work in progress

A mobile follow-up app using a serverless backend in Java


## Provision a Cloudant NoSQL DB service

1. Create service instance

   ```
   bx cf create-service cloudantNoSQLDB Lite serverless-followupapp-db
   ```

1. Create credentials to use the service with Cloud Functions

   ```
   bx cf create-service-key serverless-followupapp-db for-cli
   ```

1. Retrieve the credentials

   ```
   bx cf service-key serverless-followupapp-db for-cli
   ```

1. Create databases

   ```
   curl -X PUT https://account:password@acccount-bluemix.cloudant.com/feedback
   curl -X PUT https://account:password@acccount-bluemix.cloudant.com/moods
   curl -X PUT https://account:password@acccount-bluemix.cloudant.com/users
   ```

## Provision a Watson Tone Analyzer service

1. Create instance

   ```
   bx cf create-service tone_analyzer standard serverless-followupapp-tone
   ```

1. Create credentials to use the service with Cloud Functions

   ```
   bx cf create-service-key serverless-followupapp-tone for-cli
   ```

1. Retrieve the credentials

   ```
   bx cf service-key serverless-followupapp-tone for-cli
   ```

## Provision an App ID service

1. Create instance

   ```
   bx cf create-service AppID "Graduated tier" serverless-followupapp-appid
   ```

1. Create credentials to use the service with Cloud Functions

   ```
   bx cf create-service-key serverless-followupapp-appid for-cli
   ```

1. Retrieve the credentials

   ```
   bx cf service-key serverless-followupapp-appid for-cli
   ```

## Provision a Push Notifications service

1. Create instance

   ```
   bx cf create-service imfpush lite serverless-followupapp-mobilepush
   ```

1. Create credentials to use the service with Cloud Functions

   ```
   bx cf create-service-key serverless-followupapp-mobilepush for-cli
   ```

1. Retrieve the credentials

   ```
   bx cf service-key serverless-followupapp-mobilepush for-cli
   ```

## Clone the mobile app project

https://github.com/IBM-Bluemix/serverless-followupapp-android

## Configure Google/Firebase Push Notification

1. Create a new project in Firebase console

1. In the Settings, in the General tab, add two applications:
   1. one with the package name set to: com.ibm.mobilefirstplatform.clientsdk.android.push
   1. and one with the package name set to: serverlessfollowup.app

1. Download the google-services.json from Firebase console and place this file in the android/app folder of the repository

## Configure Push Notifications service

1. Retrieve the Sender ID and Server Key from the Cloud Messaging tab in the Settings for your Firebase project

1. In IBM Cloud console, set the Sender ID and API Key (Server Key) using the values from the Firebase project

## Create Serverless actions.

1. From the root of the project, build the actions

   ```
   ./android/gradlew -p actions clean jar
   ```

1. Copy template.local.env to local.env

   ```
   cp template.local.env local.env
   ```

1. Get the credentials for Cloudant, Tone Analyzer, Push Notifications and App ID services from the IBM Cloud dashboard (or the output of the bx commands we ran before) and replace placeholders in local.env with corresponding values. These properties will be injected into a package so that all actions can get access to the database.

1. Deploy the actions to OpenWhisk

   ```
   ./deploy.sh --install
   ```

### A package for all actions

-> has access to all service credentials

### Register the user as a feedback submitter

a sequence exposed as a web action PUT verb
  input has header Authorization: Bearer {accessToken} {idToken}
  input has device ID to be used by push notifications

action **validate_token**
  retrieve accessToken and idToken from Authorization header
  verify accessToken and idToken either by using the public key for the App ID instance
  output the input args + decoded tokens

action **create_user**
  insert or update the user record in the database
  save the user device ID for push notifications
  use the "sub" parameter of the tokens as user identifier
  output the created/updated user

### Post feedback

a sequence exposed as a web action PUT verb
  required Authorization: Bearer {accessToken}

action **validate_token**
action **put_feedback**
  retrieve the user associated with the feedback by looking at the sub of the Authorization token
  store a new feedback document, setting the user_id

### Analyze feedback

with a trigger in response to a new document in the feedback database
  load the feedback
  load the user
  call tone analysis
  find the associated mood
  send a push notification to the user

## Configure the mobile application

1. Open the Android folder project in Android Studio 2.3.3 or later

1. Edit android/app/src/main/res/values/credentials.xml and fill in the blanks with values from credentials

1. Build the project

1. Start the application on a real device or with an emulator. For the emulator to receive push notifications, make sure to pick an image with the Google APIs and to log in with a Google account within the emulator.

1. Watch the Cloud Functions in the background

   ```
   bx wsk activation poll
   ```
