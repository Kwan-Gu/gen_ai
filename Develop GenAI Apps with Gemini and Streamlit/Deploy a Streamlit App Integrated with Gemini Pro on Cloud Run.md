# Deploy a Streamlit App Integrated with Gemini Pro on Cloud Run

참고자료 : https://www.cloudskillsboost.google/course_templates/978/labs/461321

## Task 1. Run the Application Locally

In this section, you will run the Streamlit application locally in Cloud Shell.

### Clone the Repository

1. Open a new Cloud Shell terminal by clicking on the Cloud Shell icon in the top right corner of the Cloud console.
2. Run the following commands to clone the repo and navigate to `gemini-streamlit-cloudrun` directory in Cloud Shell using the following commands.

```shell
git clone https://github.com/GoogleCloudPlatform/generative-ai.git
```
```shell
cd generative-ai/gemini/sample-apps/gemini-streamlit-cloudrun
```

To run the Streamlit Application, you will need to perform some additional steps.

### Run the application

1. Setup the Python virtual environment and install the dependencies

```shell
python3 -m venv gemini-streamlit
source gemini-streamlit/bin/activate
pip install -r requirements.txt
```

- requirements.txt에는 streamlit, google-cloud-aiplatform, google-cloud-logging 있음

2. Your application requires access to two environment variables:

- `GCP_PROJECT` : This the Google Cloud project ID.
- `GCP_REGION` : This is the region in which you are deploying your Cloud Run app. For e.g. us-central1.

These variables are needed since the Vertex AI initialization needs the Google Cloud project ID and the region. The specific code line from the `app.py` function is shown here: `vertexai.init(project=PROJECT_ID, location=LOCATION)`

In Cloud Shell, execute the following commands

```shell
GCP_PROJECT='Project ID'
GCP_REGION='Region'
```

3. To run the application locally, execute the following command.

```shell
streamlit run app.py \
  --browser.serverAddress=localhost \
  --server.enableCORS=false \
  --server.enableXsrfProtection=false \
  --server.port 8080
```

4. The application will startup and you will be provided a URL to the application. Click the link to view the application in the browser or use Cloud Shell's web preview function to launch the preview page.

5. Adjust the parameters for the story generation and click Generate my story.

6. Navigate back to Cloud Shell and authorize the application to access the Gemini API. Once you have authorized the application, you can navigate back to the application to see the response.

7. After you have finished testing the application, you can stop the application by entering Ctrl + C in Cloud Shell.


## Task 2. Build and Deploy the Application to Cloud Run

In this section, you will deploy the Streamlit Application in Cloud Run.

You will now build the Docker image for the application and push it to Artifact Registry. To do this, you will need one environment variable set that will point to the Artifact Registry name. The commands below will create this Artifact Registry repository for you.

1. In Cloud Shell, execute the following command:

```shell
AR_REPO='gemini-repo'
SERVICE_NAME='gemini-streamlit-app' 
gcloud artifacts repositories create "$AR_REPO" --location="$GCP_REGION" --repository-format=Docker
gcloud builds submit --tag "$GCP_REGION-docker.pkg.dev/$GCP_PROJECT/$AR_REPO/$SERVICE_NAME"
```

2. The final step is to deploy the service in Cloud Run with the image that we had built and had pushed to the Artifact Registry in the previous step.

```shell
gcloud run deploy "$SERVICE_NAME" \
  --port=8080 \
  --image="$GCP_REGION-docker.pkg.dev/$GCP_PROJECT/$AR_REPO/$SERVICE_NAME" \
  --allow-unauthenticated \
  --region=$GCP_REGION \
  --platform=managed  \
  --project=$GCP_PROJECT \
  --set-env-vars=GCP_PROJECT=$GCP_PROJECT,GCP_REGION=$GCP_REGION
```

On successful deployment, you will be provided a URL to the Cloud Run service. You can visit that in the browser to view the Cloud Run application that you just deployed.

Choose the functionality that you would like to check out and the application will prompt the Vertex AI Gemini API and display the responses.

