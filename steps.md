# Access Management of EKS Cluster

We all know that **How important is security of our Kubernetes cluster. And when it comes to Access Management of EKS Cluster, it becomes more crucial because we must need to provide correct level access to the appropriate users to our EKS cluster**. We should not provide full access to all the users with the cluster. So, We need to manage the access to our EKS cluster properly. In this article, we will discuss how we can give access IAM users to our EKS cluster.

We are going to manage the access to our EKS cluster at two levels:

1. **Granular Access:** In this level, we will provide access to our IAM users based on their roles. Example: Developer, Viewer, etc. In this case Developer can only access the resources which are related to the development environment.

2. **Admin Access:** In this level, we will provide full access to IAM users.

### Prerequisites:

- A Linux, Windows or Mac machine with `AWS CLI`, `eksctl`, and `kubectl` installed. You can check the installation of these tools by running the below commands:

```bash
aws --version
eksctl version
kubectl version
```

![Screenshot 2024-08-05 125301](https://github.com/user-attachments/assets/81a51926-34b3-414e-ba3d-4b9799a12693)

If you don't have these tools installed on your machine, you can follow the below links to install them:

- [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [Install eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

### Steps:

In this demo, I am going to use my IAM user named "**developer-briz**". By default, this user doesn't have any access to EKS cluster. I Just created this user for this demo. Also User "**developer-briz**" have only Elastic Beanstalk Full Access till now. You can see the below screenshots to verify the permissions of this user "**developer-briz**".

![Screenshot 2024-08-05 120759](https://github.com/user-attachments/assets/a081a358-f7eb-4380-95ca-86b97dd4560b)

![Screenshot 2024-08-05 120823](https://github.com/user-attachments/assets/b71334cb-b3d3-4246-87e3-4b082136e6d4)

![Screenshot 2024-08-05 120833](https://github.com/user-attachments/assets/cd555c22-fded-4aba-a770-07ce76657f54)

Now we know, "**developer-briz**" don't have any access to my EKS Cluster. We are going to provide "**developer-briz**" with developer level access to EKS Clsuter. So that he can access the resources which are related to the development environment.<br>

Also you need to create Access Key and Secret Key for this user. You can create it by following this document: [Creating Access Key and Secret Key](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey). Once you create the Access Key and Secret Key, Paste it in some text file. We will use it later.
![Screenshot 2024-08-05 121849](https://github.com/user-attachments/assets/4a8939b2-0aef-451b-b165-f92ab04e9080)

Above screenshot is for reference only. You need to create your own IAM user and Access Key and Secret Key.

Let's start,

### Step 1: Provsisioning EKS Cluster

There are numerous ways to create an EKS cluster. You can create an EKS cluster using `eksctl` command, `AWS Management Console`, `AWS CLI` etc., In this demo, We are going to create an EKS cluster using the `eksctl` command for simplicity.

- Use the below create an EKS cluster using the `eksctl` command:

```bash
eksctl create cluster --name eksdemocluster --region us-east-1 --nodegroup-name eksdemocluster-ng --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 4 --managed
```
You can replace the cluster name, region, node type, and node count as per your requirement.

![Screenshot 2024-08-05 120622](https://github.com/user-attachments/assets/2b29badb-8ed1-425e-a7b4-cb4343ee45f4)
![Screenshot 2024-08-05 123953](https://github.com/user-attachments/assets/93e25eb9-9a6f-48f4-a69d-630e012f4796)

- Once the cluster is created, you can check the status of your cluster using the below command:

```bash
eksctl get cluster
```
![Screenshot 2024-08-05 124637](https://github.com/user-attachments/assets/7cec8467-3785-403d-ac68-84b232c1e062)


### Step 2: Creating Roles and RoleBindings for Developer Based Access

**To provide users with granular access to EKS cluster, we need to create roles and rolebindings for the EKS cluster.** In our case, we are going to create a role named "developer" and provide access to the resources like pods, services, deployments, replicasets in the development namespace.

**Create RBAC Roles and RoleBindings for EKS Cluster**

- Let's create a `role` for the developer

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "services", "deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
```
![Screenshot 2024-08-05 125840](https://github.com/user-attachments/assets/92fdc2af-1b4d-4551-ad93-d4e8e044bf25)

In the above YAML file, you can see that I have created a role named "developer" and provided access to the resources like pods, services, deployments, replicasets in the development namespace.

- Now, Let's create the `rolebinding` to bind the role to the user:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: development
subjects:
- kind: User
  name: developer # Name of the user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer # Name of the role
  apiGroup: rbac.authorization.k8s.io
```

![Screenshot 2024-08-05 125859](https://github.com/user-attachments/assets/8173e361-90bd-4423-a65f-b839d699cc48)

I have created a rolebinding named "developer-binding" and bind the developer role to the user named "developer". Don't worry about the user for now. We will add the user to the EKS cluster in the next step.

- If you note namespace on the above two files. I defined the namespace as `development`. So If this role is applied to any user, then that user can only access the resources in the development namespace. Now we don't have this namespace in our EKS cluster. So, Let's create the namespace by running the below command:

```bash
kubectl create namespace development
```

- Let's apply the roles and rolebindings to the EKS cluster:

```bash
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
```
![Screenshot 2024-08-05 130707](https://github.com/user-attachments/assets/8ebd7a53-ae94-4c2b-8520-6d35eb04005a)

You can check the roles and rolebindings by running the below commands:

```bash
kubectl get roles -n development
kubectl get rolebindings -n development
```

Now for testing purpose, I am also going to create one Pod in Default namespace. We will see how the user "developer" can access the resources in the development namespace but not in the default namespace later. You can create the Pod by running the below command:

```bash
kubectl run mypod --image=nginx
```
![Screenshot 2024-08-05 134510](https://github.com/user-attachments/assets/f851e006-e9d7-4254-88e2-ccfa4e7389e3)

We have created the required roles and rolebindings for the developer user. Now, Let's look at how we can add our IAM user to this EKS cluster.


### Step 3: Adding our IAM User as User to EKS Cluster

Before adding the IAM user to the EKS cluster, let's try to access the EKS cluster using the IAM user "**developer-briz**". I am going to switch to the IAM user "**developer-briz**" by running the below command:

```bash
aws configure # Provide the Access Key and Secret Key of the IAM user "developer
```

![Screenshot 2024-08-05 134812](https://github.com/user-attachments/assets/1a2c0f72-b72e-4c74-a7bf-6f87479a320e)

- You can verify the IAM user by running the below command:

```bash
aws sts get-caller-identity
```

- Now, let's try to list the pods in default namespace:

```bash
kubectl get pods
```
![Screenshot 2024-08-05 134855](https://github.com/user-attachments/assets/68b8db59-2b6e-4c00-a2d0-e078d5d2829d)

You can see that the user "developer-briz" doesn't have access to the EKS cluster that's why it's showing the error. Now, we need to give access this user to the EKS cluster. So, he can access the resources in the development namespace.<br>

Now, We need to switch to our user which has admin access to the EKS cluster. Because only the user with admin access can add the users to the EKS cluster. In my case, I have used "Mathesh-aws" user to spin up the EKS cluster. So, I am going to switch to this user by running the below command:

```bash
aws configure
```

Once you switch to the user with admin access, you can add the user to the EKS cluster. Let's see how we can do that:

1. **For Granting Granular Access to EKS Cluster**

- To add IAM User to the EKS cluster, we need to edit the configmap named "aws-auth" in the kube-system namespace. Let's edit the configmap by running the below command:

```bash
kubectl edit -n kube-system configmap/aws-auth
```
![Screenshot 2024-08-05 135156](https://github.com/user-attachments/assets/ae00a5dd-3db8-4d34-8e39-e69a4fc1077c)


- It will open the configmap in the default editor. I am using Windows, So in my case the default editor is Notepad. You need to add the below code in the "mapUsers" section of the configmap. You can see the below code for reference:

![Screenshot 2024-08-05 135508](https://github.com/user-attachments/assets/3607d275-670f-4979-a273-6a2d803e06b6)

```yaml
mapUsers: |
    - userarn: arn:aws:iam::655436663840:user/developer-briz
      username: developer
      groups:
      - developer
```

2. **For Granting Admin Access to EKS Cluster**

- For Providing Admin Access to EKS cluster, we don't to create any additional roles and rolebindings kind of things. We can easily provide the admin access to user by adding the user to the system:masters group. Let's see how we can do that:

```yaml
- userarn: arn:aws:iam::1234567890:user/developer-briz
  username: admin-username # Username of the user
  groups:
    - system:masters # Instead of role, we are using system:masters group
```

### Step 4: Testing the Access of our IAM User

- Now, we have create everything like roles, rolebindings, and also added the user to our EKS cluster. Let's test the access. Now I am going to switch to the IAM user "developer-briz" by running the below command:

```bash
aws configure
```

- Let's try whether the "developer-briz" user can list the pods in the default namespace:

```bash
kubectl get pods -n development
```
![Screenshot 2024-08-05 135624](https://github.com/user-attachments/assets/25eec15a-f1b5-430f-a9dc-203ac272ccba)

You can see in the above screenshot, the developer user can list the pods in the development namespace. That means the "developer-briz" now have access to the EKS cluster. But If we try to list the pods in the default namespace, we will get the error

![Screenshot 2024-08-05 135724](https://github.com/user-attachments/assets/d03b90c5-61ec-47a4-8be1-9cd248eca673)

![Screenshot 2024-08-05 135929](https://github.com/user-attachments/assets/5daead34-4e75-42f2-a16b-9caa5e30825c)

Now we can see that the developer user can access the resources in the development namespace but not in the default namespace. That's how we can provide granular access to the EKS cluster.

### Step 5: Deleting the EKS Cluster

Once you are done with the testing, you can delete the EKS cluster by running the below command:

```bash
eksctl delete cluster --name eksdemocluster
```
If you are using "developer-briz" user, then you need to switch to the user with admin access to delete the EKS cluster. Otherwise, you will get the below error:
![Screenshot 2024-08-05 140123](https://github.com/user-attachments/assets/13e78022-69a5-492d-8ed6-5c7da612da49)

Once you switch to user with admin access, you can delete the EKS cluster by running the above command.
![Screenshot 2024-08-05 140238](https://github.com/user-attachments/assets/7f822193-ead2-4687-9397-9924d53a5ea0).

That's it. We have successfully managed the access to our EKS cluster at two levels: Granular Access and Admin Access. You can follow these same steps to provide access to your IAM users to the EKS cluster.