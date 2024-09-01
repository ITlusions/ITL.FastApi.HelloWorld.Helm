## Helm Chart - Multi-Platform

### Overview

This Helm chart is designed to deploy applications to two different Kubernetes platforms: **OpenShift** and **Mirantis Kubernetes Engine (MKE)**. The chart serves as a "parent" chart that brings together configurations for both platforms, allowing you to deploy to either one or both of them based on your environment requirements.

### Chart Structure

The chart is organized with the following structure:

```markdown
.
├── .helmignore
├── Chart.yaml
├── README.md
├── values.yaml
├── charts
│   ├── mirantis
│   │   └── README.md
│   └── openshift
│       └── README.md
├── templates
│   └── deployment.yaml
└── values
    ├── acc.yaml
    ├── dev.yaml
    ├── prd.yaml
    └── tst.yaml
```

- **`charts/`**: Contains the two child charts for OpenShift and Mirantis. These charts do not have their own deployment templates and rely on the parent chart to handle the deployment configuration.
- **`values.yaml`**: The base values file that defines default settings like image repositories, tags, and replica counts.
- **`values-tst.yaml`** and **`values-prod.yaml`**: Environment-specific values files that determine which platform(s) to deploy to and can override base values such as replicas and images.

### Features

- **Deploy to Multiple Platforms**: The chart supports deploying to OpenShift, Mirantis, or both platforms at the same time.
- **Environment-Specific Configurations**: You can customize deployments based on different environments (e.g., test, production) using separate values files.
- **Flexible Deployment Control**: Easily enable or disable deployment to specific platforms using simple flags in the values files.

### Deployment Scenarios and Values

Here are some example scenarios that show how the values from `values.yaml` and environment-specific files (`values-tst.yaml` and `values-prod.yaml`) are combined to control the deployment:

| **Scenario**          | **Environment** | **Base Values (`values.yaml`)**                                                    | **Environment-Specific Values**                                         | **Effective Configuration**                       |
|-----------------------|----------------|-----------------------------------------------------------------------------------|-------------------------------------------------------------------------|--------------------------------------------------|
| Scenario 1            | Test           | `deploy.openshift: false`<br>`deploy.mirantis: false`<br>`replicaCount: 1`<br>`image: default-app:latest` | `deploy.openshift: true`<br>`deploy.mirantis: false`<br>`replicaCount: 1`<br>`image: test-app:test-latest`  | Deploys to OpenShift with 1 replica, `test-app:test-latest` image |
| Scenario 2            | Production     | `deploy.openshift: false`<br>`deploy.mirantis: false`<br>`replicaCount: 1`<br>`image: default-app:latest` | `deploy.openshift: false`<br>`deploy.mirantis: true`<br>`replicaCount: 3`<br>`image: prod-app:prod-latest`  | Deploys to Mirantis with 3 replicas, `prod-app:prod-latest` image |
| Scenario 3            | Test           | `deploy.openshift: false`<br>`deploy.mirantis: false`<br>`replicaCount: 1`<br>`image: default-app:latest` | `deploy.openshift: true`<br>`deploy.mirantis: true`<br>`replicaCount: 2`<br>`image: test-app:test-latest`   | Deploys to both OpenShift and Mirantis with 2 replicas, `test-app:test-latest` image |
| Scenario 4            | Production     | `deploy.openshift: false`<br>`deploy.mirantis: false`<br>`replicaCount: 1`<br>`image: default-app:latest` | `deploy.openshift: true`<br>`deploy.mirantis: true`<br>`replicaCount: 5`<br>`image: prod-app:prod-stable`   | Deploys to both OpenShift and Mirantis with 5 replicas, `prod-app:prod-stable` image |

### How It Works

1. **Parent Chart Configuration**: The parent chart (`my-parent-chart`) contains a `deployment.yaml` template that defines how the application should be deployed. This template checks the values provided in the environment-specific files to determine where to deploy.

2. **Child Charts**: The child charts (`openshift` and `mirantis`) are dependencies of the parent chart and are included as part of the deployment. They inherit the deployment configuration from the parent chart.

3. **Environment-Specific Files**: 
   - **`values-tst.yaml`**: Configures the deployment for the test environment. You can specify if you want to deploy to OpenShift, Mirantis, or both by setting the `deploy.openshift` and `deploy.mirantis` flags to `true` or `false`.
   - **`values-prod.yaml`**: Similar to the test file, but with settings suitable for the production environment, like a higher replica count or different image tags.

### How to Deploy

You can use Helm to deploy the chart to your Kubernetes clusters. Here are the steps:

1. **Deploy to the Test Environment:**

   Run the following command to deploy the application to the test environment. This will use both the base values and the test-specific values:

   ```bash
   helm install my-release my-parent-chart -f values.yaml -f values-tst.yaml
   ```

   This will deploy the application according to the settings in `values-tst.yaml` — for example, deploying to both OpenShift and Mirantis if both flags are set to `true`.

2. **Deploy to the Production Environment:**

   To deploy to production, use the production-specific values:

   ```bash
   helm install my-release my-parent-chart -f values.yaml -f values-prod.yaml
   ```

   This will deploy the application with production settings, such as a higher number of replicas or different container images.

### Customizing Your Deployment

- **Enable or Disable Deployment for Specific Platforms**: 
  Modify the `deploy.openshift` and `deploy.mirantis` flags in your environment-specific values files (`values-tst.yaml` or `values-prod.yaml`) to control where your application is deployed.
  
- **Adjust Replica Counts or Images**: 
  Update the `global.replicaCount` and `global.image` fields in the values files to set the number of replicas or specify different container images for different environments.

### Conclusion

This Helm chart simplifies the process of deploying applications to both OpenShift and Mirantis Kubernetes Engine. By using a single parent chart and environment-specific values files, you can easily manage and customize deployments across multiple platforms.

---

This table now clearly shows how the values from `values.yaml` (base) and environment-specific files (like `values-tst.yaml` and `values-prod.yaml`) are combined to determine the effective configuration for each deployment scenario.