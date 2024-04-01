# Deploying Hosted Cluster using ACM

Red Hat OpenShift Container Platform clusters can be deployed using two distinct control plane configurations: standalone or hosted. In the standalone setup, dedicated virtual or physical machines are used to host the control plane. Conversely, with hosted control planes, control planes are created as pods within a hosting cluster, eliminating the need for dedicated virtual or physical machines for each control plane.

In this example we will look at deploying the hosted control plane via Red Hat Advanced Cluster Management (RH ACM will refer to as ACM for this document). In the current release of ACM the hosted control plane can be deployed using the hcp utility, follow along to see how we can eliminate the need to deploy hosted cluster via CLI but also automate the creation of the hosted cluster.
    
# 1. Prerequisites

* Following Red Hat OpenShift Operators installed on the Hub Cluster
  - Red Hat Advanced Cluster Management Operator (ACM)
  - MultiCluster Engine Operator installed as part of Red Hat ACM
  - Openshift Virtualization Operator
* Fork this [hcp github](https://github.com/rohitralhan/hcp-clusters) repo. This will be used in one of the steps below

# 2. Github Structure

This git repo consists of three main folders 
1. apps - As the hosted cluster is treated as an application. This folder contains pointers to the directory containing manifests to deploy the application. Currently it is pointing to cluster directory which contains manifests to deploy cluster. In future you can add pointers to directory containing manifests to deploy operators or policies.
2. bootstrap - Contains the definition files for creating the various resources for creating the cluster like namespace, subscription, channel, rolebindings etc.
3. clusters - creates the definition(s) for creating the cluster(s). If you need to create a new cluster place the definition for that cluster in this folder, as soon as this folder is updated ACM application will start creating the cluster.
4. If you need to create a new cluster copy the cluster definition to the clusters folder as soon as the next sync with github happens the new cluster will be provisioned.

# 3. Automating Deployment of Hosted Control Plane
Procedure 
1. Login to the `Red Hat OpenShift Console` click on `All Clusters` at the top and it will bring you to the `Infrastructure → Clusters` page where all the clusters managed by ACM are listed
2. On this `Infrastructure → Clusters` Screen click on `Cluster Sets → Create Cluster Set` button
3. Provide a name for the Cluster Set and click the Create Button
4. On the next screen click on the `Manage Resource Assignment` and choose the `hub cluster` and click `Review` and then `Save` this will create a cluster set that will be used later in this document
5. Click on `Applications → Create Application → Subscription` this will take you to the `Create Application` screen
6. Provide the `name` and `namespace` for the application
7. In the same screen under the `Repository location for resource → Repository Type` click `Git`
8. On the screen that appears enter the `git repo` details (the git you cloned above) including the `Git URL, username, token etc.`
9. Under the `Select clusters for application deployment` section select the `Cluster Set` create in step 4 and provide the `Label, Operator and Value` as appropriate
10. Click `Create` on the top right of the screen.
11. Once this is created based on the state of the git repo the clusters will be created (if any available in the git repo under the clusters folder.)
12. Go to `Applications → <<name of the application you created>>` on the `Overview` tab you will see teh details about the application its `repo, status, Last Sync time etc.`
13. Click on the `Topology` tab to see the application deployment topology
14. As soon as the above application is created and the sync will github is triggered, the defination for the clusters in the `Clusters folder` are executed and the clusters are created
15. The progress of each of the clusters can be tracked in the Red Hat OpenShift Console UI under `All Clusters`. Additionally you can also navigate to the `hub cluster` and under `Home → Projects` you will see the projects appear with the name of the cluster in the format `cluster-<<cluster name per the defination>>`

     

    The image below shows each of the above mentioned steps:
    
![](https://github.com/rohitralhan/hypershift-hosted-cluster-acm/blob/main/images/ACM/output.gif)

# 4. Scaling Hosted Cluster
In this section we will take a look at the options available for scaling the hosted clusters manually and automatically.

### 4.1 Manual Scaling Procedure
1. Login to the `Red Hat OpenShift Console` click on `All Clusters` at the top and it will bring you to the `Infrastructure → Clusters` page where all the clusters managed by ACM are listed
2. Navigate to the `Cluster` you want to scale
3. On the `Overview` tab under `Control plane status` click `Cluster node pools`
4. Under `Cluster node pools` you will see a table with the summary about the cluster including the `Status, number of Nodes, Autoscaling details etc.`
5. In the summary table on the right side click ... ![]() from the dropdown click `Manage node pools`
6. In the `Manage node pools popup` click `-/+` to select the desired number of nodes
7. Click the `Update` button to initiate the scaling process

The image below shows each of the above mentioned steps:

![](https://github.com/rohitralhan/hypershift-hosted-cluster-acm/blob/main/images/Scaling/manual/output.gif)

### 4.2 Auto Scaling Procedure
1. Login to the `hub cluster` using the oc CLI tool
2. Run the following command to list the available nodepools

```
oc get nodepool -n clusters --kubeconfig kubeconfig
```

![](https://github.com/rohitralhan/hypershift-hosted-cluster-acm/blob/main/images/nodepool.png)
3. From the list of node pools get the nodepool you want to set autoscaling on and run the following command after updating the nodepool name:
```
oc edit nodepool <<nodepool name>> --kubeconfig -n clusters <<kubeconfig if not logged in with user name and password>>
```
4. The above command will bring up the noodpool definition for editing, under the spec section replace the `replicas:` section with the section below setting the min and max to the desired values for your cluster:
```yml
   autoScaling:     
         max: 4
         min: 2
```
![](https://github.com/rohitralhan/hypershift-hosted-cluster-acm/blob/main/images/Scaling/autoScaling.png)

5. Save and exit, this will automatically enable auto scaling and scale the cluster to the desired `min` state and have it ready for the `max` desired state.
6. To validate this login to the `Red Hat OpenShift Console` click on `All Clusters` at the top and it will bring you to the `Infrastructure → Clusters` page where all the clusters managed by ACM are listed.
7. Navigate to the `Cluster` you scaled in step 4 & 5 above
8. Under `Cluster node pools` you will see a table with the summary about the cluster including the `Status, number of Nodes, Autoscaling details etc.`
9. Under the `Autoscaling` column instead of `False` you will see the Min and the Max values you have set in step 4 above
10. Under the `Nodes` tab you will see that the cluster has scaled to the minimum desired number of nodes


