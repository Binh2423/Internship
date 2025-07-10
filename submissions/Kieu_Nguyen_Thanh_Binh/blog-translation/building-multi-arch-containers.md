# Xây dựng container đa kiến trúc với GitHub Actions trên AWS

> **📖 Bài viết gốc**: [Building multi-arch containers with GitHub Actions in AWS](https://aws.amazon.com/blogs/containers/building-multi-arch-containers-with-github-actions-in-aws/)  
> **👤 Tác giả**: Zakiya Randall, Technical Account Manager và đồng tác giả với Muru Bhaskaran, Sr. Specialist Solutions Architect  
> **📅 Ngày xuất bản**: 14 Tháng 3, 2025  
> **🌐 Nguồn**: AWS Containers Blog  
> **👨‍💻 Người dịch**: Kiều Nguyễn Thanh Bình - FCJ Intern  
> **📅 Ngày dịch**: 10 Tháng 7, 2025  

---

## 📋 Tóm tắt

Bài viết này trình bày giải pháp sử dụng GitHub, GitHub Actions workflows, và AWS CodeBuild để xây dựng native container image cho cả x86 và AWS Graviton-based compute trên AWS. Giải pháp tận dụng CodeBuild managed GitHub Actions runners để tự động hóa việc build và push multi-arch image lên Amazon Elastic Container Registry (Amazon ECR). Điều này giúp hỗ trợ đa dạng kiến trúc tính toán và tối ưu hóa hiệu suất trên các nền tảng phần cứng khác nhau.

**🎯 Đối tượng đọc**: DevOps Engineers, Container Developers, Solutions Architects  
**📊 Độ khó**: Intermediate  
**🏷️ Tags**: Multi-arch Containers, GitHub Actions, AWS CodeBuild, Amazon ECR, AWS Graviton

---

*Blog này được tác giả bởi Zakiya Randall, Technical Account Manager và đồng tác giả với Muru Bhaskaran, Sr. Specialist Solutions Architect.*

## Giới thiệu

Khi bối cảnh tính toán tiếp tục phát triển, có sự nhấn mạnh ngày càng tăng về việc hỗ trợ một loạt đa dạng các kiến trúc tính toán. Sự thay đổi này được thúc đẩy bởi nhu cầu về tính linh hoạt, hiệu quả, và tối ưu hóa hiệu suất trên các nền tảng phần cứng khác nhau. Do đó, việc các developer và tổ chức xây dựng container image tương thích với nhiều kiến trúc (multi-arch) trở nên ngày càng quan trọng.

[AWS CodeBuild](https://aws.amazon.com/codebuild/) là một dịch vụ continuous integration được quản lý hoàn toàn hiện [hỗ trợ](https://aws.amazon.com/about-aws/whats-new/2024/04/aws-codebuild-managed-github-action-runners/) managed [GitHub Actions](https://docs.github.com/en/actions) runner, là các self-hosted runner cho phép người dùng cấu hình các CodeBuild project của họ để nhận GitHub Actions workflow job event. Trong bài viết này, chúng tôi trình bày một giải pháp sử dụng [GitHub](https://github.com/), GitHub Actions workflow, và CodeBuild để xây dựng native container image cho cả x86 và [AWS Graviton](https://aws.amazon.com/ec2/graviton/)-based compute trên AWS. Sau khi hoàn thành GitHub Actions workflow của chúng ta, chúng ta sẽ tiến hành push multi-arch image của chúng ta lên [Amazon Elastic Container Registry (Amazon ECR)](https://aws.amazon.com/ecr/).

## Tổng quan giải pháp

Sơ đồ kiến trúc minh họa workflow xảy ra khi commit một thay đổi vào GitHub repository, chi tiết các bước tiếp theo liên quan đến việc push container image lên Amazon ECR.

![Solution Architecture Diagram](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/02/Picture1.png)
*Hình 1: Sơ đồ kiến trúc giải pháp*

1. Commit một thay đổi vào GitHub repository kích hoạt workflow
2. CodeBuild runner cho cả hai kiến trúc được khởi động để build container image
3. Code được checkout từ GitHub repository
4. Credential được cấu hình để truy cập AWS account
5. Một role được sử dụng để đăng nhập vào Amazon ECR
6. Multi-arch image được build với danh sách image cho cả hai kiến trúc

## Yêu cầu tiên quyết

Các yêu cầu tiên quyết sau đây cần thiết để hoàn thành giải pháp này:

- AWS account
- [Amazon Command Line Interface (AWS CLI)](https://aws.amazon.com/cli/)
- Một GitHub repository

## Hướng dẫn triển khai

Các bước sau đây hướng dẫn bạn qua giải pháp này.

### Tạo file GitHub repository

Để bắt đầu tạo giải pháp, bạn cần một GitHub repository để lưu trữ Dockerfile, file index.html, và GitHub Actions workflow YAML file. Tham khảo [Creating a new GitHub repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) để có hướng dẫn từng bước. Cho ví dụ này, các file sau đây nên được commit vào root của GitHub repository.

- File `index.html`

```html
<!DOCTYPE html>
<html>
 <body>
   <h1>Containers</h1>
   <p>You can run containers!</p>
 </body>
</html>
```

- Hướng dẫn `Dockerfile` để build container image

```dockerfile
FROM public.ecr.aws/nginx/nginx
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

### Tạo hai CodeBuild project cho kiến trúc tính toán x86 và arm64

Bạn phải tạo hai CodeBuild project để chạy GitHub Actions job của bạn. CodeBuild project của bạn nên tuân theo cấu trúc đặt tên sau cho <*project-name*>-x86 và <*project-name*>-arm64. Tham khảo [tutorial](https://docs.aws.amazon.com/codebuild/latest/userguide/action-runner.html) sau để thiết lập hai CodeBuild project cho môi trường tính toán x86 và arm64. Bạn nên sử dụng phương thức xác thực OAuth app để kết nối với GitHub repository của bạn. Cho các CodeBuild project x86 và arm64, chọn supported environment image phù hợp với mỗi kiến trúc tính toán. **Buildspec** build specification của bạn bị bỏ qua. Thay vào đó, CodeBuild ghi đè nó để sử dụng các lệnh thiết lập compute runner.

![CodeBuild Projects](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/1111.png)
*Hình 2: CodeBuild Project cho x86 và arm64*

### Tạo Amazon ECR repository

Bạn cũng cần tạo Amazon ECR repository để lưu trữ x86 và arm64 container image. Chạy lệnh AWS CLI sau để tạo Amazon ECR repository.

```bash
aws ecr create-repository \
    --repository-name <repository-name>
```

Sau khi tạo và định nghĩa Amazon ECR repository, bạn phải định nghĩa một role để CodeBuild runner có thể có quyền truy cập và push image của bạn lên Amazon ECR repository của bạn. Role sau đây mà bạn tạo cho phép bạn push image lên Amazon ECR repository của bạn.

1. Đi đến **IAM console** và tạo policy với các quyền sau. Trong phần Resource cho AllowPushPull statement trong policy, thay thế [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) của bạn, số tài khoản, và tên repository với [Amazon Resource Name (ARN)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) liên kết đến private repository của bạn trong Amazon ECR. Chỉ định tên cho policy này và chọn **Create Policy**.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPushPull",
            "Effect": "Allow",
            "Action": [
                "ecr:BatchGetImage",
                "ecr:BatchCheckLayerAvailability",
                "ecr:CompleteLayerUpload",
                "ecr:GetDownloadUrlForLayer",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:UploadLayerPart"
            ],
            "Resource": [
                "arn:aws:ecr:us-east-1:xxxxxxxxxxxx:repository/<repository-name>"
            ]
        },
        {
            "Sid": "AllowLogin",
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Sau khi tạo policy, đi đến [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) console để tạo Identity Provider và chọn **OpenID Connect**. Cho Provider URL, chọn https://token.actions.githubusercontent.com. Cho Audience, chọn **sts.amazonaws.com**.

2. Sau khi bạn tạo provider, tạo role và chọn **Web Identity**. Trong dropdown box, bạn sẽ thấy **Provider URL** cho https://token.actions.githubusercontent.com. Chọn tùy chọn này và chỉ định sts.amazonaws.com cho **Audience** của bạn. Trong **GitHub organization**, chỉ định GitHub Organization của bạn và thêm repository mà bạn đã tạo trong thiết lập ban đầu. Chọn **Next**.

![Creating the Role](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/Figure-3-Example-of-creating-the-role.jpg)
*Hình 3: Ví dụ về tạo role*

3. Trên trang **Add Permissions**, chọn policy mà bạn đã tạo trong Bước 1 để bạn có thể push image lên Amazon ECR. Chọn **Next**. Trên màn hình tiếp theo, đặt tên role và chọn **Create role**.

![Adding Policy Permissions](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/figure4-scaled.jpg)
*Hình 4: Ví dụ về thêm policy permission*

4. Đi đến **Settings** trong **GitHub repository** của bạn, và dưới **Security** trong panel bên trái, chọn **Secrets and Variables**. Chọn tab **Actions** trong Secrets and Variables. Chọn **New repository secret**. Cho tên, nhập AWS_ROLE_ARN, nhập AWS Role ARN của role mà bạn đã tạo trong Bước 3, và chọn **Add secret**.

![GitHub Actions Secret for AWS Role](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/figure5.jpg)
*Hình 5: Ví dụ về tạo GitHub Actions secret cho AWS Role*

5. Tạo **New repository secret** khác cho **AWS_REGION**. Chỉ định Region mà bạn đã tạo tài nguyên của mình và chọn **Add secret**.

![GitHub Actions Secret for AWS Region](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/blog67.jpg)
*Hình 6: Ví dụ về GitHub Actions secret cho AWS Region*
### Chuẩn bị GitHub Actions workflow

GitHub Actions workflow là một quy trình tự động có thể cấu hình được tạo thành từ một hoặc nhiều job và bạn có thể định nghĩa các job này trong file YAML. Bạn sẽ tạo file YAML trong thư mục `.github/workflows` trong GitHub repository của bạn để định nghĩa workflow cho giải pháp. File YAML cho GitHub Actions workflow của bạn chứa các build job được chỉ định cho CodeBuild runner của bạn. Runner environment được chỉ định trong phần `runs-on` của file YAML, tham chiếu đến mỗi CodeBuild project mà bạn tạo cho giải pháp multi-arch của bạn.

- container-image.yaml

```yaml
name: Docker

on:
  workflow_dispatch: {}
  push:
    branches: [ "main" ]
    # Publish semver tags as releases.
    tags: [ 'v*.*.*' ]

env:
  REGISTRY: xxxxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com
  IMAGE_NAME: myapp

jobs:
  build:
    strategy:
      matrix:
        arch: [arm64, x86]
    runs-on: codebuild-myapp-${{ matrix.arch }}-${{ github.run_id }}-${{ github.run_attempt }}
    permissions:
      contents: read
      id-token: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@e3dd6a429d7300a6a4c196c26e071d42e0343502 # v4.0.2
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ secrets.AWS_REGION }}
          
      - name: Login to Amazon ECR Private
        if: github.event_name != 'pull_request'
        id: login-to-ecr
        uses: aws-actions/amazon-ecr-login@062b18b96a7aff071d4dc91bc00c4c1a7945b076 # v2.0.1
        with:
          registry-type: private

      # Extract metadata (tags, labels) for Docker
      # https://github.com/docker/metadata-action
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@8e5442c4ef9f78752691e2d8f8d19755c6f78e81 # v5.5.0
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      # Build (and optionally) push Docker image (don't push on PR)
      # https://github.com/docker/build-push-action
      - name: Build and push Docker image
        id: build-and-push
        uses: docker/build-push-action@2cdde995de11925a030ce8070c3d77a52ffcf1c0 # v5.3.0
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}-${{ matrix.arch }}
          labels: ${{ steps.meta.outputs.labels }}
  
  manifest:
    needs: build
    runs-on: codebuild-myapp-x86-${{ github.run_id }}-${{ github.run_attempt }}
    permissions:
      contents: read
      id-token: write
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@e3dd6a429d7300a6a4c196c26e071d42e0343502 # v4.0.2
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ secrets.AWS_REGION }}
          
      - name: Login to Amazon ECR Private
        if: github.event_name != 'pull_request'
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@062b18b96a7aff071d4dc91bc00c4c1a7945b076 # v2.0.1
        with:
          registry-type: private

      # Extract metadata (tags, labels) for Docker
      # https://github.com/docker/metadata-action
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@8e5442c4ef9f78752691e2d8f8d19755c6f78e81 # v5.5.0
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      # Build and push image manifest (don't push on PR)
      # https://github.com/Noelware/docker-manifest-action
      - name: Build and push Docker manifest
        id: build-and-push
        uses: Noelware/docker-manifest-action@master # v0.3.0
        with:
          images: ${{ steps.meta.outputs.tags }}-arm64,${{ steps.meta.outputs.tags }}-x86
          inputs: ${{ steps.meta.outputs.tags }}
          push: ${{ github.event_name != 'pull_request' }}
```

Trong phần `env variables` trong file cấu hình GitHub Actions workflow của bạn, bạn phải định nghĩa Amazon ECR repository mà bạn đã tạo để GitHub Actions job của bạn biết nơi push container manifest và image của bạn. Repository đã tạo của bạn được đặt trong `env variable` cho `IMAGE_NAME`.

```yaml
env:
  REGISTRY: xxxxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com
  IMAGE_NAME: <repository-name>
```

Trong GitHub Actions workflow YAML file của bạn, bạn phải định nghĩa cả hai CodeBuild project trong giá trị runs-on để định nghĩa kiến trúc tính toán nào mà hosted runner của bạn đang sử dụng để thực thi job của bạn. Ở đây chúng ta đã định nghĩa các biến arm64 và x86 trong trường `matrix`. Một job chạy cho mỗi kết hợp của các biến được định nghĩa trong trường `arch` trong matrix. Bạn phải chỉ định tên CodeBuild project làm prefix cho job của bạn để được CodeBuild runner nhận.

Giá trị runs-on của bạn trong file cấu hình nên như sau:
- runs-on: codebuild-<project-name>-${{ matrix.arch }}-${{ github.run_id }}-${{ github.run_attempt }}

```yaml
jobs:
  build:
    strategy:
      matrix:
        arch: [arm64, x86]
    runs-on: codebuild-myapp-${{ matrix.arch }}-${{ github.run_id }}-${{ github.run_attempt }}
```

Trong file cấu hình GitHub Actions workflow của bạn, các biến GitHub secrets được định nghĩa cho `AWS_ROLE_ARN` và `AWS_REGION`. GitHub pull các biến secret này khi chúng ta cần cấu hình credential của bạn để truy cập AWS.

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@e3dd6a429d7300a6a4c196c26e071d42e0343502 # v4.0.2
  with:
    role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
    aws-region: ${{ secrets.AWS_REGION }}
```

Sau khi tất cả các file được upload lên GitHub repository của bạn, repository của bạn sẽ trông tương tự như hình sau. Dockerfile và file index.html được đặt trong root của repository, và file container-image.yaml được đặt trong thư mục .github/workflows để định nghĩa GitHub Actions workflow của bạn.

![File Structure in GitHub Repository](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/blog11.jpg)
*Hình 7: Ví dụ về cấu trúc file trong GitHub repository*

Bên trong thư mục .github/workflows là nơi file container-image.yaml của bạn được lưu trữ.

![GitHub Actions Workflow YAML File](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/blog10.jpg)
*Hình 8: Ví dụ về GitHub Actions workflow YAML file*

## Kiểm thử giải pháp

Bây giờ tất cả tài nguyên của bạn đã được tạo trong GitHub và AWS, đã đến lúc kiểm thử giải pháp. Thay đổi message trong file index.html của bạn trong GitHub repository và commit các thay đổi của bạn vào main branch để GitHub Actions workflow của bạn có thể kích hoạt. Khi GitHub Actions workflow của bạn kích hoạt, CodeBuild runner của bạn sẽ bắt đầu thực thi các job mà chúng ta đã chỉ định trong file cấu hình của bạn.

1. Trong file index.html của bạn, thay đổi message trong body của text. Commit các thay đổi của bạn và push lên main branch.

2. Quay lại trang chủ của GitHub repository của bạn và chọn **Actions** trong panel trên cùng. Khi bạn chọn **Actions**, bạn sẽ thấy một trang mô tả các job đang chạy cho quá trình build cho cả hai kiến trúc tính toán như được hiển thị trong hình sau

![Build Jobs Starting](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/blog7.jpg)
*Hình 9: Build job bắt đầu cho cả hai kiến trúc tính toán*

3. Trong mỗi build job, bạn sẽ thấy các bước mà quá trình build đang hoàn thành cho cả hai kiến trúc tính toán. Nó nói rằng nó đang chờ runner nhận job này

![x86 Compute Runner](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/blog6.png)
*Hình 10: x86 compute runner*

![arm64 Compute Runner](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/blog4.png)
*Hình 11: arm64 compute runner*

4. Đi đến CodeBuild trong AWS Console, bạn sẽ thấy rằng cả hai build project của chúng ta cho các kiến trúc tính toán hiện đang In-Progress. Có thể mất vài phút để build hoàn thành.

![CodeBuild Projects for Both Architectures](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/blog2.jpg)
*Hình 12: CodeBuild project cho cả hai kiến trúc*

5. Nếu bạn quay lại trang **Actions** trong **GitHub repository** của bạn, thì bạn có thể thấy rằng cả hai build job đã hoàn thành cho kiến trúc tính toán x86 và arm64. Manifest job đang tạo manifest list chứa x86 và arm64 container image của bạn.

![Completed Builds in GitHub Actions](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/Blog1.jpg)
*Hình 13: Build hoàn thành trong trang GitHub Actions*

6. Nếu bạn vào Amazon ECR repository của bạn, thì bạn sẽ thấy rằng các image đã được cập nhật cho x86 và arm64 container image của bạn cho manifest của bạn.

![Completed Image Build in ECR](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/03/10/reallast.jpg)
*Hình 14: Build image hoàn thành trong ECR*

## Dọn dẹp tài nguyên

Để tránh phát sinh thêm chi phí, hãy đảm bảo bạn phá hủy tất cả hạ tầng mà bạn đã cung cấp liên quan đến các ví dụ được chi tiết trong bài viết này. Để dọn dẹp tài nguyên GitHub, bạn có thể xóa repository được sử dụng cho giải pháp này. Để dọn dẹp tài nguyên AWS, bạn có thể xóa hai CodeBuild project với các lệnh sau.

```bash
aws codebuild delete-project --name <project-name>-x86

aws codebuild delete-project --name <project-name>-arm64
```

Bạn có thể xóa Amazon ECR repository của bạn bằng cách sử dụng lệnh sau.

```bash
aws ecr delete-repository \
    --repository-name <repository-name> \
         --force
```

## Kết luận

Trong bài viết này, chúng tôi đã trình bày cách tích hợp GitHub Actions với AWS CodeBuild để build multi-arch image. Chúng tôi cũng đã hướng dẫn qua quy trình cách lưu trữ các multi-arch image này trong Amazon ECR để chúng có thể được truy xuất bởi x86 hoặc [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/)-AWS Graviton compute. CodeBuild cung cấp một [website](https://aws.amazon.com/codebuild/) chuyên dụng cung cấp thêm thông tin, tutorial, và tài nguyên được thiết kế để hỗ trợ người dùng trong việc khởi tạo triển khai runner để nâng cao pipeline của họ. Nếu bạn muốn biết thêm về Graviton compute, tham khảo [website](https://aws.amazon.com/pm/ec2-graviton/) của chúng tôi.

