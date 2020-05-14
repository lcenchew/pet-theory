
create a topic
```
gcloud pubsub topics create new-lab-report
```

enable Cloud Run
```
gcloud services enable run.googleapis.com
```

get & setup code
```
git clone https://github.com/rosera/pet-theory.git
cd pet-theory/lab05/lab-service
npm install express
npm install body-parser
npm install @google-cloud/pubsub
```

use package.json, index.js, Dockerfile, deploy.sh from solution

key part of index.js
```
    const labReport = req.body;
    await publishPubSubMessage(labReport);
```

build and deploy the lab service
```
chmod u+x deploy.sh
./deploy.sh
```
Find under Cloud Run


set env for the report service url
```
export LAB_REPORT_SERVICE_URL=$(gcloud run services describe lab-report-service --platform managed --region us-central1 --format="value(status.address.url)")
echo $LAB_REPORT_SERVICE_URL
```

make some test
```
chmod u+x post-reports.sh
./post-reports.sh
```
see under Cloud Run > LOGS

## Email service

```
cd ~/pet-theory/lab05/email-service
npm install express
npm install body-parser
```

use files from solution
```
chmod u+x deploy.sh
./deploy.sh
```

configure pub/sub to trigger email
```
gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"
gcloud run services add-iam-policy-binding email-service \
  --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
  --role=roles/run.invoker \
  --region us-central1 \
  --platform managed
```

put project number into env
```
PROJECT_NUMBER=$(gcloud projects list --filter="qwiklabs-gcp" --format='value(PROJECT_NUMBER)')
```

enable token creation
```
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
  --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
  --role=roles/iam.serviceAccountTokenCreator
```

```
EMAIL_SERVICE_URL=$(gcloud run services describe email-service --platform managed --region us-central1 --format="value(status.address.url)")
echo $EMAIL_SERVICE_URL
```

subscribe to topic
```
gcloud pubsub subscriptions create email-service-sub --topic new-lab-report --push-endpoint=$EMAIL_SERVICE_URL --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

test it!
```
~/pet-theory/lab05/lab-service/post-reports.sh
```

## SMS service

use files from solution
```
cd ~/pet-theory/lab05/sms-service
npm install express
npm install body-parser

chmod u+x deploy.sh
./deploy.sh
```

same steps as email
``
gcloud run services add-iam-policy-binding sms-service \
  --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
  --role=roles/run.invoker \
  --region us-central1 \
  --platform managed

SMS_SERVICE_URL=$(gcloud run services describe sms-service --platform managed --region us-central1 --format="value(status.address.url)")

echo $SMS_SERVICE_URL

gcloud pubsub subscriptions create sms-service-sub \
  --topic new-lab-report \
  --push-endpoint=$SMS_SERVICE_URL \
  --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

~/pet-theory/lab05/lab-service/post-reports.sh

``

https://medium.com/google-cloud/cloud-run-as-an-internal-async-worker-480a772686e
