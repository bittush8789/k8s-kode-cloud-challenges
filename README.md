# Kubernetes Kode Cloud Challenges 🚀

Welcome to the **Kubernetes Kode Cloud Challenges** repository! This project is a curated collection of daily Kubernetes challenges designed to master container orchestration, deployment strategies, and cluster management using **KIND (Kubernetes IN Docker)**.

## 📌 Project Overview

This repository contains step-by-step guides, manifest files, and scenario-based problem statements that simulate real-world DevOps environments. Whether you are a beginner or looking to sharpen your Kubernetes skills, these challenges cover essential topics from basic pod deployment to advanced rolling updates.

## 🛠️ Tech Stack

- **Orchestration:** Kubernetes
- **Local Cluster:** KIND (Kubernetes IN Docker)
- **Container Runtime:** Docker
- **CLI Tool:** `kubectl`
- **OS Platform:** Ubuntu/Linux

## 📅 Challenge Roadmap

| Day | Topic | Description |
| :--- | :--- | :--- |
| **01** | [Deploy Pods](./day1.md) | Initial setup with KIND and basic Pod deployment using NGINX. |
| **02** | [Deployments](./day2.md) | Managing application lifecycle with Kubernetes Deployments. |
| **03** | [Namespaces](./day3.md) | Organizing resources and isolation using Kubernetes Namespaces. |
| **04** | [Resource Limits](./day4.md) | Configuring CPU/Memory requests and limits for Pod stability. |
| **05** | [Rolling Updates](./day5.md) | Implementing zero-downtime deployments and version control. |

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed on your system:
- **Docker**
- **KIND**
- **kubectl**

### Local Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/bittush8789/k8s-kode-cloud-challenges.git
   cd k8s-kode-cloud-challenges
   ```

2. **Create a KIND cluster:**
   ```bash
   kind create cluster --name k8s-challenge-cluster
   ```

3. **Explore the challenges:**
   Navigate through the `dayX.md` files and follow the instructions provided in each.

## 📂 Project Structure

```text
.
├── day1.md          # Pod Deployment
├── day2.md          # Deployments
├── day3.md          # Namespaces
├── day4.md          # Resource Limits
├── day5.md          # Rolling Updates
└── README.md        # Documentation
```

## 🤝 Contributing

Contributions are welcome! If you have more challenges or improvements, feel free to:
1. Fork the project.
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`).
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`).
4. Push to the Branch (`git push origin feature/AmazingFeature`).
5. Open a Pull Request.

## 📄 License

Distributed under the MIT License.

---
*Created by [Bittu Sharma](https://github.com/bittush8789)*
