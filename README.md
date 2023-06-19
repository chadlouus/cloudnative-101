
<h1 align="center">
Cloud Native Bootcamp
</h1>

<p align="left">
    <a href="https://github.com/ibm-cloud-architecture/learning-cloudnative-101/blob/master/LICENSE">
    <img src="https://img.shields.io/badge/license-Apache--2.0-blue.svg" alt="The Cloud Native Bootcamp is released under the Apache-2.0 license" />
    <a href="https://github.com/ibm-cloud-architecture/learning-cloudnative-101/workflows/SiteDeploy/badge.svg"><img src="https://github.com/ibm-cloud-architecture/learning-cloudnative-101/workflows/SiteDeploy/badge.svg" alt="GithubAction"></a>
  </a>
</p>

## Cloud Native Bootcamp

This course is designed to enable developers with the latest tools and techniques for developing cloud native applications.

The course materials can be viewed at- [https://cloudnative101.dev/](https://cloudnative101.dev/)


```
git clone
```

### Environment Setup
```
# install kubectl CLI
snap install kubectl --classic
# install docker
snap install docker
# install ibmcloud CLI
curl -fsSL https://clis.cloud.ibm.com/install/linux | sh

# create aliases
cat > .bash_profile << EOF
alias ic=ibmcloud
alias k=kubectl
EOF

source .bash_profile

# install Kubernetes plugin to ibmcloud CLI
ic plugin install ks
# install Container Registry plugin to ibmcloud CLI
ic plugin install cr

# login to IBM Cloud
# from https://cloud.ibm.com icon

# configure IKS cluster for kubectl
ibmcloud ks cluster config -c iks-bootcamp
```
### Install dependencies

```
npm install
```

### Local Development

After forking the repository, you can run your changes locally using the following:

```
npm run dev
```

You can access your local changes via [localhost:8000](http://localhost:8000).

### Contributors

- Hemankita Perabathini (hemankita.perabathini@ibm.com)
- Bryan Kribbs (bakribbs@us.ibm.com)
- Carlos Santana (csantana@us.ibm.com)
- Matt Oberlin (mao@us.ibm.com)
- Bharath 
