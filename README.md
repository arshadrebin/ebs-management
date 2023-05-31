### EBS Management - Increasing/ decreasing

Elastic Block Store (EBS) is a block storage service provided by Amazon Web Services (AWS). It is designed to provide persistent block-level storage volumes for use with EC2 instances. EBS is a fundamental component of the AWS infrastructure and is widely used for various purposes, such as hosting databases, storing application data, and supporting other storage-intensive workloads in the cloud.

Let's see some key characteristics and features of EBS:

* **Block-Level Storage**: EBS offers block storage, which means it provides raw storage volumes that function similarly to physical hard drives or SSDs. These volumes can be formatted with a file system and used as persistent storage for EC2 instances.

* **Persistence and Durability**: EBS volumes are designed for durability and data persistence. They provide long-term storage for your data even if the associated EC2 instance is stopped or terminated.

* **Elasticity and Scalability**: With EBS, you can easily scale the size of your storage volumes as your needs change. You can increase or decrease the volume size without impacting the running EC2 instances.

* **High Availability and Durability**: EBS volumes are designed for high availability and durability. They are replicated within a specific Availability Zone (AZ) to protect against failures, ensuring that your data remains accessible and protected.

* **Scalability**: You can easily adjust the size and performance of EBS volumes to meet the changing needs of your applications. You can also create multiple volumes and stripe them together to achieve higher performance and capacity.

### Creating a Volume and attaching to an Instance.

In order to create an EBS volume and attach it to an EC2 instance, you can follow these steps:

1. Open the AWS Management Console and navigate to the EC2 service.
2. Click on "Volumes" in the left-hand menu under "Elastic Block Store."
3. Click on the "Create Volume" button.
4. In the "Create Volume" dialog, specify the desired settings for the volume:

    * Availability Zone: Choose the same Availability Zone as the EC2 instance you want to attach the volume to.

    * Volume Type: Select the appropriate volume type based on your performance and cost requirements.
    
    * Size: Specify the size of the volume in gigabytes (GB).
    * 
<img width="451" alt="Screenshot 2023-05-31 125348" src="https://github.com/arshadrebin/ebs-management/assets/116037443/38aefa0e-d7a9-43f1-abc3-cf41d8925971">







