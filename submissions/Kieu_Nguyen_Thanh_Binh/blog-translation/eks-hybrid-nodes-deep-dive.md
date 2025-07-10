# Tìm hiểu sâu về Amazon EKS Hybrid Nodes

> **📖 Bài viết gốc**: [A deep dive into Amazon EKS Hybrid Nodes](https://aws.amazon.com/blogs/containers/a-deep-dive-into-amazon-eks-hybrid-nodes/)  
> **👤 Tác giả**: Chris Splinter, Principal Product Manager, AWS Kubernetes; Elamaran Shanmugam, Sr. Container Specialist Solutions Architect, AWS; Re Alvarez Parmar, Containers Specialist Solutions Architect, AWS  
> **📅 Ngày xuất bản**: 27 Tháng 1, 2025  
> **🌐 Nguồn**: AWS Containers Blog  
> **👨‍💻 Người dịch**: Kiều Nguyễn Thanh Bình - FCJ Intern  
> **📅 Ngày dịch**: 10 Tháng 7, 2025  

---

## 📋 Tóm tắt

Amazon EKS Hybrid Nodes là tính năng mới cho phép sử dụng hạ tầng on-premises và edge hiện có làm node trong Amazon EKS cluster, tạo ra trải nghiệm quản lý Kubernetes thống nhất trên cloud, on-premises và edge environment. Tính năng này giúp giải quyết các thách thức về modernization, machine learning, media streaming và manufacturing workload bằng cách loại bỏ độ phức tạp của việc tự quản lý Kubernetes on-premises và cung cấp trải nghiệm vận hành nhất quán với EKS cluster, feature, integration và tool quen thuộc.

**🎯 Đối tượng đọc**: Platform Engineers, Kubernetes Administrators, Hybrid Cloud Architects  
**📊 Độ khó**: Advanced  
**🏷️ Tags**: Amazon EKS, Hybrid Nodes, On-premises, Edge Computing, Kubernetes

---

*Blog này được tác giả bởi Chris Splinter, Principal Product Manager, AWS Kubernetes; Elamaran Shanmugam, Sr. Container Specialist Solutions Architect, AWS; Re Alvarez Parmar, Containers Specialist Solutions Architect, AWS.*

Chúng tôi rất vui mừng thông báo về tính khả dụng chung của tính năng mới cho [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/) mà chúng tôi đã ra mắt tại re:Invent 2024 có tên là Amazon EKS Hybrid Nodes. Với EKS Hybrid Nodes, người dùng có thể sử dụng hạ tầng on-premises và edge hiện có của họ làm node trong Amazon EKS cluster, tạo ra trải nghiệm quản lý Kubernetes thống nhất trên cloud, on-premises, và edge environment. Điều này có thể được sử dụng cho nhiều use case khác nhau, như modernization, machine learning (ML), media streaming, và manufacturing workload.

Người dùng đang sử dụng Kubernetes trong AWS Cloud cho các ứng dụng mới và hiện đại hóa thường muốn mở rộng các khả năng này để quản lý ứng dụng chạy trong on-premises và edge environment vì lý do low latency, data dependency, data sovereignty, regulatory, hoặc policy. Trong lịch sử, người dùng muốn chạy Kubernetes trong on-premises data center hoặc edge environment buộc phải chạy và vận hành open source Kubernetes hoặc các giải pháp Kubernetes self-managed tương tự. Việc tự quản lý Kubernetes on-premises rất phức tạp và thêm operational overhead, cuối cùng làm chậm kế hoạch innovation và modernization.

EKS Hybrid Nodes loại bỏ độ phức tạp và overhead đó bằng cách cho phép người dùng kết nối capacity on-premises và edge hiện có của họ làm node với managed Amazon EKS control plane trong cloud. Điều này tối ưu hóa việc chạy Kubernetes on-premises và cho phép trải nghiệm vận hành on-premises nhất quán sử dụng cùng EKS cluster, feature, integration, và tool mà người dùng đã quen thuộc để chạy workload trong cloud.

## Tổng quan

Để sử dụng EKS Hybrid Nodes, bạn cần connectivity giữa on-premises network của bạn và [Amazon Virtual Private Cloud (Amazon VPC)](https://aws.amazon.com/vpc/) mà bạn đang sử dụng cho EKS cluster của bạn. Bạn có thể sử dụng [AWS Direct Connect](https://aws.amazon.com/directconnect/), [AWS Site-to-Site VPN](https://aws.amazon.com/vpn/site-to-site-vpn/), hoặc giải pháp VPN riêng của bạn để tạo private connection giữa EKS cluster và hybrid node của bạn. EKS Hybrid Nodes tái sử dụng [cơ chế hiện có](https://aws.amazon.com/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/) trong Amazon EKS cho communication từ control plane đến worker node. Do đó, bạn có thể có node chạy trên [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) instance trong [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) và hybrid node chạy trong on-premises environment của bạn trong cùng EKS cluster.

EKS Hybrid Nodes sử dụng cách tiếp cận "bring your own infrastructure" nơi bạn chịu trách nhiệm provision và quản lý hạ tầng và operating system mà bạn sử dụng cho hybrid node. Bạn có thể sử dụng bare metal server hoặc virtualized infrastructure hiện có của bạn làm compute cho hybrid node, và hiện tại Amazon Linux 2023, Ubuntu, và Red Hat Enterprise Linux (RHEL) là các operating system được AWS hỗ trợ để tương thích với hybrid node.

EKS Hybrid Nodes có thể được cài đặt và kết nối với EKS cluster của bạn bằng EKS Hybrid Nodes CLI (nodeadm), mà bạn chạy trên mỗi on-premises host. Thay vào đó, bạn có thể bao gồm nodeadm và hybrid node dependency trong golden operating system image của bạn để tự động hóa hybrid node bootstrap, tương tự như cơ chế được sử dụng cho Amazon EKS-optimized [Amazon Machine Images (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) cho EC2 instance trong cloud. Khi hybrid node được kết nối với EKS cluster của bạn, chúng sử dụng temporary [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) credential được provision bởi [AWS Systems Manager](https://aws.amazon.com/systems-manager/) hybrid activation hoặc IAM Roles Anywhere để kết nối hybrid node một cách an toàn với EKS control plane của bạn.

EKS Hybrid Nodes cũng hỗ trợ một số Amazon EKS add-on và feature cho cluster networking, observability, và pod credential bao gồm CoreDNS, kube-proxy, [Amazon Managed Service for Prometheus](https://aws.amazon.com/prometheus/) agent-less scraper, [AWS Distro for Open Telemetry](https://aws.amazon.com/otel/), CloudWatch Observability Agent, IAM Roles for Service Accounts (IRSA), và EKS Pod Identities. Cho pod networking, Cilium và Calico Container Networking Interface (CNI) được hỗ trợ để sử dụng với hybrid node.

Để biết thông tin chi tiết về cách EKS Hybrid Nodes hoạt động, xem [EKS Hybrid Nodes user guide](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-overview.html).

## Kiến trúc

Trước khi sử dụng EKS Hybrid Nodes, bạn phải hiểu networking flow giữa cloud-hosted Amazon EKS control plane và hybrid node chạy trong environment của bạn. Node và pod network mà bạn sử dụng cho hybrid node và resource chạy trên chúng phải sử dụng IPv4 RFC-1918 Classless Inter-Domain Routing (CIDR). Bạn truyền CIDR cho các on-premises node và pod network này khi bạn tạo hybrid nodes-enabled EKS cluster của bạn. VPC và on-premises routing table của bạn phải được cấu hình với các network này cho end-to-end hybrid nodes traffic flow. Để biết thêm thông tin về networking requirement cho hybrid node, xem [Prepare networking for hybrid nodes](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-networking.html) trong Amazon EKS user guide.

![Hybrid Networking Architecture](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/01/27/Picture1-4.jpg)
*Hình 1: Kiến trúc hybrid networking cho EKS Hybrid Nodes*

Bảng sau tóm tắt các phần chính của kiến trúc networking cho hybrid node.

| **Environment** | **Component** | **Description** |
|-----------------|---------------|-----------------|
| AWS Region | EKS cluster configuration | [RemoteNodeNetwork](https://docs.aws.amazon.com/eks/latest/APIReference/API_RemoteNodeNetwork.html) của EKS cluster configuration cần thiết cho EKS control plane đến kubelet communication cho Kubernetes operation, như log, exec, và port-forward. |
| AWS Region | EKS cluster configuration | [RemotePodNetwork](https://docs.aws.amazon.com/eks/latest/APIReference/API_RemotePodNetwork.html) của EKS cluster configuration cần thiết cho EKS control plane đến webhook communication. Được khuyến nghị cấu hình `RemotePodNetwork` của bạn, nhưng nếu bạn không chạy webhook trên hybrid node, thì nó không thực sự cần thiết. |
| AWS Region | EKS cluster VPC | VPC routing table của bạn phải có route cho `RemoteNodeNetwork` và `RemotePodNetwork` của bạn đến gateway mà bạn đang sử dụng cho traffic thoát khỏi VPC. Gateway thường là [AWS Transit Gateway](https://aws.amazon.com/transit-gateway/) hoặc Virtual Private Gateway (VGW). |
| AWS Region | EKS cluster security group | Bạn phải có inbound và outbound rule cho phép traffic cho `RemoteNodeNetwork` và `RemotePodNetwork`. |
| On-premises | On-premises firewall | Bạn phải cho phép inbound access cho EKS control plane và outbound access cho `RemoteNodeNetwork` và `RemotePodNetwork`. |
| On-premises | On-premises router | On-premises router của bạn phải có thể route traffic đến `RemoteNodeNetwork` và `RemotePodNetwork` của bạn. |
| On-premises | Container Networking Interface (CNI) | Overlay network CIDR mà bạn cấu hình trong CNI của bạn phải giống với `RemotePodNetwork`. Nếu bạn đang sử dụng host networking, thì node CIDR của bạn phải giống với `RemoteNodeNetwork`. |
## Hướng dẫn triển khai

Trong walkthrough này, chúng ta thiết lập IAM credential cho hybrid node sử dụng Systems Manager hybrid activation, tạo hybrid nodes-enabled EKS cluster, kết nối hybrid node với EKS cluster, và cài đặt Cilium CNI để làm cho hybrid node sẵn sàng chạy ứng dụng. Walkthrough này sử dụng [AWS Command Line Interface (AWS CLI)](https://aws.amazon.com/cli/) và [AWS CloudFormation](https://aws.amazon.com/cloudformation/) để tạo EKS cluster, nhưng bạn có thể sử dụng các interface khác bao gồm [AWS Management Console](https://aws.amazon.com/console/), eksctl CLI, hoặc [Terraform](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest#eks-hybrid-nodes).

### Yêu cầu tiên quyết

Các yêu cầu tiên quyết sau đây cần thiết để hoàn thành giải pháp này:

- Hybrid network connectivity giữa on-premises environment và AWS của bạn
- Hạ tầng dưới dạng physical hoặc virtual machine
- Operating system tương thích với hybrid node
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) phiên bản 2.22.8 hoặc mới hơn hoặc 1.36.13 hoặc mới hơn với credential phù hợp
- [eksctl CLI](https://eksctl.io/installation/)
- IAM user chạy các bước trong walkthrough này phải có IAM permission cho các action sau: `iam:CreatePolicy`, `iam:CreateRole`, `iam:AttachRolePolicy`, `ssm:CreateActivation`, và `eks:CreateCluster`

### Chuẩn bị credential cho hybrid node

Giống như EKS node chạy trên EC2 instance trong cloud, hybrid node cần IAM role để kết nối với EKS control plane. Sau đó, IAM role cho hybrid node được sử dụng với Systems Manager hybrid activation hoặc IAM Roles Anywhere để provision temporary IAM credential. Nói chung, Systems Manager hybrid activation được khuyến nghị nếu bạn không có Public Key Infrastructure (PKI) và certificate hiện có cho on-premises environment của bạn. Nếu bạn có PKI và certificate hiện có, thì bạn có thể sử dụng chúng với IAM Roles Anywhere.

IAM role mà bạn sử dụng cho hybrid node phải có các permission sau:

- Permission cho hybrid nodes CLI (`nodeadm`) để sử dụng action `eks:DescribeCluster` để thu thập thông tin về cluster được sử dụng để kết nối hybrid node với cluster. Nếu bạn không kích hoạt `eks:DescribeCluster` action, thì bạn phải truyền Kubernetes API endpoint, cluster CA bundle, và service IPv4 CIDR trong node configuration mà bạn truyền cho `nodeadm` khi bạn chạy `nodeadm init`.
- Permission cho kubelet để sử dụng container image từ [Amazon Elastic Container Registry (Amazon ECR)](https://aws.amazon.com/ecr/) như được định nghĩa trong [AmazonEC2ContainerRegistryPullOnly](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonEC2ContainerRegistryPullOnly.html)
- Nếu sử dụng Systems Manager, thì permission cho nodeadm init để sử dụng Systems Manager hybrid activation như được định nghĩa trong [AmazonSSMManagedInstanceCore](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonSSMManagedInstanceCore.html) policy và permission để sử dụng `ssm:DeregisterManagedInstance` action và `ssm:DescribeInstanceInformation` action cho `nodeadm` uninstall để deregister instance.

Trong các bước này, chúng ta sử dụng AWS CLI và CloudFormation để tạo IAM role cho hybrid node với các permission được nêu trước đó. Sau đó, chúng ta sử dụng AWS CLI để tạo Systems Manager hybrid activation với IAM role của hybrid node.

Đầu tiên, tải xuống CloudFormation template vào máy nơi bạn chạy AWS CLI.

```bash
curl -OL 'https://raw.githubusercontent.com/aws/eks-hybrid/refs/heads/main/example/hybrid-ssm-cfn.yaml'
```

Theo mặc định, CloudFormation template thu hẹp permission cho `ssm:DeregisterManagedInstance` sao cho IAM role của hybrid node chỉ có thể deregister instance được liên kết với hybrid activation mà bạn tạo cho cluster. `SSMDeregisterConditionTagKey` và `SSMDeregisterConditionTagValue` được sử dụng trong permission cho IAM role của hybrid node phải tương ứng với tag mà bạn áp dụng khi bạn tạo Systems Manager hybrid activation của bạn, được hiển thị trong bước tiếp theo.

```bash
# Define environment variables
EKS_CLUSTER_NAME=my-hybrid-cluster
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
AWS_REGION=${AWS_REGION:=us-west-2}
EKS_CLUSTER_ARN=arn:aws:eks:${AWS_REGION}:${AWS_ACCOUNT_ID}:cluster/${EKS_CLUSTER_NAME}
ROLE_NAME=AmazonEKSHybridNodesRole

# Create cfn-ssm-parameters.json
cat << EOF > cfn-ssm-parameters.json
{
  "Parameters": {
    "RoleName": "$ROLE_NAME",
    "SSMDeregisterConditionTagKey": "EKSClusterARN",
    "SSMDeregisterConditionTagValue": "$EKS_CLUSTER_ARN"
  }
}
EOF
```

Deploy CloudFormation stack. Thay thế AWS_REGION bằng `AWS Region` mong muốn của bạn nơi hybrid activation được tạo. Region cho hybrid activation phải giống với Region cho EKS cluster của bạn.

```bash
aws cloudformation deploy \
    --stack-name EKSHybridRoleSSM \
    --region ${AWS_REGION} \
    --template-file hybrid-ssm-cfn.yaml \
    --parameter-overrides file://cfn-ssm-parameters.json \
    --capabilities CAPABILITY_NAMED_IAM
```

Sau khi tạo hybrid nodes IAM role, bước tiếp theo là tạo Systems Manager hybrid activation với role. Theo mặc định, Systems Manager hybrid activation hoạt động trong 24 giờ và max expiration là 30 ngày. Bạn có thể chỉ định `--expiration-date` khi bạn tạo hybrid activation của bạn trong timestamp format, như `2024-08-01T00:00:00`. Khi bạn sử dụng Systems Manager làm credential provider của bạn, node name cho hybrid node của bạn không thể cấu hình được, và được auto-generate bởi Systems Manager với format `mi-012345678abcdefgh`. Bạn có thể xem và quản lý Systems Manager managed instance trong Systems Manager console dưới Fleet Manager.

Sử dụng lệnh sau để tạo Systems Manager hybrid activation, truyền IAM role được tạo trong bước trước trong flag `--iam-role`. Lưu ý các tag chúng ta áp dụng khi chúng ta tạo hybrid activation tương ứng với trust policy được cấu hình cho IAM role của hybrid node được tạo trong bước trước. Hãy đảm bảo lưu output của lệnh Systems Manager create-activation, chứa activation code và activation ID mà bạn sử dụng trong bước tiếp theo khi kết nối hybrid node với EKS cluster của bạn.

```bash
# Define environment variables
EKS_CLUSTER_NAME=my-hybrid-cluster
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
AWS_REGION=${AWS_REGION:=us-west-2}
EKS_CLUSTER_ARN=arn:aws:eks:${AWS_REGION}:${AWS_ACCOUNT_ID}:cluster/${EKS_CLUSTER_NAME}
ROLE_NAME=AmazonEKSHybridNodesRole

# Create SSM hybrid activation
aws ssm create-activation \
     --region ${AWS_REGION} \
     --default-instance-name eks-hybrid-nodes \
     --description "Activation for EKS hybrid nodes" \
     --iam-role ${ROLE_NAME} \
     --tags Key=EKSClusterARN,Value=${EKS_CLUSTER_ARN} \
     --registration-limit 5
```

### Tạo EKS cluster cho hybrid node

Trong các bước này, chúng ta sử dụng AWS CLI và CloudFormation để tạo EKS cluster IAM role và hybrid nodes-enabled EKS cluster.

Đầu tiên, tải xuống CloudFormation template vào máy nơi bạn chạy AWS CLI.

```bash
curl -OL 'https://raw.githubusercontent.com/aws/eks-hybrid/refs/heads/main/example/hybrid-eks-cfn.yaml'
```

Theo mặc định, CloudFormation template tạo EKS cluster với private endpoint connectivity, có nghĩa là Kubernetes API endpoint chỉ có thể được truy cập thông qua VPC của bạn.

Nếu bạn muốn có public endpoint connectivity, thì bạn có thể đặt ClusterEndpointConnectivity thành Public trong CloudFormation parameters file của bạn.

Trong ví dụ CloudFormation parameters file sau, chúng ta sử dụng subnet hiện có đáp ứng hybrid nodes requirement. Chúng ta đang sử dụng subnet trong VPC có attachment với Transit Gateway được kết nối với on-premises environment thông qua Direct Connect. Amazon EKS attach Elastic Network Interface (ENI) vào subnet được cung cấp cho EKS control plane đến VPC connectivity. CloudFormation template cũng tạo security group cho phép traffic to/from `RemoteNodeCIDR` và `RemotePodCIDR`, và EKS control plane.

Thay thế các giá trị trong file `cfn-eks-parameters.json` bằng các giá trị cho environment riêng của bạn.

```bash
cat << EOF > cfn-eks-parameters.json
{
  "Parameters": {
    "ClusterName": "my-hybrid-cluster",
    "ClusterRoleName": "EKSHybridClusterRole",
    "SubnetId1": "subnet-0b65cdc4812345678",
    "SubnetId2": "subnet-02f526cd012345678",
    "VpcId": "vpc-0a5f3bee960d6ec71",
    "RemoteNodeCIDR": "10.80.150.0/24",
    "RemotePodCIDR": "10.80.2.0/23",
    "K8sVersion": "1.31"
  }
}
EOF
```

Deploy CloudFormation stack. Thay thế `AWS_REGION` bằng AWS Region mong muốn của bạn nơi cluster được tạo.

```bash
aws cloudformation deploy \
    --stack-name EKSHybridCluster \
    --region ${AWS_REGION} \
    --template-file hybrid-eks-cfn.yaml \
    --parameter-overrides file://cfn-eks-parameters.json \
    --capabilities CAPABILITY_NAMED_IAM
```

Cluster provisioning mất vài phút. Bạn có thể kiểm tra trạng thái stack của bạn với lệnh sau.

```bash
aws cloudformation describe-stacks \
    --stack-name EKSHybridCluster \
    --region ${AWS_REGION} \
    --query 'Stacks[].StackStatus'
```

Khi EKS cluster được tạo, tạo Amazon EKS access entry với IAM role cho hybrid node của bạn để cho phép node của bạn join cluster. Để biết thêm thông tin, xem [Prepare cluster access for hybrid nodes](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-cluster-prep.html) trong Amazon EKS user guide.

```bash
# Define environment variables
EKS_CLUSTER_NAME=my-hybrid-cluster
ROLE_NAME=AmazonEKSHybridNodesRole

# Create access entry with type HYBRID_LINUX
aws eks create-access-entry \
    --cluster-name ${EKS_CLUSTER_NAME} \
    --principal-arn ${ROLE_NAME} \
    --type HYBRID_LINUX
```

### Cài đặt và kết nối hybrid node với EKS cluster

Sau khi tạo IAM role cho hybrid node, Systems Manager hybrid activation, và hybrid nodes-enabled EKS cluster, bạn đã sẵn sàng tạo và attach hybrid node với cluster của bạn. Bạn có thể sử dụng bất kỳ x86_64 hoặc ARM physical hoặc virtual machine (VM) nào miễn là nó thỏa mãn các prerequisite trước đó. Hybrid nodes CLI, được gọi là `nodeadm`, được thiết kế để tối ưu hóa lifecycle management của hybrid node, bao gồm installation, configuration, và registration. Bạn có thể đã quen thuộc với `nodeadm` nếu bạn đã build custom AMI cho Amazon EKS dựa trên AL2023 Amazon EKS-optimized AMI. Lưu ý rằng cloud version của `nodeadm` được sử dụng trong AL2023 Amazon EKS-optimized AMI khác với hybrid nodes `nodeadm` version, và bạn nên sử dụng version phù hợp dựa trên deployment target của bạn.

Hybrid nodes CLI thực hiện hai chức năng của bootstrap process. Đầu tiên, nó cài đặt các dependency cần thiết trên host (kubelet, containerd, Systems Manager agent/IAM Roles Anywhere tool, v.v.). Thứ hai, nó cấu hình và khởi động các dependency để node có thể join EKS cluster. Amazon EKS cung cấp [Packer template](https://github.com/aws/eks-hybrid/tree/main/example/packer) để tạo Ubuntu và RHEL image cho hybrid node. Nếu bạn sẽ tạo hybrid node lặp đi lặp lại hoặc muốn tự động hóa bootstrap process, thì sử dụng prebuilt image có thể tiết kiệm thời gian và loại bỏ nhu cầu pull dependency như các process riêng biệt trên mỗi individual host.

Để cài đặt hybrid nodes dependency, chạy lệnh `nodeadm install`. Trong ví dụ sau, chúng ta sử dụng Kubernetes version `1.31` và `ssm` làm credential provider. EKS Hybrid Nodes hỗ trợ cùng Kubernetes version như Amazon EKS, bao gồm Kubernetes version dưới standard và extended support.

Lưu ý rằng `nodeadm` phải được chạy với user có root/sudo privilege trên host.

```bash
sudo nodeadm install 1.31 --credential-provider ssm
```

Khi node của bạn có các dependency cần thiết, tạo `nodeConfig.yaml` với configuration của bạn. Node configuration file bao gồm hai chi tiết chính: cluster information và mechanism được sử dụng cho credential (Systems Manager hybrid activation hoặc IAM Roles Anywhere).

Sau đây là ví dụ về file nodeConfig.yaml cho hybrid node sử dụng Systems Manager hybrid activation. Thay thế `SSM_ACTIVATION_CODE` và `SSM_ACTIVATION_ID` bằng các giá trị từ output của bước Systems Manager create activation trước đó.

```yaml
apiVersion: node.eks.aws/v1alpha1
kind: NodeConfig
spec:
  cluster:
    name: my-hybrid-cluster
    region: us-west-2
  hybrid:
    ssm:
      activationCode: SSM_ACTIVATION_CODE
      activationId: SSM_ACTIVATION_ID
```

Để kết nối hybrid node của bạn với EKS cluster của bạn, chạy lệnh nodeadm init với `nodeConfig.yaml` của bạn.

```bash
sudo nodeadm init -c file://nodeConfig.yaml
```

Nếu lệnh trước hoàn thành thành công và không có lỗi trong kubelet log, thì hybrid node của bạn đã join EKS cluster của bạn. Bạn có thể xác minh điều này trong EKS console bằng cách điều hướng đến **Compute tab** cho cluster của bạn ([đảm bảo IAM principal có permission để xem](https://docs.aws.amazon.com/eks/latest/userguide/view-kubernetes-resources.html#view-kubernetes-resources-permissions)) hoặc với `kubectl get nodes`.

```bash
NAME                   STATUS     ROLES    AGE    VERSION
mi-036ecab1709d75ee1   Not Ready  <none>   1h     v1.31.2-eks-94953ac
```

Nếu bạn chưa cài đặt alternative CNI trong cluster, thì các node bạn kết nối vẫn ở trạng thái `Not Ready` cho đến khi CNI được cài đặt và chạy.

### Cài đặt CNI cho hybrid node

Cilium và Calico được hỗ trợ làm CNI cho hybrid node. Bạn có thể quản lý các CNI này với lựa chọn tooling của bạn như Helm. Amazon VPC CNI không tương thích với hybrid node, và VPC CNI được cấu hình với anti-affinity cho label `eks.amazonaws.com/compute-type: hybrid` theo mặc định. Để biết thêm thông tin về vận hành Cilium và Calico với hybrid node, xem [Configure a CNI for hybrid nodes](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-cni.html) trong Amazon EKS user guide.

Để đảm bảo rằng CNI DaemonSet chỉ được schedule trên hybrid node, bạn có thể cấu hình affinity cho label `eks.amazonaws.com/compute-type=hybrid`, được tự động áp dụng bởi `nodeadm` khi hybrid node join cluster. Label này cho phép kiểm soát workload placement, cho phép bạn xác định component nào, bao gồm CNI, nên hoặc không nên chạy trên hybrid node.

`cilium-values.yaml` sau đây hiển thị Helm value để cài đặt Cilium. Lưu ý affinity cho hybrid nodes label và IP Address Management (IPAM) setting. Trong ví dụ, chúng ta đang sử dụng cluster-pool overlay IPAM mode, nơi bạn cấu hình `clusterPoolIPv4PodCIDRList` của bạn, nên tương ứng với `RemotePodNetwork` CIDR mà bạn đã chỉ định trong quá trình tạo EKS cluster. Thay thế 10.80.2.0/23 trong ví dụ bằng giá trị cho RemotePodNetwork của bạn. Trong ví dụ này, `clusterPoolIPv4MaskSize` được đặt thành 25, cho phép 128 IP address mỗi node.

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: eks.amazonaws.com/compute-type
          operator: In
          values:
          - hybrid
ipam:
  mode: cluster-pool
  operator:
    clusterPoolIPv4MaskSize: 25
    clusterPoolIPv4PodCIDRList:
    - 10.80.2.0/23
operator:
  unmanagedPodWatcher:
    restart: false
```

Sau khi tạo file `cilium-values.yaml` với setting của bạn, bạn có thể cài đặt Cilium sử dụng Helm.

```bash
CILIUM_VERSION=1.16.4

helm repo add cilium https://helm.cilium.io/

helm install cilium cilium/cilium \
    --version ${CILIUM_VERSION} \
    --namespace kube-system \
    --values cilium-values.yaml    
```

Sau khi deploy CNI, chạy `kubectl get nodes` lại và xác minh rằng node ở trạng thái Ready.

```bash
NAME                   STATUS     ROLES    AGE    VERSION
mi-036ecab1709d75ee1   Ready      <none>   1h     v1.31.2-eks-94953ac
```

### Ingress và load balancing cho workload chạy trên hybrid node

Đối với nhiều use case, workload chạy trên Kubernetes cluster cần được expose bên ngoài cluster để external resource truy cập chúng. Trong Kubernetes điều này thường được thực hiện bằng cách expose Service thông qua ingress và load balancer. Với hybrid node, có hai path chung cho application traffic. Đầu tiên là cho application traffic bắt nguồn trong Region và liên hệ workload chạy on-premises trên hybrid node. Thứ hai là cho application traffic bắt nguồn từ on-premises environment và ở local với on-premises environment.

Đối với category đầu tiên của AWS Region-originating application traffic, bạn có thể sử dụng [AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html) và [Application Load Balancer (ALB)](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/) hoặc [Network Load Balancer (NLB)](https://aws.amazon.com/elasticloadbalancing/network-load-balancer/) với target type ip cho workload trên hybrid node được kết nối với Direct Connect hoặc Site-to-Site VPN. Vì [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/) sử dụng [webhook](https://kubernetes.io/docs/reference/access-authn-authz/webhook/), bạn phải cấu hình RemotePodNetwork của bạn khi tạo EKS cluster của bạn nếu bạn chạy AWS Load Balancer Controller trên hybrid node của bạn.

Đối với category thứ hai của local application traffic, có nhiều partner và Kubernetes community option có sẵn để sử dụng với hybrid node. Khi điều hướng các option, hãy xem xét công nghệ hiện có trong on-premises environment của bạn và application requirement của bạn. Các option phổ biến cho on-premises environment bao gồm Cilium (BGP hoặc L2-aware load balancing), Calico (BGP load balancing), MetalLB, NGINX, HAProxy, Apache APISIX, Emissary Ingress, và Citrix Ingress. Cũng có các service mesh technology như Istio cung cấp khả năng tương tự trong số các chức năng khác. Nói chung, Amazon EKS và hybrid node tương thích 100% upstream Kubernetes, và hầu hết Kubernetes option cho ingress và load balancing có thể được sử dụng cho ứng dụng của bạn chạy trên hybrid node.

### Dọn dẹp tài nguyên

Bạn có thể xóa các resource được tạo trong các bước trước để tránh phát sinh phí với các lệnh sau. Nếu bạn sử dụng CloudFormation stack name khác, thì thay thế `EKSHybridRoleSSM` và `EKSHybridCluster` bằng stack name của bạn trong các lệnh sau.

```bash
aws cloudformation delete-stack --stack-name EKSHybridCluster

aws cloudformation delete-stack --stack-name EKSHybridRoleSSM

# to remove hybrid nodes components from your hosts
sudo nodeadm uninstall --skip node-validation,pod-validation
```
## Launch Partner

Một loạt partner bao gồm Independent Software Vendor (ISV), Independent Hardware Vendor (IHV), và Operating System vendor (OSV) đã tham gia vào hybrid nodes launch. Chúng tôi rất vui mừng được làm việc với họ và trong cộng đồng Kubernetes. Sau đây là danh sách các partner đã tham gia launch này. Một số ISV được liệt kê đã validate software solution của họ thông qua [Conformitron](https://aws.amazon.com/blogs/containers/conformitron-validate-third-party-software-with-amazon-eks-and-amazon-eks-anywhere/), một framework để validate third-party software với Amazon EKS và EKS Anywhere, mở rộng GitOps driven integration của họ sang EKS Hybrid Nodes. Người dùng có thể deploy các validated solution mà các partner này cung cấp để vận hành hybrid node của họ, giải quyết các production readiness area phổ biến như secrets management, storage, và maintenance của third-party component trên distributed fleet của device.

### Các partner chính:

- **[AccuKnox](https://www.accuknox.com/blog/securing-eks-hybrid-node)** (ISV): Công ty cybersecurity tập trung cung cấp zero-trust security solution cho cloud-native và Kubernetes environment.

- **[AMD](https://www.amd.com/en/solutions/data-center/cloud-computing.html)** (IHV): Công ty semiconductor hàng đầu với EPYC processor cho data center và cloud computing.

- **[Aqua](https://www.aquasec.com/)** (ISV): Nhà cung cấp hàng đầu cloud-native security solution cho containerized và serverless environment.

- **[CIQ (Ctrl IQ)](https://ciq.com/blog/setup-an-aws-eks-hybrid-kubernetes-cluster-with-rocky-linux/)** (OSV): Chuyên về high-performance computing (HPC) solution và enterprise support cho Rocky Linux.

- **[Dell Technologies](https://infohub.delltechnologies.com/en-us/p/what-if-unleashing-innovation-with-amazon-eks-hybrid-nodes-and-dell-powerflex/)** (IHV): Công ty công nghệ toàn cầu hàng đầu với PowerEdge server và VxRail hyperconverged infrastructure.

- **[Dynatrace](https://www.dynatrace.com/news/blog/new-integrations-announced-at-aws-reinvent-enhance-cloud-performance-security-and-automation/)** (ISV): Software intelligence platform cho application performance monitoring (APM) và observability.

- **[HashiCorp](https://www.hashicorp.com/blog/terraform-launch-day-support-amazon-s3-tables-eks-hybrid-nodes-and-more)** (ISV): Cung cấp Terraform, Vault, và Consul cho infrastructure as code và service networking.

- **[Kong](https://konghq.com/)** (ISV): API gateway và service connectivity platform cho Kubernetes environment.

- **[NetApp](https://community.netapp.com/t5/Tech-ONTAP-Blogs/NetApp-and-Amazon-EKS-Hybrid-Nodes-Pioneering-the-future-of-hybrid-cloud/ba-p/456940)** (ISV): Leader trong cloud data service và storage solution với Astra product line.

- **[Spectro Cloud](https://www.spectrocloud.com/blog/eks-hybrid-nodes)** (ISV): Kubernetes management platform cho multi-environment deployment.

## Kết luận

Chạy workload với Kubernetes on-premises hoặc tại edge thường mất thời gian, effort, và maintenance để định nghĩa và tích hợp tooling và process với open source Kubernetes. Điều này thêm operational burden lên team và tạo silo giữa on-premises và cloud environment. Bạn có thể giảm toil này với EKS Hybrid Nodes và có thể đưa on-premises deployment của bạn phù hợp hơn với cách bạn chạy workload trong cloud.

Cho dù bạn đang tìm cách modernize on-premises application của mình, sử dụng on-premises hardware hiện có, hoặc đáp ứng data residency requirement bằng cách giữ dữ liệu trong một quốc gia cụ thể, bạn có thể sử dụng EKS Hybrid Nodes để chạy on-premises workload của mình một cách hiệu quả mà không phải đối phó với operational overhead của việc quản lý Kubernetes control plane.

Để tìm hiểu thêm và bắt đầu với EKS Hybrid Nodes, truy cập [EKS Hybrid Nodes User Guide](https://docs.aws.amazon.com/eks/latest/userguide/hybrid-nodes-overview.html) và xem [re:Invent 2024 session](https://www.youtube.com/watch?v=ZxC7SkemxvU) (KUB205) nơi chúng tôi đề cập cách hybrid nodes hoạt động, feature của nó, và best practice.

---

