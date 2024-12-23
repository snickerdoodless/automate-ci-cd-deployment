# Overview
---

<img src="./assets/project-1-assets(1).svg" alt="Description of the image" style="width: 100%; max-width: 800px;" />



<br />

I aim to reach this following goals for this project:

1.  **Continuous learning** through hands-on implementation.
2. **Building my DevOps portfolio** with real-world examples.
3. **Creating basic CI/CD pipelines** to gain practical experience.

<br />

# Tech Stack
---

| ![Image 1](assets/project-1-assets(1).png) | ![Image 2](assets/project-1-assets(2).png) | ![Image 3](assets/project-1-assets(3).png) | ![Image 4](assets/project-1-assets(4).png) | ![Image 5](assets/project-1-assets(5).png) | ![](./assets/project-1-assets(6).png) |
| ------------------------------------------ | ------------------------------------------ | ------------------------------------------ | ------------------------------------------ | ------------------------------------------ | ------------------------------------- |

<br />

🔹 **Jenkins:** CI/CD Pipelines <br />
🔹 **Docker:** Container & Registry <br />
🔹 **Slack:** Notification Channel <br />
🔹 **AWS:** EC2, Cloudwatch <br />
🔹 **GitHub:** SCM Repository <br />
🔹 **Flask:** App Backend Services <br />

<br />

# Flow
---

➥ **Multi-Instance Architecture:** I'll be utilizing three EC2 instances to create a dedicated environment for development, Jenkins (Master & Slave), and a separate Flask application server.  <br />

➥ **Automated Source Code Fetch:** The pipeline will automatically fetch the latest source code from the version control system upon triggering through a webhook. <br />

 ➥ **Versioned Image Builds:** I'll build Docker images for both the Flaskapp application and the database, ensuring version control and easy deployment. These images will be uploaded to a container registry for storage and retrieval. <br />

➥ **Streamlined Deployment:** The pipeline will seamlessly pull the required images from the registry and deploy them onto the development server, streamlining the deployment process. <br />

 ➥ **Developer Notifications:** Once the deployment is complete, the pipeline will notify developers to review the updated application on the development server. <br />

➥ **Proactive Monitoring:** To ensure application health and optimize resource utilization, I'll set up CloudWatch notifications for health checks, billing alerts, and resource usage monitoring. <br />

<br />

# Spesifications
---

#### 3 EC2 Instances

**Jenkins Master & Node**

> OS: Ubuntu 22.04,
> 1 CPU,
> Ram: 1GB,
> Rom: 8GB

<br />

**Deployment Server**

> OS: Ubuntu 24.04,
> 1 CPU,
> Ram: 1GB,
> Rom: 8GB

<br />

> [!note] 
> *Free Tier Eligible*


<br />

# Demonstration
---

<img src="./assets/demo.gif" alt="Description of the image" style="width: 100%; max-width: 800px;" />


<br />

# Feedback
---

**I’ve been teaching myself DevOps for over a semester, and now I’ve made my first CI/CD project. I know it’s probably not perfect—I might’ve messed up the flow or missed better ways to do things. If you’ve got any feedback, [drop it here](https://docs.google.com/forms/d/e/1FAIpQLSdHPjiyGjCactEcK1N34bO1gZvzv2AllK8AgA_AFSKMxHfOLg/viewform?usp=sharing) . I’d really appreciate it!**
