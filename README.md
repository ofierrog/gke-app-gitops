# Production-Grade GitOps on GKE with Terraform, GitHub Actions & Argo CD

This project demonstrates a complete, secure, and automated CI/CD pipeline for deploying containerized applications to Google Kubernetes Engine (GKE). It is built from the ground up using modern DevOps principles, showcasing a deep understanding of Infrastructure as Code (IaC), GitOps, and cloud-native security best practices.

The core of this project is an automated system where a `git push` to the application's source code repository triggers a chain of events that results in a new, verified version of the application running securely in a GKE cluster, all without manual intervention.

---
## Architecture

This pipeline utilizes a pull-based GitOps model with a strict separation of concerns between the application code and the infrastructure/deployment configuration.

* **`gke-app-source` Repo:** Contains the application source code and the CI pipeline definition.
* **`gke-app-gitops` Repo:** The single source of truth for the system's desired state, containing all Terraform (IaC) and Kubernetes (manifests) configurations.



**Workflow:**
1.  A developer pushes code to the `main` branch of the `gke-app-source` repository.
2.  A **GitHub Actions CI workflow** is triggered to build and push a new Docker image to **Google Artifact Registry**.
3.  Upon success, the CI workflow automatically updates a Kubernetes manifest in the `gke-app-gitops` repository with the new image tag.
4.  **Argo CD**, running in the GKE cluster, detects the change in the `gke-app-gitops` repository.
5.  Argo CD pulls the new configuration and synchronizes the cluster state, deploying the new version of the application.

---
## Key Principles & Skills Demonstrated

This project is a practical demonstration of several key DevOps and Cloud Engineering competencies:
* **Infrastructure as Code (IaC):** All cloud resources (VPC, Subnets, GKE Cluster) are defined declaratively and managed through Terraform, ensuring a repeatable and version-controlled environment.
* **GitOps:** Git is used as the single source of truth. Argo CD ensures that the state of the Kubernetes cluster always converges to the state defined in the `gke-app-gitops` repository.
* **CI/CD Automation:** End-to-end automation from code commit to production deployment using GitHub Actions.
* **Secure, Passwordless Authentication:** Implementation of **Workload Identity Federation** (OIDC) to grant GitHub Actions secure, short-lived credentials to access Google Cloud, eliminating the need for static service account keys.
* **Principle of Least Privilege (PoLP):** Fine-grained IAM permissions are applied. For instance, the GKE node's service account is granted only the `Artifact Registry Reader` role, the minimum permission required for it to perform its function.
* **Containerization Best Practices:** Use of multi-stage Docker builds to create minimal, secure, and efficient final container images.

---
## Technology Stack

| Category | Technology | Purpose |
| :--- | :--- | :--- |
| **Cloud Provider** | Google Cloud Platform (GCP) | Hosting all infrastructure resources. |
| **IaC** | Terraform | Provisioning and managing the GKE cluster and networking. |
| **CI/CD** | GitHub Actions | Automating the build, push, and deployment trigger workflow. |
| **GitOps** | Argo CD | Continuously deploying the application by syncing with the Git repo. |
| **Orchestration** | Google Kubernetes Engine (GKE) | Running the containerized application at scale. |
| **Containerization** | Docker / Rancher Desktop | Packaging the application and its dependencies. |
| **Registry** | Google Artifact Registry | Storing and managing Docker container images. |
| **Authentication** | GCP Workload Identity Federation | Secure, keyless authentication between GitHub Actions and GCP. |

---
## Project Setup & Deployment

To replicate this environment, follow these high-level steps:

1.  **Prerequisites:**
    * A GCP account with an active billing project and the required APIs enabled.
    * Local tools installed: `gcloud` CLI, `terraform`, `kubectl`, `git`, and a container engine like Rancher Desktop.
    * Two GitHub repositories (`gke-app-source`, `gke-app-gitops`).

2.  **Provision Infrastructure:**
    * From the `gke-app-gitops` repository, configure the Terraform GCS backend.
    * Run `terraform init`, `terraform plan`, and `terraform apply` to create the VPC, subnetwork, and GKE cluster.

3.  **Configure Secure Authentication & Permissions:**
    * Set up Workload Identity Federation in GCP, creating a pool and a provider with a strict `--attribute-condition` for enhanced security.
    * Create IAM Service Accounts for the CI pipeline and grant them appropriate roles.
    * Grant the GKE node's service account the `roles/artifactregistry.reader` role to allow it to pull images.

4.  **Deploy Argo CD:**
    * Install the Argo CD operator into the GKE cluster.
    * Apply the `application.yaml` manifest to configure Argo CD to monitor the `gke-app-gitops` repository's `app-manifests` path.

## How to Trigger the Pipeline

With the infrastructure and Argo CD in place, the end-to-end deployment process is triggered by a simple `git push` to the application repository:

1.  Make a code change in the `gke-app-source` repository.
2.  Commit and push the changes to the `main` branch:
    ```bash
    git commit -m "feat: Update application feature"
    git push origin main
    ```
This will automatically trigger the CI/CD pipeline, resulting in a new version of the application being deployed to the GKE cluster.

---
## Verification & Teardown

* **Verification:** A successful deployment can be confirmed by checking the application status in the Argo CD UI and by accessing the `EXTERNAL-IP` of the `hello-world-service` LoadBalancer.
    ```bash
    kubectl get service hello-world-service
    ```
* **Teardown:** To avoid ongoing costs, all cloud resources can be destroyed by running `terraform destroy` from the `gke-app-gitops` repository.
