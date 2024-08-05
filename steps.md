# Access Management in EKS Cluster

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

![Screenshot 2024-08-05 125301](https://github.com/user-attachments/assets/3b677461-84bc-4f85-9695-c851c50dce49)


If you don't have these tools installed on your machine, you can follow the below links to install them:

- [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [Install eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

### Steps:

In this demo, I am going to use my IAM user named "**developer-briz**". By default, this user doesn't have any access to EKS cluster. I Just created this user for this demo. Also User "**developer-briz**" have only Elastic Beanstalk Full Access till now. You can see the below screenshots to verify the permissions of this user "**developer-briz**".

![Screenshot 2024-08-05 120759](https://github.com/user-attachments/assets/a08b4534-6d5e-4976-b70c-d9a257b4159e)

![Screenshot 2024-08-05 120823](https://github.com/user-attachments/assets/c22f159c-7c16-4192-b5b5-c34b66cd4bcc)

![Screenshot 2024-08-05 120833](https://github.com/user-attachments/assets/4e5ace5d-37c5-41ba-8c6d-4d006524d7cc)


Now we know, "**developer-briz**" don't have any access to my EKS Cluster. We are going to provide "**developer-briz**" with developer level access to EKS Clsuter. So that he can access the resources which are related to the development environment.<br>

Also you need to create Access Key and Secret Key for this user. You can create it by following this document: [Creating Access Key and Secret Key](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey). Once you create the Access Key and Secret Key, Paste it in some text file. We will use it later.

![Screenshot 2024-08-05 121849](https://github.com/user-attachments/assets/d97986a3-eebc-409f-9e0a-f1a4cf61d70d)


Above screenshot is for reference only. You need to create your own IAM user and Access Key and Secret Key.

Let's start,

### Step 1: Provsisioning EKS Cluster

There are numerous ways to create an EKS cluster. You can create an EKS cluster using `eksctl` command, `AWS Management Console`, `AWS CLI` etc., In this demo, We are going to create an EKS cluster using the `eksctl` command for simplicity.

- Use the below create an EKS cluster using the `eksctl` command:

```bash
eksctl create cluster --name eksdemocluster --region us-east-1 --nodegroup-name eksdemocluster-ng --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 4 --managed
```
You can replace the cluster name, region, node type, and node count as per your requirement.

![Screenshot 2024-08-05 120622](https://github.com/user-attachments/assets/f4e4a577-350d-4227-92c2-98db4cefb927)

![Screenshot 2024-08-05 123953](https://github.com/user-attachments/assets/4ca7b743-ad0f-4116-bb99-15291617de14)


- Once the cluster is created, you can check the status of your cluster using the below command:

```bash
eksctl get cluster
```

![Screenshot 2024-08-05 124637](https://github.com/user-attachments/assets/86f137e3-7f1c-4e85-8e38-75b02c6e5222)


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

![Screenshot 2024-08-05 125840](https://github.com/user-attachments/assets/408ae9ae-3be6-48a6-8ecd-5e031630c0b2)

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

![Screenshot 2024-08-05 125859](https://github.com/user-attachments/assets/0f03880c-0527-4ada-a647-e86efa05712f)

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

![Screenshot 2024-08-05 130707](https://github.com/user-attachments/assets/6002306e-d9ee-44f2-ab9c-1f1bd4760e5b)

You can check the roles and rolebindings by running the below commands:

```bash
kubectl get roles -n development
kubectl get rolebindings -n development
```

Now for testing purpose, I am also going to create one Pod in Default namespace. We will see how the user "developer" can access the resources in the development namespace but not in the default namespace later. You can create the Pod by running the below command:

```bash
kubectl run mypod --image=nginx
```

![Screenshot 2024-08-05 134510](https://github.com/user-attachments/assets/f11fc51c-8222-4ba8-8499-83d2d0a318ad)

We have created the required roles and rolebindings for the developer user. Now, Let's look at how we can add our IAM user to this EKS cluster.


### Step 3: Adding our IAM User as User to EKS Cluster

Before adding the IAM user to the EKS cluster, let's try to access the EKS cluster using the IAM user "**developer-briz**". I am going to switch to the IAM user "**developer-briz**" by running the below command:

```bash
aws configure # Provide the Access Key and Secret Key of the IAM user "developer
```

![Screenshot 2024-08-05 134812](https://github.com/user-attachments/assets/dbb93e3b-7c9b-41ad-ad01-57abf8c8843b)


- You can verify the IAM user by running the below command:

```bash
aws sts get-caller-identity
```

- Now, let's try to list the pods in default namespace:

```bash
kubectl get pods
```

![Screenshot 2024-08-05 134855](https://github.com/user-attachments/assets/ed1fa634-63e7-48f5-8299-6da3d245ddc5)


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

![Screenshot 2024-08-05 135211](https://github.com/user-attachments/assets/9a5f04f1-9f4e-45d9-a9d3-5c0b74a4231d)


- It will open the configmap in the default editor. I am using Windows, So in my case the default editor is Notepad. You need to add the below code in the "mapUsers" section of the configmap. You can see the below code for reference:
```yaml
mapUsers: |
    - userarn: arn:aws:iam::655436663840:user/developer-briz
      username: developer
      groups:
      - developer
```

![Screenshot 2024-08-05 135156](https://github.com/user-attachments/assets/dc5ef191-b28f-4ce6-b761-d8e56a21bf3a)
![Screenshot 2024-08-05 135508](https://github.com/user-attachments/assets/68c10f9b-6553-4032-a068-b6aeafa330c0)

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

![Screenshot 2024-08-05 135624](https://github.com/user-attachments/assets/efe7957e-f704-4419-b30d-be8d242567bc)


You can see in the above screenshot, the developer user can list the pods in the development namespace. That means the "developer-briz" now have access to the EKS cluster. But If we try to list the pods in the default namespace, we will get the error that's also you can see in that Picture.

![Screenshot 2024-08-05 135724](https://github.com/user-attachments/assets/5c1ef4d1-7cee-4212-91c3-5a42ab149718)

![Screenshot 2024-08-05 135929](https://github.com/user-attachments/assets/186a2e15-90a9-46b4-9d9f-7e983d2f4219)


Now we can see that the developer user can access the resources in the development namespace but not in the default namespace. That's how we can provide granular access to the EKS cluster.

### Step 5: Deleting the EKS Cluster

Once you are done with the testing, you can delete the EKS cluster by running the below command:

```bash
eksctl delete cluster --name eksdemocluster
```
If you are using "developer-briz" user, then you need to switch to the user with admin access to delete the EKS cluster. Otherwise, you will get the below error:

![Screenshot 2024-08-05 140123](https://github.com/user-attachments/assets/0ec4d46a-0f3a-4623-a264-acca434cde8f)

Once you switch to user with admin access, you can delete the EKS cluster by running the above command.

![Screenshot 2024-08-05 140238](https://github.com/user-attachments/assets/60a79bc3-735d-4637-bf01-c4dd7068d830)

That's it. We have successfully managed the access to our EKS cluster at two levels: Granular Access and Admin Access. You can follow these same steps to provide access to your IAM users to the EKS cluster.
