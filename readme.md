# Corator

**Coraza-based WAF with Transparent File Interception for Digital Forensics**

[![Go Version](https://img.shields.io/badge/go-1.25+-blue.svg)](https://golang.org/)


---

## The Problem: Digital Evidence in Stateless Environments

In modern, stateless environments like **Kubernetes**, collecting digital evidence for forensic analysis is a significant challenge. When a pod is terminated or an application crashes, any files uploaded by users—which could be crucial evidence—are often lost forever.

Forcing developers to redesign their applications to upload every file to a persistent storage like S3 is often impractical, costly, and significantly slows down development cycles.

## The Solution: Corator 🕵️‍♂️

**Corator** is a high-performance reverse proxy that solves this problem without requiring a single line of code change in your existing applications. It acts as a transparent layer in front of your service, providing two key functions:

1.  **Transparent File Interception**: It automatically "sniffs" out and intercepts any file within incoming HTTP requests, whether it's a standard file upload (`multipart/form-data`) or a Base64-encoded string hidden in a form field.
2.  **Full WAF Protection**: It uses the powerful **Coraza WAF engine** to inspect all traffic, protecting your application from common web attacks based on the OWASP Core Rule Set.

Intercepted files are then securely and automatically uploaded to a persistent, centralized location (S3-compatible storage or a local folder), complete with metadata like timestamps and request IDs for easy tracking.

---

## How It Works

Corator sits between the client and your application. Every request passes through it, allowing for real-time inspection and interception before the request even hits your service. The file interception and upload process runs asynchronously to ensure minimal impact on performance.


*Diagram Alur Sederhana:*
`Client -> Corator (WAF + File Interceptor) -> Backend Application`
`          |`
`          v`
`          -> [ S3 / Local Storage ]`

---

## Key Features

-   🔎 **Transparent Interception**: Captures files without needing any changes to the backend application's architecture.
-   📂 **Multipart & Base64 Detection**: Intelligently detects standard file uploads and Base64-encoded files in any request field.
-   🛡️ **Enterprise-Grade WAF**: Full integration with Coraza WAF and the OWASP Core Rule Set.
-   💾 **Persistent Evidence Storage**: Securely uploads intercepted files to S3-compatible storage or a local disk.
-   📝 **Structured JSON Logging**: Detailed logging for every intercepted file, ready for integration with SIEMs like Elasticsearch.
-   ⚙️ **Highly Configurable**: Control everything via environment variables, from enabling detectors to configuring storage endpoints.

---

## Use Case: Solving Forensic Challenges in Kubernetes

Corator is ideally deployed as a **sidecar container** within your Kubernetes pods. This pattern allows Corator to intercept all traffic destined for your application container seamlessly.

By doing this, you instantly gain a robust forensic evidence collection mechanism for every pod, ensuring that no matter what happens to the pod, the evidence is already safe in your S3 bucket.



---

## Getting Started

### Prerequisites
-   Go 1.18 or higher

### Installation
1.  Clone the repository:
    ```bash
    git clone [https://github.com/luhtaf/corator.git](https://github.com/luhtaf/corator.git)
    cd corator
    ```
2.  Install dependencies:
    ```bash
    go mod tidy
    ```

---

## Configuration

Corator is configured entirely through **environment variables**.

| Variable                      | Description                                                  | Default Value                |
| ----------------------------- | ------------------------------------------------------------ | ---------------------------- |
| **Server** |                                                              |                              |
| `SERVER_LISTEN_ADDRESS`         | The address and port for Corator to listen on.                 | `:8080`                      |
| `SERVER_BACKEND_URL`            | The full URL of the backend application to proxy traffic to.   | `http://localhost:3000`      |
| **WAF** |                                                              |                              |
| `WAF_CORAZA_CONFIG_PATH`        | The absolute path to your `coraza.conf` file.                | (none)                       |
| **Detectors** |                                                              |                              |
| `DETECTORS_ENABLE_FILE`         | Enable the `multipart/form-data` file detector.              | `false`                      |
| `DETECTORS_ENABLE_BASE64`       | Enable the Base64-encoded file detector.                     | `false`                      |
| **Uploader** |                                                              |                              |
| `UPLOADER_TYPE`                 | The destination for intercepted files. Can be `local` or `s3`. | `local`                      |
| `UPLOADER_LOCAL_PATH`           | The directory path for local storage.                        | `/tmp/uploads`               |
| `UPLOADER_S3_ENDPOINT`          | The endpoint URL for S3-compatible storage.                  | (none)                       |
| `UPLOADER_S3_BUCKET`            | The S3 bucket name.                                          | (none)                       |
| `UPLOADER_S3_REGION`            | The S3 region.                                               | (none)                       |
| `UPLOADER_S3_ACCESS_KEY`        | The S3 access key.                                           | (none)                       |
| `UPLOADER_S3_SECRET_KEY`        | The S3 secret key.                                           | (none)                       |
| **Logger** |                                                              |                              |
| `LOGGER_ENABLE_FILE`            | Enable logging to a local file.                              | `false`                      |
| `LOGGER_ENABLE_ELASTIC`         | Enable logging to Elasticsearch.                             | `false`                      |
| `LOGGER_FILE_PATH`              | The path for the local log file.                             | `/tmp/interceptor.log`       |
| `LOGGER_ELASTIC_URLS`           | Comma-separated list of Elasticsearch URLs.                  | (none)                       |
| `LOGGER_ELASTIC_INDEX`          | The Elasticsearch index name.                                | `coraza-interceptor`         |


---

## Running the Application

1.  **Set your environment variables: (.env)**

2.  **Run the application:**
    ```bash
    go run [github.com/luhtaf/corator/cmd/main.go](https://github.com/luhtaf/corator/cmd/main.go)
    ```

### Testing
From another terminal, send a test file upload:
```bash
# Create a dummy file
echo "forensic evidence" > evidence.txt

# Send the request to Corator
curl -X POST http://localhost:8080/upload -F "file=@evidence.txt" -F "user=test"

Check the /tmp/corator_uploads directory for the intercepted file and /tmp/corator.log for the log entry.