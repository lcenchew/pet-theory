
Enable Cloud Run API

use package.json from solution

```
npm install express
npm install body-parser
npm install child_process
npm install @google-cloud/storage
```

Build the container and deploy
view in Container Registry > Images
```
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter

gcloud beta run deploy pdf-converter \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter \
  --platform managed \
  --region us-central1 \
  --no-allow-unauthenticated

```

set SERVICE_UR in env
```
SERVICE_URL=$(gcloud beta run services describe pdf-converter --platform managed --region us-central1 --format="value(status.url)")
echo $SERVICE_URL
```

test access the service
```
curl -X POST $SERVICE_URL

curl -X POST -H "Authorization: Bearer $(gcloud auth print-identity-token)" $SERVICE_URL
```

create the buckets
```
gsutil mb gs://$GOOGLE_CLOUD_PROJECT-upload
gsutil mb gs://$GOOGLE_CLOUD_PROJECT-processed
```

create notification to pub/sub when file is uploaded
```
gsutil notification create -t new-doc -f json -e OBJECT_FINALIZE gs://$GOOGLE_CLOUD_PROJECT-upload
```

create service account and set permission for pub/sub to invoke cloud run
```
gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"

gcloud beta run services add-iam-policy-binding pdf-converter \
  --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
  --role=roles/run.invoker \
  --platform managed \
  --region us-central1

```

list project to get Project Number
```
gcloud projects list
```

use Project Number to create Cloud Pub/Sub authentication tokens
```
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
  --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
  --role=roles/iam.serviceAccountTokenCreator
```

create a subscription 
```
gcloud beta pubsub subscriptions create pdf-conv-sub \
  --topic new-doc \
  --push-endpoint=$SERVICE_URL \
  --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

put some test file
```
gsutil -m cp gs://spls/gsp644/* gs://$GOOGLE_CLOUD_PROJECT-upload
```

check under Logging, 'Cloud Run Revision' filter


Use the Dockerfile in solution
build container
```
gcloud builds submit \
  --tag gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter
```

Use the index.js in solution
```
gcloud beta run deploy pdf-converter \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/pdf-converter \
  --platform managed \
  --region us-central1 \
  --memory=2Gi \
  --no-allow-unauthenticated \
  --set-env-vars PDF_BUCKET=$GOOGLE_CLOUD_PROJECT-processed
```

test it
```
curl -X POST -H "Authorization: Bearer $(gcloud auth print-identity-token)" $SERVICE_URL
```

upload some example to bucket
```
gsutil -m cp gs://spls/gsp644/* gs://$GOOGLE_CLOUD_PROJECT-upload
```

Go to Storage to see processed PDF



