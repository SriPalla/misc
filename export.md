To achieve this setup, you'll need to create a Google Cloud Function in Python 3.12 that triggers when a new file is added to a Google Cloud Storage bucket. This Cloud Function will then invoke a Cloud Run job, passing the filename as an argument. The Cloud Run job, which is a Kotlin Spring Boot application, will read the filename from the arguments.

### Step 1: Create a Google Cloud Function (Python 3.12)

Here’s the Python code for the Google Cloud Function:

```python
import os
from google.cloud import storage, run_v2
from google.oauth2 import service_account

# Set up your project and service account details
PROJECT_ID = "your-project-id"
LOCATION = "us-central1"  # Adjust as needed
SERVICE_ACCOUNT_KEY = "path/to/your/service-account-key.json"
CLOUD_RUN_SERVICE_NAME = "your-cloud-run-job-name"

# Initialize Cloud Run and Storage clients
credentials = service_account.Credentials.from_service_account_file(SERVICE_ACCOUNT_KEY)
run_client = run_v2.JobsClient(credentials=credentials)
storage_client = storage.Client(credentials=credentials)

def trigger_cloud_run(data, context):
    # Extract the bucket name and file name from the Cloud Storage event
    bucket_name = data['bucket']
    file_name = data['name']

    # Define the Cloud Run job request
    job_request = run_v2.RunJobRequest(
        name=f"projects/{PROJECT_ID}/locations/{LOCATION}/jobs/{CLOUD_RUN_SERVICE_NAME}",
        args=[file_name]
    )

    # Invoke the Cloud Run job
    response = run_client.run_job(request=job_request)

    print(f"Cloud Run Job triggered for file: {file_name}")
    print(f"Response: {response}")

    return response
```

**Notes:**
- Replace `"your-project-id"`, `"us-central1"`, `"path/to/your/service-account-key.json"`, and `"your-cloud-run-job-name"` with your actual project ID, location, service account key file path, and Cloud Run job name.

### Step 2: Deploy the Google Cloud Function

1. Save the above code in a `main.py` file.
2. Create a `requirements.txt` file with the following content:

    ```
    google-cloud-storage
    google-cloud-run
    google-auth
    ```

3. Deploy the function using the following command:

    ```bash
    gcloud functions deploy trigger-cloud-run-job \
    --runtime python312 \
    --trigger-resource your-bucket-name \
    --trigger-event google.storage.object.finalize \
    --entry-point trigger_cloud_run \
    --region us-central1 \
    --service-account your-service-account@your-project-id.iam.gserviceaccount.com
    ```

### Step 3: Modify the Cloud Run Job (Kotlin Spring Boot)

In your Kotlin Spring Boot application, you need to read the argument passed by the Cloud Run job. Here’s how you can modify the Spring Boot application to capture and process the filename:

```kotlin
import org.springframework.boot.ApplicationArguments
import org.springframework.boot.ApplicationRunner
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class CloudRunJobApplication : ApplicationRunner {

    override fun run(args: ApplicationArguments) {
        val fileName = args.getOptionValues("fileName")?.firstOrNull()

        if (fileName != null) {
            println("Processing file: $fileName")
            // Add your file processing logic here
        } else {
            println("No filename provided")
        }
    }
}

fun main(args: Array<String>) {
    runApplication<CloudRunJobApplication>(*args)
}
```

In this code:
- The `ApplicationArguments` object captures the arguments passed to the Cloud Run job.
- The filename is retrieved using `args.getOptionValues("fileName")`.

### Step 4: Deploy the Cloud Run Job

1. Build your Kotlin Spring Boot application:

    ```bash
    ./gradlew bootBuildImage
    ```

2. Deploy it to Cloud Run:

    ```bash
    gcloud run jobs create your-cloud-run-job-name \
    --image gcr.io/your-project-id/your-image-name \
    --region us-central1
    ```

### Summary
- The Cloud Function is triggered by a new file in a storage bucket, invokes the Cloud Run job, and passes the filename as an argument.
- The Cloud Run job, implemented as a Kotlin Spring Boot application, reads and processes the filename.
