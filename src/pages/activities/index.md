---
title: Course Activities
description: Course Activities
---

## Main Activities

<Accordion>
  <AccordionItem title="Containers">

  | Task                            | Description         | Link        |
  | --------------------------------| ------------------  |:----------- |
  | *** Walkthroughs ***                         |         |         |     |
  | What is a Container? | A look under the the covers at what is a Linux Container? | <a href="https://www.katacoda.com/courses/container-runtimes/what-is-a-container" target="_blank">Understand Containers</a> |
  | What is an Image? | A look under the the covers at what is a Linux Container Image? | <a href="https://www.katacoda.com/courses/container-runtimes/what-is-a-container-image" target="_blank">Container Images</a> |
  | Docker Basics | Set of walkthroughs that cover docker basics | <a href="https://www.katacoda.com/courses/docker" target="_blank">Docker Basics</a> |
  | *** Try It Yourself ***                         |         |         |
  | IBM Container Registry | Build and Deploy Run using IBM Container Registry  | [IBM Container Registry](../lectures/containers/activities/ibmcloud-cr) |
  | Docker Lab | Running a Sample Application on Docker | [Docker Lab](../lectures/containers/activities) |

  </AccordionItem>

  <AccordionItem title="Kubernetes">

  | Task                            | Description         | Link        |
  | --------------------------------| ------------------  |:----------- |
  | *** Try It Yourself ***                         |         |         |
  | Pod Creation | Challenge yourself to create a Pod YAML file to meet certain parameters. | [Pod Creation](../lectures/kube-overview/activities/labs/lab1) |
  | Pod Configuration | Configure a pod to meet compute resource requirements. | [Pod Configuration](../lectures/kube-overview/activities/labs/lab2) |
  | Multiple Containers | Build a container using legacy container image.| [Multiple Containers](../lectures/kube-overview/activities/labs/lab3) |
  | Probes | Create some Health & Startup Probes to find what's causing an issue.  | [Probes](../lectures/kube-overview/activities/labs/lab4) |
  | Rolling Updates Lab | Create a Rolling Update for your application.  | [Rolling Updates](../lectures/kube-overview/activities/labs/lab6) |
  | Cron Jobs Lab | Using Tekton to test new versions of applications. | [Crons Jobs](../lectures/kube-overview/activities/labs/lab7) |
  | Creating Services | Create two services with certain requirements. | [Setting up Services](../lectures/kube-overview/activities/labs/lab8) |
  | Setting up Persistent Volumes | Create a Persistent Volume that's accessible from a SQL Pod. | [Setting up Persistent Volumes](../lectures/kube-overview/activities/labs/lab10) |
  | Debugging | Find which service is breaking in your cluster and find out why.  | [Debugging](../lectures/kube-overview/activities/labs/lab5) |
  | IKS Ingress Controller | Configure Ingress on Free IKS Cluster | [Setting IKS Ingress](../lectures/kube-overview/activities/labs/ingress-iks) |
  | *** Solutions ***                         |         |         |
  | Lab Solutions | Solutions for the Kubernetes Labs  | [Solutions](../lectures/kube-overview/activities/labs/solutions) |

  </AccordionItem>

  <AccordionItem title="Continuous Integration">

  | Task                            | Description         | Link        |
  | --------------------------------| ------------------  |:----------- |
  | *** Walkthroughs ***                         |         |         |
  | Deploying Applications From Source |  Using OpenShift 4 | [S2I](https://learn.openshift.com/introduction/deploying-python/) |
  | *** Try It Yourself ***                         |         |         |
  | Tekton Lab | Using Tekton to test new versions of applications. | [Tekton](../lectures/continuous-integration/activities/tekton/openshift/) |
  | IBM Cloud DevOps | Using IBM Cloud ToolChain with Tekton | [Tekton on IBM Cloud](../lectures/continuous-integration/activities/ibm-toolchain/) |
  | Jenkins Lab | Using Jenkins to test new versions of applications. | [Jenkins](../lectures/continuous-integration/activities/jenkins/openshift/) |

  </AccordionItem>

  <AccordionItem title="Continuous Deployment">

  | Task                            | Description         | Link        |
  | --------------------------------| ------------------  |:----------- |
  | *** Walkthroughs ***                         |         |         |     |
  | GitOps | Introduction to GitOps with OpenShift | [Learn OpenShift](https://learn.openshift.com/introduction/gitops-introduction/) |
  | GitOps Multi-cluster | Multi-cluster GitOps with OpenShift | [Learn OpenShift](https://learn.openshift.com/introduction/gitops-multicluster/) |
  | *** Try It Yourself ***                         |         |         |
  | ArgoCD Lab | Learn how to setup ArgoCD and Deploy Application | [ArgoCD](../lectures/continuous-deployment/activities/openshift) |

  </AccordionItem>

</Accordion>

## Elective Activities

<Accordion>
  <AccordionItem title="Cloud Foundry Migration">

  | Task                            | Description         | Link        |
  | --------------------------------| ------------------  |:----------- |
  | *** Try It Yourself ***                         |         |         |
  | Migrating to IKS | Migrating from Cloud Foundry to IKS | [IKS](../electives/cf-to-k8s/iks-migration/) |
  | Migrating to OpenShift | Migrating from Cloud Foundry to OpenShift | [OCP](../electives/cf-to-k8s/oc-migration/) |

  </AccordionItem>
  <AccordionItem title="API Connect">

  | Task                            | Description         | Link        |
  | --------------------------------| ------------------  |:----------- |
    | *** Try It Yourself ***                         |         |         |
  | Accessing API Connect | Creating and accessing an API Connect Instance. | [Access API Connect](../electives/api-connect/activities/accessAPI) |
  | Creating APIs | Creating REST APIs using API Connect. | [Creating APIs](../electives/api-connect/activities/creatingAPIs) |
  | Importing an API | Import an existing OpenAPI 2.0 definition. | [Importing APIs](../electives/api-connect/activities/importingAPIs) |
    
  </AccordionItem>
  <AccordionItem title="Cloud Data Services">

  | Task                            | Description         | Link        |
  | --------------------------------| ------------------  |:----------- |
  | Cloud Data | Getting Started with Cloud Data Services (postgres) | [Cloud Data Lab](../electives/data-services/activities/labs/lab1/) |
  
  </AccordionItem>
  <AccordionItem title="Event-Driven Architecture">

  | Task                            | Description         | Link        |
  | --------------------------------| ------------------  |:----------- |
  | *** Walkthroughs ***                         |         |         |     |
  | Console Samples on Kubernetes | Getting Started with Producing & Consuming on Kubernetes | [EDA Lab 0](../electives/eda/activities/labs/lab0/) [Solution](../electives/eda/activities/labs/lab0/solution) |
  | Console Samples on Docker | Getting Started with Producing & Consuming on Docker | [EDA Lab 1](../electives/eda/activities/labs/lab1/) [Solution](../electives/eda/activities/labs/lab1/solution) |
  | Spring for Apache Kafka | Getting Started with the Spring for Apache Kafka project | [EDA Lab 2](../electives/eda/activities/labs/lab2/)[Solution](../electives/eda/activities/labs/lab2/solution) |
  | Reactive Messaging | Getting Started with Reactive Messaging, MicroProfile, and Quarkus | [EDA Lab 3](../electives/eda/activities/labs/lab3/)[Solution](../electives/eda/activities/labs/lab3/solution) |
  | *** Try It Yourself ***                         |         |         |
  | Choose Your Own Adventure! | Utilizing what you have learned in the bootcamp, take a look at some real-world event-driven scenarios and implementations. | [EDA Lab 4](../electives/eda/activities/labs/lab4/)  |
  
  </AccordionItem>
  <AccordionItem title="Monitoring">

  | Activity                        | Link       |
  | --------------------------------|----------- |
  | Install Sysdig Agent on IBM Kubernetes Service (IKS)|  [Sysdig with IKS](../electives/monitoring/sysdig/activities/iks/) |
  | Analyze metrics on minikube using Sysdig |  [Sysdig with minikube](../electives/monitoring/sysdig/activities/minikube/) |
  | Using Sysdig Dashboards |  [Sysdig Dashboards](../electives/monitoring/sysdig/activities/dashboards/) |
  | Sysdig Alerts  |  [Sysdig Alerts](../electives/monitoring/sysdig/activities/alerts/) |
  | Prometheus Java Metrics with Sysdig  |  [Sysdig Java Prometheus](../electives/monitoring/sysdig/activities/prometheus/java) |
  | Prometheus Node.js Metrics with Sysdig  |  [Sysdig Node.js Prometheus](../electives/monitoring/sysdig/activities/prometheus/nodejs) |
  
  </AccordionItem>
  <AccordionItem title="Logging">

  | Activity                        | Link       |
  | --------------------------------|----------- |
  | Install LogDNA agent on IBM Kubernetes Service (IKS)|  [LogDNA with IKS](../electives/logging/logdna/activities/iks/) |
  | Install LogDNA agent on RedHat OpenShift|  [LogDNA with OpenShift](https://cloud.ibm.com/docs/Log-Analysis-with-LogDNA?topic=Log-Analysis-with-LogDNA-config_agent_os_cluster) |
  | Install LogDNA agent on minikube |  [LogDNA with minikube](../electives/logging/logdna/activities/minikube/) |
  | Using LogDNA Dashboard |  [LogDNA Dashboards](../electives/logging/logdna/activities/dashboards/) |
  | Alers with LogDNA |  [Alers with LogDNA](../electives/logging/logdna/activities/alerts/) |
  | JSON logs with LogDNA |  [JSON logs with LogDNA ](../electives/logging/logdna/activities/logger/nodejs) |

  </AccordionItem>
   <AccordionItem title="Distributed Tracing">

  | Activity                        | Link       |
  | --------------------------------|----------- |
  | NodeJS | [NodeJS](../electives/dist-trace/activities/lab1) |
  | Java | [Java](../electives/dist-trace/activities/lab2) |
  | Open Liberty | [Open Liberty](../electives/dist-trace/activities/lab3) |
  | Jaeger in OpenShift & Kubernetes | [Istio Kubernetes](../electives/dist-trace/activities/lab4) |
  | Jaeger with Istio - NodeJS | [Istio NodeJS](../electives/dist-trace/activities/lab5) |
  | Jaeger with Istio - Java | [Istio Java](../electives/dist-trace/activities/lab6) |
  | Jaeger with Istio - Open Liberty | [Istio Open Liberty](../electives/dist-trace/activities/lab7) |

  </AccordionItem>
</Accordion>

## Projects

<Accordion>
  <AccordionItem title="BootCamp Projects">

| Task                            | Description         | Link        |
| --------------------------------| ------------------  |:----------- |
| *** Try It Yourself ***                         |         |         |
| OpenShift Project | Building a Devops Pipeline with Openshift and Tekton | [CICD Project](../projects/project-cicd/) |
| Microservices Project | Cloud Native Starter | [MicroProfile Project](../projects/project-cn-starter/) |

  </AccordionItem>
  
</Accordion>
