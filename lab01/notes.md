

Storage > Firestore
Select Native Mode
Create Database

get code
```
git clone https://github.com/rosera/pet-theory
cd pet-theory/lab01
npm install @google-cloud/firestore
npm install @google-cloud/logging
```

use files in solution folder

create test data
```
npm install faker
```

set the project id and then set in env
```
gcloud config set project PROJECT_ID
PROJECT_ID=$(gcloud config get-value project)
```

create & import test data
```
node createTestData 1000
npm install csv-parse
node importTestData customers_1000.csv
```

check data in Firestore

see https://cloud.google.com/iam/docs/understanding-roles#predefined_roles
set roles for viewing log and write to source repository
```
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=user:[EMAIL] --role=roles/logging.viewer

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=user:[EMAIL] --role roles/source.writer
```  
