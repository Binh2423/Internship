# Worklog - Ngày 15/05/2025

## 📅 Thông tin cơ bản
- **Ngày**: 15/05/2025
- **Thứ**: Thứ Năm
- **Tuần thực tập**: Tuần thứ 1/12
- **Thời gian làm việc**: 8:00 - 17:00
- **Mood**: 😊 + Thích thú với cloud development environment

## 🎯 Mục tiêu ngày hôm nay
- [x] Thiết lập AWS Cloud9 development environment
- [x] Tìm hiểu về Amazon S3 và static website hosting
- [x] Deploy first static website lên S3

## 💼 Công việc đã thực hiện

### 1. Cloud Development with AWS Cloud9 ⏱️ 3 giờ
- **Mô tả**: Tạo Cloud9 environment, explore IDE features, setup development workflow
- **Kết quả**: Cloud9 environment ready for development, integrated với AWS services
- **Tools/Tech**: AWS Cloud9, EC2 (underlying), IAM permissions
- **Links**: Cloud9 environment setup, Development workflow documentation

### 2. Static Website Hosting with Amazon S3 ⏱️ 4 giờ
- **Mô tả**: Tạo S3 bucket, configure static website hosting, upload HTML/CSS files
- **Kết quả**: Static website deployed và accessible via S3 website endpoint
- **Tools/Tech**: Amazon S3, S3 Static Website Hosting, HTML/CSS/JavaScript
- **Links**: Website URL, S3 bucket configuration

## 📚 Kiến thức học được

### 🔧 Technical Skills
- **AWS Services**: Cloud9, S3 (Static Website Hosting, Bucket Policies)
- **Web Development**: HTML, CSS, JavaScript basics
- **Development Tools**: Cloud-based IDE, integrated terminal

### 💡 Concepts & Theory
- **New Concepts**: Object storage vs block storage, S3 storage classes
- **Best Practices**: S3 bucket naming, website hosting configuration
- **Industry Knowledge**: Cloud-based development environments

### 🤝 Soft Skills
- **Creativity**: Designing simple but effective website
- **Project Management**: Planning website structure
- **Problem Solving**: Debugging website hosting issues

## 🚧 Khó khăn và giải pháp

### Vấn đề 1: Cloud9 Environment Slow Performance
- **Mô tả**: Cloud9 environment chạy chậm, lag khi typing
- **Impact**: Giảm productivity, frustrating experience
- **Root Cause**: t2.micro instance không đủ resources cho development
- **Solution**: Upgrade to t3.small instance type
- **Result**: Performance improved significantly
- **Lesson**: Choose appropriate instance size for workload

### Vấn đề 2: S3 Website Access Denied
- **Mô tả**: Website không accessible sau khi enable static hosting
- **Impact**: Mất 1 giờ để troubleshoot
- **Root Cause**: Bucket policy không allow public read access
- **Solution**: Configure proper bucket policy for public website access
- **Result**: Website accessible publicly
- **Lesson**: S3 permissions can be tricky, cần hiểu bucket policies

## 💭 Reflection & Insights

### What went well today?
- Successfully deployed first static website to AWS
- Hiểu được power của cloud-based development
- Nắm được S3 basics và website hosting

### What could be improved?
- Cần học thêm về S3 advanced features
- Nên practice thêm web development skills
- Cần hiểu rõ hơn về S3 security

### Key Insights
- Cloud9 provides powerful development environment
- S3 is more than just storage - it's a platform
- Static websites can be very cost-effective on AWS

---

*Worklog created by: Kieu Nguyen Thanh Binh*  
*Next review: 16/05/2025*
