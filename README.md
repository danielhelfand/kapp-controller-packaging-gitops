# GitOps with kapp-controller Package Management Resources

This repository contains an example of using kapp-controller's package management 
features with a GitOps workflow.

### Prerequisities

* Kubernetes cluster
* [Install kapp-controller](https://carvel.dev/kapp-controller/docs/latest/install/) on the cluster
* Have [kapp and ytt installed](https://carvel.dev/#whole-suite) locally

### GitOps Workflow

This repository contains two folders: `app` and `packaging`. 

The `app` folder contains an App custom resource that will be used to fetch, create, sync, and update 
the contents of the `packaging` folder in this repository.

The App custom resource is kapp-controller's continuous delivery resource used for keeping a Kubernetes 
environment in sync with a source, which in this case will be a git repository.

The steps needed to carry out this GitOps workflow are as follows:
1. Create the contents of the `app` folder on your cluster
2. Make a change to one of the PackageInstall resources under `packaging` and commit the change to a git repository
3. Observe how the App syncs the changes made in your git repository to the cluster

#### Create the App

In order to make sure you are working with a repository you have push acceess to, it it recommended to fork 
this repository. 

After forking the repository, clone it locally and cd into the directory.

Once you have your local git repo set up, you can deploy the contents of the `app` folder using the command below. 
**NOTE:** Make sure to replace `YOURGITREPO` in the command with the url of your forked repository.

```shell
ytt -f app -v gitrepo=YOURGITREPO | kapp deploy -a pkg-gitops -f- -y
```

After the command succeeds, you will be able to see two PackageInstalls created on your cluster:

```
k get pkgi -n pkg-gitops

NAMESPACE    NAME           PACKAGE NAME                              PACKAGE VERSION   DESCRIPTION           AGE
pkg-gitops   cert-manager   cert-manager.community.tanzu.vmware.com   1.5.4             Reconcile succeeded   2m
pkg-gitops   contour        contour.community.tanzu.vmware.com        1.17.1            Reconcile succeeded   83s
```

#### Make a Change in git

With your environment set up on your Kubernetes cluster, now any changes made to your git repository 
will eventually be synced in by the App you just created. 

To demonstrate this, edit the [`packageinstalls.yml` file](/packaging/packageinstalls.yml). Update the 
contour PackageInstall to use version 1.17.2 by simply editing the line where 1.17.1 is.

Check in your change and push the commit to your forked repository:

After pushing up the change, you can observe the change by waiting for the PackageInstall for contour 
to update its version to 1.17.2 with the kubectl command below. This information will be under the 
PACKAGE VERSION column of the kubectl output.

```shell
kubectl get pkgi/contour -n pkg-gitops -w
```

Once the DESCRIPTION column from kubectl has `Reconcile succeeded`, the change has successfully taken 
place from your git commit.

#### Cleaning up this example

To remove the resources created by this example, run the following command:

```
kapp delete -a pkg-gitops
```
