# Access Management in AWS EKS Cluster

We all know that **How important is security of our Kubernetes cluster. And when it comes to Access Management of EKS Cluster, it becomes more crucial because we must need to provide correct level access to the appropriate users to our EKS cluster**. We should not provide full access to all the users with the cluster. So, We need to manage the access to our EKS cluster properly. In this article, we will discuss how we can give access IAM users to our EKS cluster.<br>

This Repository contains the configuration files and steps to implement the below workflow of Access Management in AWS EKS Cluster.


## Architecture/Workflow Diagram

![Access Management in EKS](https://github.com/user-attachments/assets/3170a6d6-34c3-43d0-b50f-7a4ad5a6b540)


## Prerequisites

- A Linux, Windows or Mac machine with `AWS CLI`, `eksctl`, and `kubectl` installed.
- An AWS account with necessary permissions to create EKS cluster.

## Usage

- Refer the [Steps](./steps.md) to implement the workflow.<br>
- Medium Article: [Access Management in AWS EKS Cluster](https://medium.com/@mathesh-me/access-management-in-aws-eks-cluster-1b3b3b3b3b3b)

## Contributing

Contributions are always welcome!. If you have any other ideas or better approaches, please feel free to open an issue and create a pull request.

## Author

- [Mathesh M](https://www.linkedin.com/in/mathesh-me/)
- Checkout my articles on [Medium](https://medium.com/@mathesh-me)
