# Kubernetes Kode Cloud Challenges 🚀

[![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

Welcome to the **Kubernetes Kode Cloud Challenges** repository! This project is a curated collection of daily Kubernetes challenges designed to master container orchestration, deployment strategies, and cluster management using **KIND (Kubernetes IN Docker)**.

---

## 📖 Table of Contents
- [📌 Project Overview](#-project-overview)
- [✨ Key Features](#-key-features)
- [🛠️ Tech Stack](#️-tech-stack)
- [📅 Challenge Roadmap](#-challenge-roadmap)
- [🚀 Getting Started](#-getting-started)
- [📂 Project Structure](#-project-structure)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)

---

## 📌 Project Overview

This repository contains step-by-step guides, manifest files, and scenario-based problem statements that simulate real-world DevOps environments. Whether you are a beginner or looking to sharpen your Kubernetes skills, these challenges cover essential topics from basic pod deployment to advanced troubleshooting.

## ✨ Key Features

- **Hands-on Learning:** Real-world scenarios from KodeCloud and xFusionCorp.
- **Local Development:** Uses KIND for lightweight, local Kubernetes clusters.
- **Comprehensive Topics:** Covers Pods, Deployments, Services, Jobs, and Troubleshooting.
- **Detailed Documentation:** Each day has its own guide with problem statements and solutions.

## 🛠️ Tech Stack

- **Orchestration:** Kubernetes (k8s)
- **Local Cluster:** KIND (Kubernetes IN Docker)
- **Container Runtime:** Docker
- **CLI Tool:** `kubectl`
- **OS Platform:** Ubuntu/Linux/Windows

---

## 📅 Challenge Roadmap

| Day | Topic | Description |
| :--- | :--- | :--- |
| **01** | [Deploy Pods](./day1.md) | Initial setup with KIND and basic Pod deployment using NGINX. |
| **02** | [Deployments](./day2.md) | Managing application lifecycle with Kubernetes Deployments. |
| **03** | [Namespaces](./day3.md) | Organizing resources and isolation using Kubernetes Namespaces. |
| **04** | [Resource Limits](./day4.md) | Configuring CPU/Memory requests and limits for Pod stability. |
| **05** | [Rolling Updates](./day5.md) | Implementing zero-downtime deployments and version control. |
| **06** | [Revert Deployment](./day6.md) | Safely roll back Kubernetes Deployments to previous stable versions. |
| **07** | [ReplicaSets](./day7.md) | Deploy and manage multiple identical Pods for fault tolerance. |
| **08** | [CronJobs](./day8.md) | Automate scheduled tasks using Kubernetes CronJobs. |
| **09** | [Jobs](./day9.md) | Execute one-time batch processing tasks using Kubernetes Jobs. |
| **10** | [Time Check](./day10.md) | Run a container that continuously checks system time. |
| **11** | [Troubleshooting Pods](./day11.md) | Resolve Pod deployment failures inside a cluster. |
| **12** | [Update Deployments](./day12.md) | Update Deployments and Services for new app versions. |
| **13** | [NodePort Service](./day13.md) | Expose applications externally using NodePort Services. |
| **14** | [VolumeMounts Fix](./day14.md) | Resolve VolumeMount issues in Kubernetes Pods. |
| **16** | [Shared Volumes](./day16.md) | Configure shared storage between multiple containers using `emptyDir`. |
| **17** | [Sidecar Containers](./day17.md) | Implement sidecar architecture for centralized logging and monitoring. |
| **18** | [Nginx Web Server](./day18.md) | Deploy and scale an Nginx web server using Deployment and Service. |
| **19** | [Print Environment Variables](./day19.md) | Inspect and print system environment variables inside a Linux system. |
| **20** | [Rolling Updates and Rollbacks](./day20.md) | Implement zero-downtime rolling updates and rollbacks on Deployments. |
| **21** | [Deploy Jenkins](./day21.md) | Deploy Jenkins on a Kubernetes cluster for continuous integration. |
| **22** | [Deploy Grafana](./day22.md) | Deploy Grafana on a Kubernetes cluster for monitoring and visualization. |
| **23** | [Deploy Tomcat App](./day23.md) | Deploy and configure Apache Tomcat Java application inside Kubernetes. |
| **24** | [Deploy Node.js App](./day24.md) | Containerize and deploy Node.js applications with Kubernetes. |
| **25** | [Troubleshoot Deployment](./day25.md) | Analyze, diagnose, and fix common Deployment issues in Kubernetes. |
| **26** | [Fix LAMP Environment](./day26.md) | Fix environmental configuration issues in a deployed LAMP stack. |
| **27** | [Deploy Apache Web Server](./day27.md) | Deploy an Apache Web Server on a Kubernetes cluster with high availability. |
| **28** | [Deploy LAMP Stack](./day28.md) | Deploy a fully functional LAMP (Linux, Apache, MySQL, PHP) Stack. |
| **29** | [Init Containers](./day29.md) | Use Init Containers to pre-configure environments or wait for readiness. |
| **30** | [Persistent Volumes](./day30.md) | Implement Persistent Volumes (PV) and claims (PVC) for storage. |
| **31** | [Manage Secrets](./day31.md) | Securely manage sensitive data like credentials using Kubernetes Secrets. |
| **32** | [Environment Variables](./day32.md) | Inject configuration settings into containers using Env Variables. |
| **33** | [Kubernetes LEMP Setup](./day33.md) | Deploy a full LEMP (Linux, Nginx, MySQL, PHP) Stack. |
| **34** | [Kubernetes Troubleshooting](./day34.md) | Advanced Kubernetes troubleshooting involving Pods and Network. |
| **35** | [Deploy Iron Gallery](./day35.md) | Deploy the Iron Gallery web application on Kubernetes. |
| **36** | [Fix Python App](./day36.md) | Fix configuration errors in a deployed Python application. |
| **37** | [Deploy Redis](./day37.md) | Deploy Redis Cache as a Kubernetes Deployment for session management. |
| **38** | [Deploy MySQL](./day38.md) | Deploy a MySQL relational database for enterprise storage needs. |
| **39** | [Nginx and PHP-FPM](./day39.md) | Setup Nginx front-end integrated with PHP-FPM back-end processing. |
| **40** | [Deploy Drupal App](./day40.md) | Deploy and configure Drupal Content Management System on Kubernetes. |
| **41** | [Deploy Guest Book](./day41.md) | Deploy a multi-tier Guest Book application with Redis backend. |
| **42** | [Fix ImagePullBackOff](./day42.md) | Troubleshoot and fix ImagePullBackOff using private registry credentials. |
| **43** | [Fix CrashLoopBackOff](./day43.md) | Diagnose and fix CrashLoopBackOff across 3 container restart scenarios. |
| **44** | [Scheduling Controls](./day44.md) | Control pod placement using Node Affinity, Taints, and Tolerations. |
| **45** | [StatefulSet PV Fix](./day45.md) | Resolve Persistent Volume attachment issues for StatefulSets. |
| **46** | [Network Policies](./day46.md) | Secure pod-to-pod and pod-to-database traffic using Network Policies. |
| **47** | [HELM Package Manager](./day47.md) | Master HELM, the package manager, to manage application life-cycles. |
| **48** | [Kubectl Port Forward](./day48.md) | Expose and access Kubernetes Pods/Services locally with port forwarding. |

---

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed on your system:
- **Docker Desktop** or **Docker Engine**
- **KIND** (Kubernetes IN Docker)
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

---

## 📂 Project Structure

```text
.
├── day1.md - day48.md   # Daily challenge guides (1-14, 16-48)
├── README.md            # Main documentation
└── .git                 # Git configuration
```

---

## 🤝 Contributing

Contributions are welcome! If you have more challenges or improvements, feel free to:
1. **Fork** the project.
2. **Create** your Feature Branch (`git checkout -b feature/AmazingFeature`).
3. **Commit** your changes (`git commit -m 'Add some AmazingFeature'`).
4. **Push** to the Branch (`git push origin feature/AmazingFeature`).
5. **Open** a Pull Request.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.

---
<p align="center">
  Developed with ❤️ by <a href="https://github.com/bittush8789">Bittu Sharma</a>
</p>
