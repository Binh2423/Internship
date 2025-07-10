# Tối ưu hóa triển khai đa môi trường với Amazon EKS Blueprints và CDK Pipelines

> **📖 Bài viết gốc**: [Streamline multi-environment deployments with Amazon EKS Blueprints and CDK pipelines](https://aws.amazon.com/blogs/containers/streamline-multi-environment-deployments-with-amazon-eks-blueprints-and-cdk-pipelines/)  
> **👤 Tác giả**: Elamaran Shanmugam, Sr. Specialist Partner Solutions Architect – Containers; Mikhail Shapirov, Principal Partner Solutions Architect – Industry Solutions; Jayaprakash Alawala, Principal Specialist Solutions Architect – Containers; Bhavye Sharma, Partner Solutions Architect  
> **📅 Ngày xuất bản**: 23 Tháng 5, 2025  
> **🌐 Nguồn**: AWS Containers Blog  
> **👨‍💻 Người dịch**: [Tên của bạn] - FCJ Intern  
> **📅 Ngày dịch**: 10 Tháng 7, 2025  

---

## 📋 Tóm tắt

Bài viết này hướng dẫn cách thiết lập pipeline tự động để triển khai và cập nhật hạ tầng Amazon EKS sử dụng CDK Pipelines module, một phần của Amazon EKS Blueprints for CDK. Giải pháp tập trung vào trường hợp sử dụng blue/green cluster upgrades, triển khai hai EKS cluster với các phiên bản khác nhau và sử dụng Amazon Route 53 để điều hướng traffic giữa các deployment. Điều này giúp tự động hóa việc quản lý EKS fleet trên nhiều môi trường, tài khoản và AWS Region.

**🎯 Đối tượng đọc**: DevOps Engineers, Platform Engineers, Solutions Architects  
**📊 Độ khó**: Intermediate to Advanced  
**🏷️ Tags**: Amazon EKS, CDK, CodePipeline, Blue/Green Deployment, GitOps

---

---

*Bài viết này được đồng tác giả bởi Elamaran Shanmugam, Sr. Specialist Partner Solutions Architect – Containers; Mikhail Shapirov, Principal Partner Solutions Architect – Industry Solutions; Jayaprakash Alawala, Principal Specialist Solutions Architect – Containers; Bhavye Sharma, Partner Solutions Architect*

Container đã cách mạng hóa việc phân phối phần mềm bằng cách cung cấp một môi trường portable và nhất quán giải quyết các thách thức liên quan đến framework phức tạp và dependencies. Người dùng đang tìm kiếm cách để tự động hóa việc triển khai và bảo trì các [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/) cluster của họ trên các phiên bản, môi trường, tài khoản, và [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) khác nhau. Việc triển khai các cluster này bao gồm các tác vụ như [tạo cluster](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html) với cấu hình networking và logging mong muốn, chọn [Amazon EKS add-on](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html), và khi sẵn sàng, triển khai các thành phần hạ tầng khác và công cụ vận hành Day 2.

Bài viết này cho thấy cách thiết lập pipeline tự động để triển khai và cập nhật hạ tầng Amazon EKS sử dụng [CDK pipelines module](https://aws-quickstart.github.io/cdk-eks-blueprints/pipelines/), một phần của [Amazon EKS Blueprints for CDK](https://aws-quickstart.github.io/cdk-eks-blueprints/). Quản lý Amazon EKS fleet là một chủ đề rất rộng và phức tạp bao gồm nhiều khía cạnh như quản lý lifecycle của cluster, add-on và application. Tuy nhiên, cho bài viết này chúng tôi đã chọn một trong những trường hợp sử dụng phổ biến, blue/green cluster upgrade, có thể áp dụng cho nhiều môi trường. Nó triển khai hai EKS cluster với phiên bản khác nhau cho blue và green deployment. Nó cũng triển khai một sample application [EchoServer](https://github.com/aws-samples/aws-cdk-pipelines-eks-cluster/tree/main/lib/application) trong cả hai EKS cluster và cho thấy cách điều hướng routing giữa blue/green deployment sử dụng [Amazon Route 53](https://aws.amazon.com/route53/).

## Kiến trúc giải pháp

Chúng ta tạo một sample [AWS CodePipeline](https://aws.amazon.com/codepipeline/) sử dụng [EKS Blueprints Pipelines](https://aws-quickstart.github.io/cdk-eks-blueprints/pipelines/) module giúp dễ dàng thiết lập continuous deployment pipeline cho các ứng dụng [AWS Cloud Development Kit (AWS CDK)](https://aws.amazon.com/cdk/) của bạn. Pipeline này tạo hai EKS cluster khác nhau, một trên phiên bản 1.30 và một khác trên 1.31, với một [managed node group](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html) cho mỗi cluster. Nó cũng triển khai các controller và operator như [AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html), [ExternalDNS](https://github.com/kubernetes-sigs/external-dns), [Metrics Server](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html), và [Cert Manager](https://cert-manager.io/).

Nó cũng bao gồm một stage để chuyển đổi người dùng của sample application sử dụng chiến lược [blue/green](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html) trên các cluster khác nhau. Giải pháp cần một Route 53 public domain hiện có (ví dụ example.org) như một yêu cầu tiên quyết. Bạn tạo hai subdomain (`blue.example.org` và `green.example.org`, hướng dẫn được cung cấp sau trong bài viết) trong Route 53 public domain. ExternalDNS controller, được triển khai trong mỗi EKS cluster, tạo một [CNAME](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/ResourceRecordTypes.html#CNAMEFormat) record trong subdomain tương ứng trỏ đến AWS Load Balancer endpoint liên kết với EchoServer application cụ thể.

Sơ đồ kiến trúc sau đây trình bày thiết kế tổng thể cho pipeline tự động hóa việc triển khai và quản lý traffic giữa blue và green deployment.

![Architecture Diagram](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/541.png)
*Hình 1: Sơ đồ kiến trúc cho CodePipeline sử dụng EKS Blueprints Pipeline*

### Các stage của CodePipeline:

- **Source**: Stage này lấy source của CDK app từ forked GitHub repo của bạn và kích hoạt pipeline mỗi khi bạn push commit mới vào nó.

- **Create EKS Blueprints**: Stage này sử dụng [AWS CodeBuild](https://aws.amazon.com/codebuild/) để compile code của bạn (nếu cần thiết) và thực hiện CDK synth. Output của bước đó là một cloud assembly, được sử dụng để thực hiện tất cả các hành động trong phần còn lại của pipeline. Bạn có thể định nghĩa [EKS blueprint framework](https://aws-quickstart.github.io/cdk-eks-blueprints/core-concepts/#blueprint) được triển khai trên các stage của pipeline. Blueprint định nghĩa một đặc tả của tất cả các thành phần được triển khai vào cluster như cấu hình cluster, managed node group, add-on, và application. Mặc dù blue/green deployment thường sử dụng blueprint giống hệt nhau cho cả hai môi trường, các tình huống khác có thể sử dụng cấu hình cơ sở chung với các tùy chỉnh cụ thể cho môi trường. Ví dụ, môi trường production thường cần sức mạnh tính toán và khả năng mở rộng nâng cao so với môi trường development.

```typescript
export default class MultiClusterBuilderConstruct {
    build(scope: Construct, id: string, account?: string, region?: string ) {
          ...
    create(scope: Construct, account?: string, region?: string ) {
        // Setup platform team
        const accountID = account ?? process.env.CDK_DEFAULT_ACCOUNT! ;
        const awsRegion =  region ?? process.env.CDK_DEFAULT_REGION! ;
        const parentDomain = blueprints.utils.valueFromContext(scope, "parent.hostedzone.name", "example.org");
        
        return blueprints.EksBlueprint.builder()
                    .account(accountID)
                    .region(awsRegion)
                    .resourceProvider(blueprints.GlobalResources.HostedZone, new blueprints.LookupHostedZoneProvider(parentDomain))
                    .addOns(
                        new blueprints.AwsLoadBalancerControllerAddOn(),
                        new blueprints.KubeStateMetricsAddOn(),
                        new blueprints.PrometheusNodeExporterAddOn(),
                        new blueprints.CertManagerAddOn(),
                        new blueprints.AdotCollectorAddOn(),
                        new blueprints.XrayAdotAddOn(),
                        new blueprints.ClusterAutoScalerAddOn(),
                        new blueprints.CalicoOperatorAddOn(),
                        new blueprints.ExternalDnsAddOn({
                                hostedZoneResources: [blueprints.GlobalResources.HostedZone],
                        })
                        new EchoServerApp(k8sProviderVersion, parentDomain),

                    )
    }
}
```

- **Pipeline Deploy IaC**: Stage này sử dụng [AWS CodeDeploy](https://aws.amazon.com/codedeploy/) để triển khai các CDK application của bạn trong hai Stack khác nhau mô tả các EKS cluster, cấu hình, và thành phần của bạn. Bạn có thể tạo pipeline sử dụng [CodePipeline](https://aws-quickstart.github.io/cdk-eks-blueprints/pipelines/). Ví dụ code pipeline sau đây cho thấy cách tạo các stage pipeline khác nhau sử dụng wave, ví dụ `eks-stage` và `dns-stage`.
```typescript
export class PipelineBlueGreenCluster {

    async buildAsync(scope: Construct) {
        ...
        const stagesEks : blueprints.StackStage[] = [];

        const blueprintBuilder = new MultiClusterBuilderConstruct().create(scope, accountID, region); 
        const blueprintBlue = blueprintBuilder
            .version(eks.KubernetesVersion.V1_30)
            .clusterProvider(new blueprints.MngClusterProvider());

            stagesEks.push({
                id: clusterANameSuffix+"-cluster",
                stackBuilder : blueprintBlue.clone(region),
            })

        const blueprintGreen = blueprintBuilder
            .version(eks.KubernetesVersion.V1_31)
            .clusterProvider(new blueprints.MngClusterProvider());
        
        stagesEks.push({
            id: clusterBNameSuffix+"-cluster",
            stackBuilder : blueprintGreen.clone(region),
        })
        const stagesDns : blueprints.StackStage[] = [];

        const prodEnv = clusterBNameSuffix;

        const dnsStackBuilder =  new DnsStackBuilderConstruct(prodEnv)
        stagesDns.push({
            id: `dns-${prodEnv}`,
            stackBuilder: dnsStackBuilder,
        });

        const gitOwner = 'aws-samples';
        const gitRepositoryName = 'cdk-eks-blueprints-patterns';

        blueprints.CodePipelineStack.builder()
            .application('npx ts-node bin/pipeline-bluegreen.ts')
            .name('blue-green-pipeline')
            .owner(gitOwner)
            .codeBuildPolicies(blueprints.DEFAULT_BUILD_POLICIES)
            .repository({
                repoUrl: gitRepositoryName,
                credentialsSecretName: 'cdk_blueprints_github_secret',
                targetRevision: 'main',
                trigger: blueprints.GitHubTrigger.POLL
            })
            .wave({
                id: "eks-stage",
                stages: stagesEks
            })
            .wave({
                id: "dns-stage",
                stages: stagesDns,
                props:{
                    pre:[
                        new ManualApprovalStep(`Promote-${prodEnv}-Environment`)
                       ]
                }
            })
            .build(scope, "blue-green-pipeline", {
                env: {
                    account: process.env.CDK_DEFAULT_ACCOUNT,
                    region: region,
                }
            });
    }
   
}
```

- **Manual approval**: Cần [manual approval](https://docs.aws.amazon.com/codepipeline/latest/userguide/approvals-action-add.html) để chuyển traffic từ blue sang green deployment hoặc ngược lại.

- **DNS switch**: Stage này cập nhật Route 53 record của bạn để trỏ đến cluster được chỉ định làm môi trường production trong code của bạn. Bạn có thể sử dụng [AWS CDK Route 53 module](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_route53-readme.html) để thêm CNAME record vào Route 53 parent public hosted zone.

```typescript
class DnsStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props: DnsStackProps) {
    super(scope, id, props);

    const hostZoneId = ssm.StringParameter.valueForStringParameter(
      this,
      "/eks-cdk-pipelines/hostZoneId"
    );

    const zoneName = ssm.StringParameter.valueForStringParameter(
      this,
      "/eks-cdk-pipelines/zoneName"
    );

    const zone = route53.HostedZone.fromHostedZoneAttributes(this, "appZone", {
      zoneName: zoneName,
      hostedZoneId: hostZoneId,
    });

    new route53.CnameRecord(this, "appCnameRecord", {
      zone: zone,
      recordName: "app",
      domainName: `echoserver.${props.envName}.${zoneName}`,
      ttl: cdk.Duration.seconds(30),
    });
  }
}
```

Sơ đồ kiến trúc sau đây cho thấy hạ tầng hoàn chỉnh được thiết lập cho pipeline này, sử dụng giải pháp được mô tả trong sơ đồ trước:

![EKS and Route 53 Infrastructure](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/542.png)
*Hình 2: Hạ tầng Amazon EKS và Route 53 được cung cấp thông qua CodePipeline*

## Yêu cầu tiên quyết

Các yêu cầu tiên quyết sau đây cần thiết để hoàn thành giải pháp này:

- Một [tài khoản AWS](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup)
- Một [tài khoản GitHub](https://github.com/signup)
- [GitHub Access Token](http://GitHub Access Token)
- Một [public hosted zone hiện có trong Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/AboutHZWorkingWith.html), hoặc bạn có thể [đăng ký domain mới với Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html) hoặc [sử dụng nó cho domain hiện có](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/MigratingDNS.html)
- [AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html#getting_started_install) phiên bản 2.151.0 hoặc mới hơn
- [AWS Command Line Interface (AWS CLI)](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [git](https://git-scm.com/docs)
- [npm](https://www.npmjs.com/) phiên bản 10.7.0 hoặc mới hơn

## Hướng dẫn triển khai

Ở mức độ cao, chúng ta sử dụng các bước sau để triển khai hạ tầng:

1. Fork sample repository.
2. Tạo các parameter và secret cụ thể cho môi trường.
3. Tạo [subdomain](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-routing-traffic-for-subdomains.html) cho mỗi cluster (ví dụ blue.example.org và green.example.org) trong Route 53.
4. Thay đổi fork của bạn và push các thay đổi.
5. Triển khai AWS CDK stack(s) của bạn.

### Tạo AWS Secrets Manager secret

Tạo GitHub personal access token (PAT) với scope **repo** và **admin:repo_hook** sử dụng [link](https://github.com/settings/tokens/new?scopes=repo,admin:repo_hook&description=eks-cdk-pipelines) sau hoặc hướng dẫn [từng bước](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token). Tạo một plain-text secret để giữ PAT token trong region mong muốn, và đặt tên của nó làm giá trị cho biến môi trường GITHUB_SECRET. Giá trị mặc định là `cdk_blueprints_github_secret`.

**CẢNH BÁO**: Khi chuyển đổi CDK giữa các AWS Region, hãy nhớ sao chép secret này.

```bash
export ACCOUNT_ID=$(aws sts get-caller-identity —output text —query Account)
export AWS_REGION="us-west-2"
export CDK_REPO_GITHUB_PAT_TOKEN=<set_token_here>
export CDK_REPO_AWS_SECRET_NAME="cdk_blueprints_github_secret"
aws secretsmanager create-secret —region $AWS_REGION \
--name $CDK_REPO_AWS_SECRET_NAME \
--description "GitHub Personal Access Token for CodePipeline to access GitHub account" \
--secret-string $CDK_REPO_GITHUB_PAT_TOKEN
```

### Thiết lập AWS Systems Manager Parameter Store

Giải pháp mong đợi Route 53 public hosted zone ID và name có sẵn trong [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html). Sao chép hosted zone ID và name từ Route 53 console, như được hiển thị trong hình sau.

![Route 53 Public Hosted Zone](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/543.png)
*Hình 3: Route 53 public hosted zone*

Lưu trữ hosted zone ID và name trong AWS System Manager Parameter Store bằng cách sử dụng các lệnh sau.

```bash
export HOSTED_ZONE_NAME="example.org" // thay đổi thành parent public hosted zone name của bạn
export HOSTED_ZONE_ID=$(aws route53 list-hosted-zones-by-name --dns-name $HOSTED_ZONE_NAME --query 'HostedZones[0].Id' --output text |  cut -d'/' -f3)

aws ssm put-parameter —name '/eks-cdk-pipelines/zoneName' —type String —value "$HOSTED_ZONE_NAME"
aws ssm put-parameter —name '/eks-cdk-pipelines/hostZoneId' —type String —value "$HOSTED_ZONE_ID"
```

### Tạo subdomain trong Route 53 parent hosted zone

Tạo một [subdomain](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-routing-traffic-for-subdomains.html#dns-routing-traffic-for-subdomains-new-hosted-zone) cho mỗi cluster (ví dụ blue.example.org và green.example.org) trong Route 53 parent hosted zone example.org. Bạn cũng tạo một subdomain cho mỗi cluster, để mỗi cluster application có domain name riêng và bạn có thể chuyển đổi traffic sử dụng pipeline. Cho ví dụ này, bạn tạo blue và green subdomain cho domain name mà bạn đã sử dụng trong bước trước.

1. Tạo subdomain cho blue.example.org.

```bash
export HOSTED_ZONE_NAME="example.org"
export SUB_DOMAIN_HOSTED_ZONE_NAME="blue.${HOSTED_ZONE_NAME}"
export RANDOM=$(date +"%s%3N")
aws route53 create-hosted-zone --name $SUB_DOMAIN_HOSTED_ZONE_NAME --caller-reference $RANDOM --hosted-zone-config Comment="command-line version",PrivateZone=false
```

2. Route 53 tự động gán name server khi bạn tạo hosted zone mới, như được hiển thị trong hình sau.

![Route 53 Subdomain Hosted Zone](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/545.png)
*Hình 4: Route 53 public hosted zone cho subdomain*

3. Bạn tạo NS record mới trong hosted zone cho parent domain của bạn (example.org), và bạn chỉ định bốn name server được tạo trong Bước 2.

```bash
export SUB_DOMAIN_HOSTED_ZONE_ID=$(aws route53 list-hosted-zones-by-name --dns-name $SUB_DOMAIN_HOSTED_ZONE_NAME --query 'HostedZones[0].Id' --output text  |  cut -d'/' -f3)
echo $SUB_DOMAIN_HOSTED_ZONE_ID

cat << EOF > route53_change_recordset.json
{
   "Changes":[
      {
         "Action": "CREATE",
         "ResourceRecordSet":{
            "Name": "${SUB_DOMAIN_HOSTED_ZONE_NAME}",
            "Type": "NS",
            "TTL": 300,
            "ResourceRecords": [
               {
                  "Value": "$(aws route53 list-resource-record-sets \
                  --hosted-zone-id $SUB_DOMAIN_HOSTED_ZONE_ID \
                  --query 'ResourceRecordSets[0].ResourceRecords[0]' \
                  --output text)"
               },
               {
                  "Value": "$(aws route53 list-resource-record-sets \
                  --hosted-zone-id $SUB_DOMAIN_HOSTED_ZONE_ID \
                  --query 'ResourceRecordSets[0].ResourceRecords[1]' \
                  --output text)"
               },
               {
                  "Value": "$(aws route53 list-resource-record-sets \
                  --hosted-zone-id $SUB_DOMAIN_HOSTED_ZONE_ID \
                  --query 'ResourceRecordSets[0].ResourceRecords[2]' \
                  --output text)"
               },
               {
                  "Value": "$(aws route53 list-resource-record-sets \
                  --hosted-zone-id $SUB_DOMAIN_HOSTED_ZONE_ID \
                  --query 'ResourceRecordSets[0].ResourceRecords[3]' \
                  --output text)"
               }
            ]
         }
      }
   ]
}
EOF

aws route53 change-resource-record-sets \
  --hosted-zone-id $HOSTED_ZONE_ID \
  --change-batch file://route53_change_recordset.json 
```

4. Lặp lại Bước 1–3 sử dụng green.example.org bằng cách đặt biến môi trường sau.

```bash
export SUB_DOMAIN_HOSTED_ZONE_NAME="green.${HOSTED_ZONE_NAME}"
```

### Triển khai giải pháp

1. Để bắt đầu, [fork sample repository](https://docs.github.com/en/get-started/quickstart/fork-a-repo#forking-a-repository) của chúng tôi và clone nó. Repository này chứa AWS CDK v2 code được viết bằng TypeScript.

```bash
git clone https://github.com/<YOUR-USERNAME>/cdk-eks-blueprints-patterns.git
cd cdk-eks-blueprints-patterns
npm i
```

2. Thực hiện các lệnh sau để bootstrap AWS environment.

```bash
cdk bootstrap aws://$ACCOUNT_ID/$AWS_REGION
```

3. Chỉnh sửa các file sau:

Thay đổi GitHub Repo Owner từ `aws-samples` thành GitHub Handle của bạn trong file `pipeline.ts`.

```bash
const gitOwner = 'aws-samples';
```

Thay đổi Parent hosted zone ID từ `example.org` thành public hosted zone của bạn trong file `multi-cluster-builder.ts`.

```bash
const parentDomain = blueprints.utils.valueFromContext(scope, "parent.hostedzone.name", "example.org");
```

4. Sau khi các thay đổi hoàn tất, commit và push các thay đổi vào repository của bạn sử dụng:

```bash
git add .
git commit -m "Update cluster configuration."
git push
```

5. Chạy lệnh sau từ root của repository này để triển khai pipeline stack:

```bash
make clean
make build
make list
make pattern pipeline-bluegreen deploy
```

Ban đầu, bạn phải triển khai pipeline của mình thủ công bằng cách sử dụng lệnh CDK deploy. Sau đó, mỗi thay đổi bạn push vào repository sẽ kích hoạt pipeline của bạn, pipeline sẽ tự cập nhật và thực thi. Lần thực thi đầu tiên của bạn mất một lúc, vì một số tài nguyên, như EKS cluster(s) và [managed node group](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html), có thể mất vài phút để sẵn sàng. Bạn theo dõi tiến trình bằng cách truy cập pipeline thông qua [AWS CodePipeline](https://us-west-2.console.aws.amazon.com/codesuite/codepipeline/pipelines?region=us-west-2).

Giải pháp này triển khai các thành phần sau:

**Hai EKS cluster**: Nó tạo hai EKS cluster một cho blue và một cho green environment sử dụng [Amazon EKS Blueprints for CDK](https://aws-quickstart.github.io/cdk-eks-blueprints/core-concepts/#blueprint). Mỗi cluster được triển khai với các thành phần sau:

- **AWSLoadBalancerController**: Quản lý cho Kubernetes cluster. Bạn có thể sử dụng controller để expose cluster app ra internet. Controller cung cấp AWS load balancer trỏ đến cluster Service hoặc Ingress resource. Thành phần này cần thiết để tạo [AWS Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) cho ingress traffic cho EchoServer application được triển khai trong cluster.

- **ExternalDNS**: ExternalDNS đồng bộ hóa các Kubernetes Service và Ingress được expose với DNS provider. ExternalDNS controller này trong mỗi cluster tạo Alias record cho ingress ALB trong Route 53 hosted zone/subdomain tương ứng.

- **EchoServer**: Cài đặt sample application để xác thực rằng các thành phần khác hoạt động đúng cách, như AWS Load Balancer Controller và [ExternalDNS](https://github.com/kubernetes-sigs/external-dns).

Đi đến CodePipeline console và đảm bảo rằng Pipeline được triển khai thành công, như được hiển thị trong các hình sau.

![CodePipeline Part 1](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/546.jpg)
*Hình 5: CodePipeline cho EKS Infra và DNS Switch – Phần 1*

![CodePipeline Part 2](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/548.jpg)
*Hình 6: CodePipeline cho EKS Infra và DNS Switch – Phần 2*

Nếu bạn kiểm tra [AWS CloudFormation stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html) của mình, thì bạn sẽ tìm thấy một stack cho pipeline (EksPipelineStack) và một stack (với [nested stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html)) cho mỗi EKS cluster, như được hiển thị trong hình sau.

![CloudFormation Stacks](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/564.jpg)
*Hình 7: CloudFormation stack được triển khai thông qua EKS Blueprints Pipeline*

## Truy cập EKS clusters

Trong output của EKSCluster Stack(s) của bạn, có các lệnh để thiết lập `kubeconfig` của bạn để truy cập cluster. Sao chép lệnh vào terminal và chạy nó để truy cập cluster sử dụng `kubectl`, như được hiển thị trong các hình sau.

![EKS Blue Cluster Output](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/565.png)
*Hình 8: CloudFormation stack output cho EKS Blue cluster*

![EKS Green Cluster Output](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/568.png)
*Hình 9: CloudFormation stack output cho EKS green cluster*

Bạn có thể truy cập application trực tiếp từ browser sử dụng URL `echoserver.example.org`

```bash
curl app.example.org
```

Cấu hình pipeline sử dụng biến `prodEnv` để chuyển routing đến target cluster cho EchoServer application. Khi bạn đặt nó và push vào fork của bạn, bạn phải approve thủ công thay đổi này trong CodePipeline, như được hiển thị trong hình sau.

![Manual Approval Stage](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/05/23/569.png)
*Hình 10: Manual approval stage trong CodePipeline cho DNS switching*

Sau khi nó hoàn thành việc cập nhật DNS record và được propagate, bạn có thể kiểm tra xem `app.example.org` record đang trỏ đến đâu. Để chuyển traffic đến green cluster sau khi nâng cấp lên 1.31, thay đổi biến `prodEnv` trong `pipeline.ts` và redeploy stack.

```bash
const prodEnv = clusterBNameSuffix; //thay đổi thành clusterANameSuffix
```

## Dọn dẹp tài nguyên

Bạn phải xóa các tài nguyên được cung cấp để tránh chi phí không mong muốn. Để dọn dẹp các blueprint resource được cung cấp, chạy lệnh sau:

```bash
make pattern pipeline-bluegreen destroy
```

Hơn nữa, bạn phải đi đến Amazon EKS console và xóa hai EKS cluster `blue-cluster-blueprint` và `green-cluster-blueprint` thủ công.

## Kết luận

Amazon EKS Blueprints Pipelines module cho phép bạn thiết lập continuous deployment pipeline cho các AWS CDK application của bạn. Trong bài viết này, chúng tôi đã cho thấy cách bạn có thể thiết lập continuous deployment pipeline cho các AWS CDK application của bạn sử dụng EKS Blueprints Pipelines module. Cách tiếp cận IaC này cung cấp các thay đổi thông qua một pipeline tự động và chuẩn hóa cũng được định nghĩa trong AWS CDK, cho phép bạn triển khai và nâng cấp cluster một cách nhất quán trên các phiên bản, môi trường, tài khoản, và AWS Region khác nhau trong khi theo dõi các thay đổi cluster và pipeline của bạn thông qua Git.

Sample code cung cấp một số ví dụ về các thành phần thường được cài đặt trong EKS cluster, như [AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html), [ExternalDNS](https://github.com/kubernetes-sigs/external-dns), và [Metrics Server](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html).

Demonstration này có thể được sử dụng làm điểm khởi đầu để xây dựng giải pháp riêng của bạn để tự động hóa việc triển khai EKS cluster(s) của riêng bạn. Truy cập tài liệu [Amazon EKS Blueprints Quick Start](https://aws-quickstart.github.io/cdk-eks-blueprints/) để biết thêm thông tin về việc sử dụng các thư viện này. Chúng tôi khuyến khích bạn sử dụng [EKS Blueprints Pipelines](https://aws-quickstart.github.io/cdk-eks-blueprints/pipelines/) module cho workload của bạn.

---

