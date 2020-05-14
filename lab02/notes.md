
enable Cloud Firestore API

Firestore > Select Native Mode
Create Database

Go to https://console.firebase.google.com/
Add project
Continue, "Pay as you go"
Continue, deselect Analutics
Add Firebase
Continue


Add web app
Set up Firebase hosting
Register app
Next > Next > Continue to console


setup to edit the file locally
```
git clone https://github.com/rosera/pet-theory.git
cd pet-theory/lab02
npm init --yes
npm install -g firebase-tools
```

link google account with firebase
```
firebase login --no-localhost
```
Use browser to get access code
Copy the access code, paste access code back into the Cloud Shell prompt

set project region to be the same as firebase 
```
gcloud config set compute/region us-central
```

init local firebase project
```
firebase init
```

select Firestore and Hosting
configure Firebase
  - select project id
  - use firestore.rules
  - use firestore.indexes.json
  - N rewrite
  - N overwrite
  

## Setup Authentication

Go to Firebase Console
Authentication > Set up sign-in method button
select Sign-in Providers
enable


## Edit application files

deploy the application
```
firebase deploy
```

Got to Hosting URL

deploy codes from solutions

Go to the Firebase Console and click Database to view data



