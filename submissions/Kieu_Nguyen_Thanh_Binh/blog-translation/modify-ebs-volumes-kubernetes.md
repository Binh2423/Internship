# Chỉnh sửa Amazon EBS volume trên Kubernetes với Volume Attributes Classes

> **📖 Bài viết gốc**: [Modify Amazon EBS volumes on Kubernetes with Volume Attributes Classes](https://aws.amazon.com/blogs/containers/modify-amazon-ebs-volumes-on-kubernetes-with-volume-attributes-classes/)  
> **👤 Tác giả**: Kevin Liu (Senior PMT), Jens-Uwe Walther (Senior STAM-Containers), và Drew Sirenko (Software Dev Engineer)  
> **📅 Ngày xuất bản**: 05 Tháng 3, 2025  
> **🌐 Nguồn**: AWS Containers Blog  
> **👨‍💻 Người dịch**: Kiều Nguyễn Thanh Bình - FCJ Intern  
> **📅 Ngày dịch**: 10 Tháng 7, 2025  


---

## 📋 Tóm tắt

Bài viết này khám phá cách chỉnh sửa Amazon EBS volume trên Kubernetes mà không gây downtime cho ứng dụng. Học cách sử dụng VolumeAttributesClass API cùng với Amazon EBS Container Storage Interface (CSI) driver để điều chỉnh hiệu suất provisioned, migrate sang gp3 volume, và tự động hóa workflow backup dữ liệu. Tính năng này có sẵn trong Amazon EKS 1.31 và Amazon EBS CSI driver 1.35.0, cho phép thay đổi volume type, IOPS, throughput và AWS resource tag mà không cần restart pod.

**🎯 Đối tượng đọc**: Kubernetes Administrators, Storage Engineers, DevOps Engineers  
**📊 Độ khó**: Intermediate  
**🏷️ Tags**: Amazon EBS, Kubernetes, Volume Management, CSI Driver, Storage Classes

---

*Bài viết này được đồng tác giả bởi Kevin Liu (Senior PMT), Jens-Uwe Walther (Senior STAM-Containers), và Drew Sirenko (Software Dev Engineer).*

## Giới thiệu

Trong bài viết này, chúng ta khám phá cách chỉnh sửa [Amazon Elastic Block Store (Amazon EBS)](https://aws.amazon.com/ebs/) volume trên Kubernetes mà không gây downtime cho ứng dụng. Học cách sử dụng [VolumeAttributesClass API](https://kubernetes.io/docs/concepts/storage/volume-attributes-classes/) cùng với [Amazon EBS Container Storage Interface (CSI) driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html) để điều chỉnh hiệu suất provisioned, migrate sang gp3 volume, và tự động hóa workflow backup dữ liệu của bạn.

Các ứng dụng containerized hiện đại sử dụng persistent storage, như data analytics, database, hoặc video encoding và decoding, cần nhiều đặc tính storage đa dạng như low latency, high throughput, hoặc nhiều hơn nữa. Amazon EBS là một lựa chọn phù hợp cho các workload này. EBS volume cung cấp các loại storage khác nhau, các thuộc tính storage có thể cấu hình như IOPS và throughput, và khả năng chỉnh sửa các thuộc tính này mà không gây downtime.

Nếu bạn triển khai stateful container workload của mình trên Kubernetes, thì bạn có thể sử dụng Amazon EBS CSI driver để provision và quản lý EBS volume thay mặt cho bạn. Khi bạn tạo [Persistent Volume Claim (PVC)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#introduction) resource với kích thước được chỉ định và một [Storage Class (SC)](https://kubernetes.io/docs/concepts/storage/storage-classes/), Kubernetes làm việc với Amazon EBS CSI driver để triển khai workload của bạn với storage cần thiết bằng cách tạo, attach, và format EBS volume thay mặt cho bạn.

Việc thay đổi đặc tính storage của volume trong Kubernetes từng là một hành động offline, đòi hỏi cluster operator tạo SC khác, và migrate sang PVC và [Persistent Volume (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) resource mới. Quy trình này đòi hỏi downtime ứng dụng và ảnh hưởng đến availability. Đó là lý do tại sao, vào năm 2023, AWS đã phát hành [volume-modifier-for-k8s sidecar](https://github.com/awslabs/volume-modifier-for-k8s), cho phép cập nhật online các loại volume và đặc tính hiệu suất bằng cách áp dụng annotation vào PVC. Mặc dù việc cung cấp giải pháp cho người dùng là quan trọng, AWS đã làm việc với các storage provider khác để tạo giải pháp chỉnh sửa volume có sẵn như một native Kubernetes API. Trong năm qua AWS đã làm việc với cộng đồng Kubernetes để phát hành [VolumeAttributesClass Kubernetes Enhancement](https://github.com/kubernetes/enhancements/blob/master/keps/sig-storage/3751-volume-attributes-class/README.md), đã đạt beta trong Kubernetes 1.31. Mặc dù tính năng bị vô hiệu hóa theo mặc định trong upstream Kubernetes, nó được tự động kích hoạt cho bạn trong [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/) 1.31 cluster của bạn.

Bắt đầu từ Amazon EKS 1.31, bạn có thể sử dụng tính năng [*VolumeAttributesClass (VAC)*](https://kubernetes.io/docs/concepts/storage/volume-attributes-classes/) để chỉnh sửa volume type, IOPS, hoặc throughput của EBS volume mà không cần volume-modifier-for-k8s sidecar. Hơn nữa, bạn có thể sử dụng VAC để thêm, chỉnh sửa, và xóa AWS resource tag từ volume của bạn. Amazon EBS CSI driver kích hoạt tính năng này theo mặc định khi bạn nâng cấp lên phiên bản 1.35.0 hoặc mới hơn.

Sử dụng Amazon EKS 1.31 và Amazon EBS CSI driver 1.35.0, bạn đã có thể bắt đầu sử dụng VAC thay vì PVC annotation. Trong hai phần tiếp theo, chúng tôi giới thiệu VolumeAttributesClass và cho bạn thấy cách kích hoạt tính năng trên cluster của bạn. Sau đó, chúng tôi hướng dẫn qua ba workflow phổ biến để chỉnh sửa volume của bạn với VAC:

1. Điều chỉnh throughput và input output operations per second (IOPS) performance characteristic của volume của bạn.
2. Migrate sang EBS [gp3 volume](https://aws.amazon.com/blogs/storage/migrate-your-amazon-ebs-volumes-from-gp2-to-gp3-and-save-up-to-20-on-costs/) để tiết kiệm tới 20% giá thấp hơn mỗi GB so với gp2 volume.
3. Chỉnh sửa EBS volume resource tag của bạn để tự động hóa workflow backup dữ liệu với [Amazon Data Lifecycle Manager](https://docs.aws.amazon.com/ebs/latest/userguide/snapshot-lifecycle.html).

## Tổng quan giải pháp

Cluster operator có thể dựa vào Amazon EBS CSI driver để quản lý declaratively persistent volume của họ. Bạn có thể yêu cầu persistent storage cho workload của mình bằng cách tạo PVC resource với kích thước, SC, và VAC cụ thể.

Một VAC được tạo thành từ tên của nó, driverName của CSI driver, và danh sách các parameter có thể thay đổi cụ thể cho storage-provider như Amazon EBS.

Các parameter EBS volume có thể thay đổi, như throughput, IOPS, volume type, và AWS resource tag có thể được chỉ định trong VAC resource:

```yaml
apiVersion: storage.k8s.io/v1beta1
kind: VolumeAttributesClass
metadata:
  name: ebs-blog-gp3-standard-performance
driverName: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  tagSpecification_1: "performance=standard"
```

Bạn có thể thêm VAC vào PVC sử dụng trường `spec.volumeAttributesClassName`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-blog-overview
spec:
  ...
  volumeAttributesClassName: ebs-blog-gp3-standard-performance
```

Volume được provision với các parameter được chỉ định trong cả VAC và SC, với các parameter VAC có ưu tiên hơn bất kỳ parameter xung đột nào. VAC của PVC cũng ghi đè tất cả modification dựa trên annotation, và ngăn chặn modification dựa trên annotation trong tương lai.

Resource VAC riêng biệt này có lợi cho các tổ chức chia trách nhiệm giữa các team vận hành cluster và các team triển khai ứng dụng lên nó. Cluster operator có thể tạo cluster-wide SC và VAC object, sau đó có thể được sử dụng bởi application developer trong PVC resource có namespace của họ.
## Yêu cầu tiên quyết

Trên Amazon EKS 1.31 hoặc mới hơn, beta `VolumeAttributesClass` feature gate và `storage.k8s.io/v1beta1` API group được tự động kích hoạt trên các thành phần control plane của cluster của bạn. Hơn nữa, bạn phải sử dụng Amazon EBS CSI driver phiên bản v1.35.0 hoặc mới hơn, được cài đặt bởi [Amazon EKS Managed add-on](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html) v1.35.0-eksbuild.2 hoặc [Helm chart](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md) v2.35.1.

Nếu bạn vận hành một self-managed (ví dụ, kOps) Kubernetes v1.31 cluster, bạn phải kích hoạt `VolumeAttributesClass` feature gate trên kube-apiserver, kube-scheduler, và kube-controller-manager, cũng như kích hoạt `storage.k8s.io/v1beta1` API group thông qua kube-apiserver runtime-config.

Tham khảo [Kubernetes VolumeAttributesClass documentation](https://kubernetes.io/docs/concepts/storage/volume-attributes-classes/) để có danh sách đầy đủ các yêu cầu.

Để chỉnh sửa Amazon EBS resource tag thông qua VAC, hãy đảm bảo rằng bạn attach [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) Policy sau vào role được sử dụng bởi Amazon EBS CSI driver của bạn:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:CreateTags"
      ],
      "Resource": [
        "arn:aws:ec2:*:*:volume/*",
        "arn:aws:ec2:*:*:snapshot/*"
      ]
    }
  ]
}
```

## Hướng dẫn triển khai

Các bước sau đây hướng dẫn bạn qua giải pháp này.

### Provision một volume và tăng hiệu suất của nó

Amazon EKS 1.31 không đi kèm với SC theo mặc định. Chúng ta phải tạo SC sau:

```bash
$ kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-blog-sc
provisioner: ebs.csi.aws.com
allowVolumeExpansion: true
EOF
```

Đây là hai ví dụ VAC resource, mỗi cái với các parameter quality-of-service volume khác nhau:

```bash
$ kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1beta1
kind: VolumeAttributesClass
metadata:
  name: ebs-blog-gp3-standard-performance
driverName: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
---
apiVersion: storage.k8s.io/v1beta1
kind: VolumeAttributesClass
metadata:
  name: ebs-blog-gp3-increased-performance
driverName: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "4000"
  throughput: "130"
EOF
```

Provision một volume với standard performance VAC:

```bash
$ kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-blog-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-blog-sc
  volumeAttributesClassName: ebs-blog-gp3-standard-performance
  resources:
    requests:
      storage: 10Gi
EOF
```

Để có hiệu suất cao hơn, bạn có thể patch PVC để trỏ đến VAC `ebs-blog-gp3-increased-performance` mới:

```bash
$ kubectl patch pvc ebs-blog-claim -p '{"spec": { "volumeAttributesClassName": "ebs-blog-gp3-increased-performance" }}'
```

Amazon EBS CSI driver quan sát rằng VAC hiện tại và mong muốn của PVC khác nhau, chỉnh sửa EBS volume, và cập nhật PV resource sang VAC mới.

Bạn chỉ có thể thực hiện một volume modification mỗi khoảng thời gian sáu giờ theo [tài liệu Amazon EBS](https://docs.aws.amazon.com/ebs/latest/userguide/modify-volume-requirements.html#elastic-volumes-limitations). Do đó, bạn nên merge việc tăng kích thước storage PVC với thay đổi VAC thành một Kubernetes patch:

```bash
$ kubectl patch pvc ebs-blog-claim --type json -p '[{"op": "replace", "path": "/spec/volumeAttributesClassName", "value": "ebs-blog-gp3-increased-performance" }, {"op": "replace", "path": "/spec/resources/requests/storage", "value": "11Gi"}]'
```

Nếu không, bạn sẽ quan sát failure event sau trên PVC resource:

```bash
VolumeResizeFailed: "pvc-716766d1-58d5-411b-8d4a-7671ef9894e9" by resizer "ebs.csi.aws.com" failed… VolumeModificationRateExceeded: You've reached the maximum modification rate per volume limit. Wait at least 6 hours between modifications per EBS volume.
```

### Tiết kiệm chi phí bằng cách migrate sang gp3 volume

VolumeAttributesClass có thể được sử dụng với PersistentVolumeClaim hiện có được tạo mà không có VAC. Điều này đặc biệt hữu ích để migrate từ gp2 sang gp3 volume. EBS gp3 volume cho phép bạn provision IOPS và throughput độc lập với kích thước storage. Bạn có thể migrate sang gp3 volume trong bất kỳ use case nào mà gp2 volume được sử dụng, và có thể thấy tiết kiệm chi phí lên tới 20% vì sự độc lập này. Nếu bạn đang tìm kiếm hiệu suất cao hơn nữa, thì bạn có thể scale gp3 volume lên tới 1,000 MiB/s, cao gấp bốn lần throughput tối đa của gp2 volume.

Ở đây chúng ta tạo một stateful workload ban đầu dựa vào gp2 volume:

```bash
$ kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-blog-migration
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-blog-migration
  resources:
    requests:
      storage: 1Gi
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-blog-migration
provisioner: ebs.csi.aws.com
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp2
---
apiVersion: v1
kind: Pod
metadata:
  name: ebs-blog-migration-app
spec:
  containers:
  - name: ebs-blog-app
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date -u) >> /data/out.txt; sleep 5; done"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: ebs-blog-migration
EOF
```

Để migrate volume này sang gp3, trước tiên áp dụng VAC resource vào cluster của bạn:

```bash
$ kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1beta1
kind: VolumeAttributesClass
metadata:
  name: ebs-blog-gp3 
driverName: ebs.csi.aws.com
parameters:
  type: gp3
EOF
```

Sau đó patch *PVC* liên quan của nó:

```bash
$ kubectl patch pvc ebs-blog-migration -p '{"spec": {"volumeAttributesClassName": "ebs-blog-gp3"}}'
```

Lưu ý, tính đến Kubernetes phiên bản 1.31, bạn không thể chỉnh sửa volume tham chiếu in-tree SC thông qua VAC. Đảm bảo rằng SC liên kết với volume mà bạn đang chỉnh sửa tham chiếu provisioner `ebs.csi.aws.com`, không phải `kubernetes.io/aws-ebs`.

### Workflow chỉnh sửa tag

Để dễ dàng quản lý EBS volume của bạn, bạn có thể sử dụng tag. [Tag](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/what-are-tags.html) là các cặp key-value mà bạn gán cho AWS resource của mình, cho phép bạn phân loại chúng theo mục đích, chủ sở hữu, hoặc môi trường. Nếu bạn có nhiều volume trong tài khoản của mình, thì tag có thể giúp bạn xác định một volume cụ thể, hoặc nhóm các resource liên quan. Amazon EBS CSI driver cho phép bạn thêm, thay thế, và xóa volume resource tag thông qua [parameter VAC `tagSpecification`](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/tagging.md#storageclass-tagging).

Ngoài tổ chức, volume tag có thể được sử dụng với Amazon Data Lifecycle Manager policy để backup EBS volume của bạn một cách an toàn. [Amazon EBS Snapshot](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html) capture trạng thái của volume tại một thời điểm cụ thể, cho phép phục hồi trong trường hợp mất dữ liệu hoặc corruption. Để tự động hóa việc tạo, retention, và deletion của các snapshot này, bạn có thể tạo volume Amazon Data Lifecycle Manager policy nhắm mục tiêu volume với resource tag cụ thể.

Ví dụ, policy sau tạo daily snapshot của bất kỳ volume nào liên kết với VAC `ebs-blog-daily-backup`:

```bash
$ aws dlm get-lifecycle-policies
{
    "Policies": [
        {
            "PolicyId": "policy-045203fd0b8e2bf79",
            "Description": "This policy creates daily backups of my K8s PVs",
            "State": "ENABLED",
            "Tags": {
                "backup-interval": "daily"
            },
            "PolicyType": "EBS_SNAPSHOT_MANAGEMENT",
            "DefaultPolicy": false
        }
    ]
}
```

Đây là ví dụ VAC sẽ đặt resource tag cho target tag của policy:

```bash
$ kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1beta1
kind: VolumeAttributesClass
metadata:
  name: ebs-blog-daily-backup 
driverName: ebs.csi.aws.com
parameters:
  tagSpecification_1: "backup-interval=daily"
EOF
```

## Dọn dẹp tài nguyên

Để tránh phát sinh thêm chi phí, xóa bất kỳ Kubernetes và AWS resource ví dụ nào mà bạn đã provision cho các ví dụ này.

Xóa stateful workload example resource:

```bash
kubectl delete pod ebs-blog-migration-app
kubectl delete pvc ebs-blog-claim
kubectl delete pvc ebs-blog-migration
kubectl delete vac ebs-blog-gp3
kubectl delete vac ebs-blog-gp3-increased-performance
kubectl delete vac ebs-blog-gp3-standard-performance
kubectl delete vac ebs-blog-daily-backup 
kubectl delete sc ebs-blog-sc
kubectl delete sc ebs-blog-migration
```

Bạn có thể xác nhận việc xóa EBS volume thông qua [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/) console.

## Kết luận

Trong bài viết này chúng ta đã khám phá cách chỉnh sửa Amazon EBS volume trên Kubernetes với VolumeAttributesClass. Tính năng mới này giúp bạn điều chỉnh đặc tính hiệu suất của stateful workload bằng cách cập nhật volume type, IOPS, và throughput mà không gây downtime ứng dụng. Hơn nữa, tự động hóa workflow backup dữ liệu của bạn bằng cách chỉnh sửa resource tag để phù hợp với Amazon Data Lifecycle Manager policy của bạn.

Truy cập [trang sản phẩm Amazon EBS](https://aws.amazon.com/ebs/), [trang sản phẩm Amazon EKS](https://aws.amazon.com/eks/), và dự án open source trên [GitHub](https://github.com/kubernetes-sigs/aws-ebs-csi-driver) để tìm hiểu thêm về việc sử dụng EBS volume cho stateful workload của bạn.

Để biết thêm thông tin về tương tác giữa modification dựa trên annotation, VAC, và SC, đọc [Amazon EBS CSI driver FAQ](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/faq.md).

Cảm ơn bạn đã đọc bài viết này, và đừng ngần ngại để lại câu hỏi trong phần comment hoặc submit feature request trên [GitHub project](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/issues) của chúng tôi.

---

