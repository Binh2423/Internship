# Tìm hiểu sâu: Amazon EKS Dashboard để có tầm nhìn tổng quan về vận hành và quản trị đa cluster

> **📖 Bài viết gốc**: [Deep Dive: Amazon EKS Dashboard for Visibility into Multi-Cluster Operations and Governance](https://aws.amazon.com/blogs/containers/deep-dive-amazon-eks-dashboard-for-visibility-into-multi-cluster-operations-and-governance/)  
> **👤 Tác giả**: Carlos Santana, Sr. Solution Architect, Containers; Sriram Ranganathan, Sr. Product Manager, Kubernetes; Sabari Sawant, Product Marketing Manager, Kubernetes; và Frank Carta, Sr. GTM specialist, Containers  
> **📅 Ngày xuất bản**: 03 Tháng 6, 2025  
> **🌐 Nguồn**: AWS Containers Blog  
> **👨‍💻 Người dịch**: Kiều Nguyễn Thanh Bình - FCJ Intern  


---

## 📋 Tóm tắt

Bài viết này giới thiệu Amazon EKS Dashboard - một tính năng mới trong AWS Console giúp các tổ chức có tầm nhìn tổng quan về các Kubernetes cluster trên nhiều AWS Region và tài khoản. Dashboard cung cấp khả năng giám sát trạng thái cluster, theo dõi phiên bản Kubernetes, dự báo chi phí extended support, và quản lý add-ons một cách tập trung. Điều này giúp giải quyết các thách thức về cluster sprawl, rủi ro bảo mật, và tối ưu hóa chi phí vận hành.

**🎯 Đối tượng đọc**: Platform Engineers, Cluster Administrators, Cloud Architects  
**📊 Độ khó**: Intermediate  
**🏷️ Tags**: Amazon EKS, Kubernetes, Multi-cluster Management, Dashboard, Governance

---

---

*Bài viết này được đồng tác giả bởi Carlos Santana, Sr. Solution Architect, Containers; Sriram Ranganathan, Sr. Product Manager, Kubernetes; Sabari Sawant, Product Marketing Manager, Kubernetes; và Frank Carta, Sr. GTM specialist, Containers.*

Khi các tổ chức mở rộng hạ tầng Kubernetes của họ trên các [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) và tài khoản, họ đối mặt với những thách thức ngày càng tăng trong việc duy trì sự giám sát các Kubernetes cluster của mình. Nếu không có tầm nhìn tập trung, các team thường phát hiện ra các cluster đang chạy phần mềm lỗi thời, bỏ lỡ các bản cập nhật bảo mật, thực hiện các nâng cấp không có kế hoạch, và gặp phải các chi phí hỗ trợ mở rộng bất ngờ. [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/) Dashboard giải quyết những thách thức này bằng cách cung cấp một giao diện tập trung cho phép các kiến trúc sư đám mây và quản trị viên cluster duy trì tầm nhìn toàn tổ chức trên các Kubernetes cluster của họ.

## Thách thức về tầm nhìn trong quản lý Kubernetes đa cluster

Khi các tổ chức mở rộng footprint Kubernetes của họ để đáp ứng các nhu cầu kinh doanh đa dạng—chẳng hạn như cải thiện tính khả dụng, giảm độ trễ, thực thi cách ly workload, hoặc đảm bảo data residency—họ thường triển khai nhiều EKS cluster trên các AWS Region và tài khoản. Mặc dù mô hình phân tán này mang lại lợi ích về kiến trúc và vận hành, nó cũng giới thiệu một lớp thách thức mới có thể cản trở governance và hiệu quả vận hành ở quy mô lớn:

- **Cluster sprawl**: Không có kiểm soát tập trung, các cluster phát triển nhanh chóng trên các team, tài khoản, và AWS Region. Các platform team thường mất dấu inventory hạ tầng Kubernetes, dẫn đến các môi trường không được quản lý và thiếu governance.

- **Rủi ro bảo mật**: Các cluster có thể chạy trên các phiên bản Kubernetes không được hỗ trợ hoặc chứa các add-on lỗi thời. Những lỗ hổng này làm lộ môi trường với các rủi ro bảo mật và vi phạm tuân thủ, trong khi việc thiếu theo dõi toàn tổ chức làm cho việc khắc phục kịp thời trở nên khó khăn.

- **Tính không hiệu quả trong vận hành**: Phối hợp thủ công các nâng cấp và bảo trì trên các cluster làm chậm các hoạt động. Các team thiếu tầm nhìn trung tâm về upgrade readiness insights, dẫn đến tính không hiệu quả trong việc nâng cấp cluster.

- **Chi phí hỗ trợ tăng**: Các cluster vượt quá lifecycle hỗ trợ của chúng tích lũy thêm các khoản phí hỗ trợ mở rộng. Không có dự báo chi phí hoặc tầm nhìn về version lag, các tổ chức đối mặt với các chi phí có thể tránh được và những bất ngờ về ngân sách.

Những thách thức này nhấn mạnh nhu cầu về một cái nhìn toàn diện, toàn tổ chức vào hạ tầng Kubernetes—một khả năng mà Amazon EKS Dashboard được xây dựng có mục đích để cung cấp.

## Giới thiệu Amazon EKS Dashboard

Để giải quyết các thách thức về tầm nhìn, tuân thủ, và vận hành được giới thiệu bởi các môi trường Kubernetes phân tán, Amazon EKS hiện bao gồm một trải nghiệm dashboard tập trung. Tính năng AWS console native này cung cấp một cái nhìn thống nhất về các tài nguyên Kubernetes—cluster, managed node group, và Amazon EKS add-on—trên tất cả các tài khoản AWS và Region trong tổ chức của bạn. Amazon EKS Dashboard cho phép các platform engineer và cluster administrator giám sát trạng thái cluster và trạng thái phiên bản Kubernetes, xác định các cluster được lên lịch cho end-of-support auto-upgrade, dự báo tác động chi phí cho các cluster sử dụng extended support, và xác định các node group và add-on cần cập nhật phiên bản.

Việc hợp nhất metadata như phân phối phiên bản, loại node, lifecycle hỗ trợ, và cấu hình cho phép Amazon EKS Dashboard giảm thiểu nhu cầu về công cụ tùy chỉnh hoặc giải pháp bên thứ ba để có tầm nhìn trên tất cả các Kubernetes cluster của bạn. Giao diện toàn diện này cho phép các platform team vận hành với sự tự tin và rõ ràng, đảm bảo governance chủ động trên các Kubernetes cluster của họ. Giao diện này cung cấp chức năng để:

- Giám sát trạng thái cluster và trạng thái phiên bản Kubernetes
- Xác định các cluster đang tiến gần đến end of support auto upgrade
- Dự báo tác động tài chính của việc chạy các phiên bản cluster cũ hơn
- Tối ưu hóa theo dõi inventory mà không cần công cụ bên thứ ba hoặc script tùy chỉnh

Dashboard có sẵn như một tính năng Amazon EKS console native, đảm bảo trải nghiệm người dùng liền mạch và loại bỏ gánh nặng bảo trì công cụ bên ngoài, như được hiển thị trong hình sau.

![Amazon EKS Dashboard Overview](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/muli-cluster-dashboard-hero.png)

## Bắt đầu với Amazon EKS Dashboard

Người dùng có thể truy cập Dashboard trong Amazon EKS console thông qua [AWS management](https://docs.aws.amazon.com/organizations/latest/userguide/orgs-manage_accounts_management.html) và [delegated administrator](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_integrate_delegated_admin.html) account. Quy trình thiết lập trực tiếp và bao gồm việc kích hoạt trusted access như một thiết lập một lần trong trang cài đặt Dashboard của Amazon EKS console, như được hiển thị trong hình sau. Việc kích hoạt trusted access cho phép management account xem Dashboard trong Amazon EKS console. Để biết thêm thông tin về thiết lập và cấu hình, các tổ chức có thể tham khảo [tài liệu AWS chính thức](https://docs.aws.amazon.com/eks/latest/userguide/cluster-dashboard.html).

![Dashboard Settings](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/13/figure-2-fixed.png)
*Hình 2: Cài đặt Dashboard*

Đối với các tổ chức muốn sử dụng Amazon EKS Dashboard từ một tài khoản không phải management, bất kỳ member account nào trong Organizations của họ có thể được đăng ký làm delegated administrator account cho Amazon EKS để xem dashboard. Thông qua trang cài đặt dashboard trong management account, các tổ chức có thể đăng ký bất kỳ member account nào trong Organizations làm delegated administrator cho Amazon EKS, như được hiển thị trong hình sau. Khi việc đăng ký hoàn tất, người dùng có thể truy cập Dashboard bằng cách đăng nhập vào delegated administrator account và điều hướng đến liên kết **Dashboard** trên thanh bên trái của Amazon EKS console.

![Register Delegated Administrator](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture4-1.jpg)
*Hình 3: Đăng ký delegated administrator*

![Provide Organizations Member Account](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture5.jpg)
*Hình 4: Cung cấp bất kỳ Organizations member account nào làm delegated administrator cho Amazon EKS*

![Access Dashboard Link](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture7.jpg)
*Hình 5: Truy cập Dashboard sử dụng liên kết trong Amazon EKS console*

Amazon EKS multi-cluster Dashboard cung cấp tầm nhìn vào ba loại tài nguyên chính:

**Cluster**: Xem thông tin tổng hợp về EKS cluster như:
- Cluster với upgrade insights
- Phân phối cluster dựa trên loại hỗ trợ
- Phân tích cluster theo phiên bản Kubernetes

**Managed node group**: Xem thông tin tổng hợp về managed node group như:
- Node group theo loại [Amazon Machine Image (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html), chẳng hạn như [Amazon Linux](https://aws.amazon.com/amazon-linux-2/) hoặc [Bottlerocket](https://aws.amazon.com/bottlerocket/)
- Cấu hình auto-repair của node group
- Phân phối loại instance

**Amazon EKS add-on**: Xem thông tin tổng hợp về Amazon EKS add-on như:
- Số lượng cài đặt mỗi add-on
- Phân phối phiên bản mỗi add-on
- Add-on có vấn đề về sức khỏe
## Các trường hợp sử dụng Amazon EKS Dashboard

Trong các phần sau, chúng ta khám phá một số tình huống vận hành phổ biến nơi Amazon EKS Dashboard cho phép một số tình huống vận hành.

### Trường hợp sử dụng 1: Lifecycle phiên bản và dự báo chi phí

Cluster administrator có thể sử dụng dashboard để xác định các cluster được đăng ký trong extended support. Visualization sau đây trình bày sự phân tích các cluster trong tổ chức dựa trên cấu hình loại hỗ trợ của họ giữa standard và extended support.

![Clusters by Support Type](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/image-117.png)
*Hình 6: Dashboard widget của Cluster theo loại hỗ trợ*

Người dùng cũng có thể visualize các cluster đang tiến gần đến end-of-support auto-upgrade được nhóm theo phiên bản Kubernetes cluster. Một menu dropdown linh hoạt cho phép người dùng chọn các khoảng thời gian khác nhau, chẳng hạn như 30 ngày tới hoặc các khoảng thời gian được xác định trước khác, như được hiển thị trong hình sau. Các team có thể xem cluster nào được thiết lập cho end-of-support auto-upgrade, cho phép theo dõi chủ động và lập kế hoạch cho các nâng cấp Kubernetes cluster trong khi giảm thiểu gián đoạn hạ tầng.

![Clusters Scheduled for Auto Upgrade](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture9.jpg)
*Hình 7: Chế độ xem có thể lọc hiển thị các cluster được lên lịch cho auto upgrade trong các khung thời gian đã chọn*

Ngoài quản lý phiên bản, các tổ chức cũng có thể sử dụng dashboard để dự báo chi phí tiềm năng liên quan đến EKS cluster khi các cluster không được nâng cấp trong các timeline được khuyến nghị, như được hiển thị trong hình sau. Một widget theo dõi giám sát số ngày mà các cluster vẫn ở trên các phiên bản cũ hơn và cung cấp ước tính rõ ràng về tác động tài chính của việc trì hoãn nâng cấp. Điều này cho phép các tổ chức điều chỉnh bảo trì hạ tầng với kế hoạch tài chính, đảm bảo cả khả năng phục hồi vận hành và hiệu quả chi phí.

![Cost Forecasting Widget](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture10.jpg)
*Hình 8: Widget dự báo chi phí ước tính các khoản phí extended support*

Mặc dù theo dõi các phiên bản Kubernetes và dự báo chi phí extended support cung cấp giám sát chiến lược cho tổ chức của bạn, đảm bảo sức khỏe vận hành của các cluster và node group của bạn cũng quan trọng không kém để duy trì một hạ tầng mạnh mẽ.

### Trường hợp sử dụng 2: Sức khỏe cluster và node group

**2.1 Sẵn sàng nâng cấp:** Cluster administrator sử dụng Dashboard để lọc và hiển thị các cluster có vấn đề nghiêm trọng về upgrade insight, như được hiển thị trong hình sau. Dashboard trình bày một chế độ xem sẵn sàng nâng cấp làm nổi bật các cluster dựa trên số lượng và mức độ nghiêm trọng của upgrade insight. Những insight này đánh dấu các incompatibility tiềm năng hoặc cấu hình có thể can thiệp vào một nâng cấp phiên bản thành công. Sử dụng dữ liệu này cho phép administrator giải quyết chủ động các vấn đề trước khi khởi tạo nâng cấp, do đó đảm bảo chuyển đổi mượt mà hơn, giảm thiểu gián đoạn, và duy trì tính toàn vẹn hệ thống trong suốt lifecycle nâng cấp.

![Upgrade Insights Severity Issues](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture11.jpg)
*Hình 9: Chế độ xem tài nguyên hiển thị các vấn đề nghiêm trọng upgrade insight mỗi cluster*

**2.2 Sức khỏe node group**: Mặc dù sẵn sàng nâng cấp tập trung vào sức khỏe cấp cluster, không kém phần quan trọng là sức khỏe của các node group riêng lẻ. Cluster administrator phân tích cấu hình node group trên các tài khoản và AWS Region để phát hiện những cái có auto-repair bị vô hiệu hóa. Dashboard trình bày một phân tích trực quan làm nổi bật có bao nhiêu node group có tính năng quan trọng này bị tắt, như được hiển thị trong hình sau. Auto-repair giảm thiểu downtime bằng cách tự động thay thế các node không khỏe mạnh, đảm bảo rằng các dịch vụ vẫn có sẵn trong các lỗi instance. Xác định và khắc phục các node group mà không có auto-repair được kích hoạt cho phép các tổ chức tăng cường khả năng chịu lỗi, hỗ trợ các mục tiêu tính khả dụng cao, và giảm can thiệp thủ công trong các tình huống phục hồi.

![Auto Repair Disabled](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/13/figure10_ocr_blurred.png)
*Hình 10: Chế độ xem tài nguyên được lọc theo auto repair bị vô hiệu hóa*

Ngoài việc duy trì sức khỏe cluster và node group thông qua các tính năng như auto-repair và kiểm tra sẵn sàng nâng cấp, các tổ chức phải đảm bảo quản lý nhất quán các Amazon EKS add-on của họ trên toàn bộ hạ tầng để duy trì các tiêu chuẩn bảo mật và tuân thủ.

### Trường hợp sử dụng 3: Governance Amazon EKS Add-on

Các team sử dụng dashboard để tối ưu hóa quản lý Amazon EKS add-on trên nhiều tài khoản AWS và Region thông qua một chế độ xem phân phối phiên bản tập trung, như được hiển thị trong hình sau. Administrator chọn các add-on cụ thể để xem phân phối phiên bản của họ, cho phép xác định hiệu quả các cluster cần nâng cấp. Khả năng này đặc biệt có giá trị khi giải quyết các lỗ hổng bảo mật hoặc các yêu cầu tuân thủ. Tính năng **view resources** cho phép các team xác định chính xác các cluster chạy các phiên bản add-on lỗi thời, do đó tạo điều kiện cho việc lập kế hoạch bảo trì có mục tiêu. Tầm nhìn tập trung này cho phép các tổ chức duy trì các phiên bản add-on nhất quán trên các cluster của họ.

![EKS Add-ons Version Distribution](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture13.jpg)
*Hình 11: Phân phối phiên bản Amazon EKS add-on*

Với giám sát vận hành, quản lý sức khỏe, và governance Add-on được thiết lập thông qua dashboard, các tổ chức có thể sử dụng khả năng báo cáo toàn diện để có được insight sâu hơn vào hạ tầng Kubernetes của họ trên các AWS Region và tài khoản. Điều này cho phép ra quyết định dựa trên dữ liệu và xác minh tuân thủ.

![EKS Add-ons Resource View](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/image-2025-06-03T101319.497.png)
*Hình 12: Chế độ xem tài nguyên cho Amazon EKS add-on được lọc theo tên và phiên bản*
### Trường hợp sử dụng 4: Báo cáo và insight mở rộng

**4.1 Xuất dữ liệu và tích hợp:** Các team sử dụng chức năng xuất CSV để trích xuất cả bộ dữ liệu hoàn chỉnh và được lọc từ dashboard. Các tổ chức sử dụng tính năng xuất này cho nhiều mục đích, chẳng hạn như tải lên các dịch vụ lưu trữ như [Amazon S3](https://aws.amazon.com/s3/), chia sẻ thông tin inventory với auditor, và tích hợp với các nền tảng báo cáo tùy chỉnh. Tính linh hoạt này cho phép các team mở rộng cluster và insight hạ tầng ngoài giao diện dashboard để phân tích nâng cao, như được hiển thị trong hình sau.

![Export to CSV](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/13/figure13_selective_blur.png)
*Hình 13: Chọn Export to csv để tải xuống file csv*

**4.2 Chế độ xem tuân thủ tài nguyên AWS Regional:** Để hoàn thành bức tranh phân tích, các team truy cập chế độ xem bản đồ toàn cầu để giám sát phân phối tài nguyên trên các AWS Region cho các yêu cầu tuân thủ và quy định. Dashboard cho phép truy cập một cú nhấp chuột vào các chế độ xem được lọc của tài nguyên, cho phép các tổ chức xác minh rằng các cluster trong các AWS Region cụ thể đáp ứng các yêu cầu quy định kinh doanh, như được hiển thị trong hình sau.

![Global View by Region](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture20.jpg)
*Hình 14: Chế độ xem toàn cầu hiển thị cluster mỗi AWS Region, chọn một Region hiển thị số lượng cluster*

**4.3 Phân tích xu hướng lịch sử:** Các platform team theo dõi các xu hướng hạ tầng Kubernetes quan trọng thông qua visualization dữ liệu lịch sử. Dashboard hiển thị tăng trưởng inventory cluster, việc áp dụng phiên bản Kubernetes, và các mẫu đăng ký loại hỗ trợ theo thời gian, như được hiển thị trong hình sau. Tầm nhìn này cho phép các team đưa ra quyết định dựa trên dữ liệu trong khi giám sát sự tiến triển hệ sinh thái Kubernetes của tổ chức họ.

![Historical Trend Analysis](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2025/06/03/Picture21.jpg)
*Hình 15: Xu hướng lịch sử*

### Hãy thử Amazon EKS Dashboard!

Amazon EKS Dashboard hoạt động từ AWS Region us-east-1 (N. Virginia), tổng hợp metadata EKS cluster từ tất cả các AWS Region thương mại mà không tốn thêm chi phí. Các team có thể bắt đầu visualize các Kubernetes cluster của họ bằng cách truy cập Amazon EKS console và kích hoạt trải nghiệm dashboard mới.

### Kiểm tra roadmap container của chúng tôi!

Nếu bạn có ý tưởng về cách chúng tôi có thể cải thiện Amazon EKS Dashboard hoặc các khía cạnh khác của các dịch vụ container của chúng tôi, thì vui lòng sử dụng [containers roadmap](https://github.com/aws/containers-roadmap/projects/1) của chúng tôi để cung cấp phản hồi và xem xét các mục roadmap hiện có của chúng tôi.

## Kết luận

Amazon EKS Dashboard giải quyết một thách thức quan trọng trong quản lý Kubernetes ở quy mô lớn: thiếu tầm nhìn tập trung trên các cluster phân tán. Bằng cách cung cấp một giao diện thống nhất để giám sát trạng thái cluster, theo dõi phiên bản, dự báo chi phí, và quản lý add-on, dashboard này cho phép các tổ chức duy trì governance chủ động và tối ưu hóa hiệu quả vận hành.

Các tính năng chính bao gồm khả năng theo dõi upgrade readiness, xác định các cluster cần auto-repair, và có được insight về phân phối tài nguyên trên các AWS Region. Điều này đặc biệt có giá trị cho các platform team và cluster administrator cần duy trì oversight trên hạ tầng Kubernetes phức tạp.

---

## 📖 Glossary - Thuật ngữ

| English | Tiếng Việt | Định nghĩa |
|---------|------------|------------|
| **Amazon EKS** | Amazon EKS | Amazon Elastic Kubernetes Service - dịch vụ Kubernetes được quản lý của AWS |
| **Cluster sprawl** | Sự lan tràn cluster | Hiện tượng các cluster phát triển không kiểm soát trên nhiều môi trường |
| **Dashboard** | Dashboard / Bảng điều khiển | Giao diện tập trung để giám sát và quản lý |
| **Delegated administrator** | Quản trị viên được ủy quyền | Tài khoản được cấp quyền quản trị thay mặt cho management account |
| **Extended support** | Hỗ trợ mở rộng | Gói hỗ trợ có phí cho các phiên bản Kubernetes cũ |
| **Governance** | Quản trị | Việc thiết lập và thực thi các chính sách, quy trình quản lý |
| **Managed node groups** | Nhóm node được quản lý | Các nhóm EC2 instance được AWS quản lý cho EKS cluster |
| **Multi-cluster** | Đa cluster | Kiến trúc sử dụng nhiều Kubernetes cluster |
| **Node group auto-repair** | Tự động sửa chữa nhóm node | Tính năng tự động thay thế các node bị lỗi |
| **Trusted access** | Truy cập tin cậy | Cơ chế cho phép AWS service truy cập tài nguyên Organizations |
| **Upgrade insights** | Thông tin chi tiết nâng cấp | Phân tích về khả năng và rủi ro khi nâng cấp cluster |
| **Version lifecycle** | Vòng đời phiên bản | Quá trình hỗ trợ từ khi phát hành đến khi ngừng hỗ trợ |

## 🔗 Tài liệu tham khảo

### Tài liệu gốc
- [Deep Dive: Amazon EKS Dashboard for Visibility into Multi-Cluster Operations and Governance](https://aws.amazon.com/blogs/containers/deep-dive-amazon-eks-dashboard-for-visibility-into-multi-cluster-operations-and-governance/): Bài viết gốc
- [AWS EKS Dashboard Documentation](https://docs.aws.amazon.com/eks/latest/userguide/cluster-dashboard.html): Tài liệu chính thức về EKS Dashboard
- [AWS Organizations Documentation](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html): Hướng dẫn về AWS Organizations

### Tài liệu tiếng Việt
- [AWS Documentation VN](https://docs.aws.amazon.com/): Tài liệu AWS đa ngôn ngữ
- [AWS EKS User Guide](https://docs.aws.amazon.com/eks/latest/userguide/): Hướng dẫn sử dụng Amazon EKS
- [AWS Best Practices](https://aws.amazon.com/architecture/well-architected/): Các thực hành tốt nhất trên AWS

### Tools và Services
- [Amazon EKS](https://aws.amazon.com/eks/): Dịch vụ Kubernetes được quản lý
- [AWS Organizations](https://aws.amazon.com/organizations/): Quản lý tài khoản AWS tập trung
- [Amazon S3](https://aws.amazon.com/s3/): Dịch vụ lưu trữ đối tượng
- [AWS Containers Roadmap](https://github.com/aws/containers-roadmap/projects/1): Lộ trình phát triển dịch vụ container

---


