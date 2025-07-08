# Worklog - Ngày 14/05/2025

## 📅 Thông tin cơ bản
- **Ngày**: 14/05/2025
- **Thứ**: Thứ Tư
- **Tuần thực tập**: Tuần thứ 1/12
- **Thời gian làm việc**: 8:00 - 17:00
- **Mood**: 😊 + Excited về hands-on với EC2

## 🎯 Mục tiêu ngày hôm nay
- [x] Tìm hiểu về Amazon EC2 và launch first instance
- [x] Thiết lập IAM Roles for EC2
- [x] Thực hành connect và manage EC2 instances

## 💼 Công việc đã thực hiện

### 1. Compute Essentials with Amazon EC2 ⏱️ 4 giờ
- **Mô tả**: Học về EC2 instance types, AMIs, security groups. Launch và configure EC2 instance
- **Kết quả**: Successfully launched t2.micro instance, configured security groups, connected via SSH
- **Tools/Tech**: Amazon EC2, Security Groups, Key Pairs, SSH
- **Links**: EC2 instance configuration, Security group rules

### 2. Instance Profiling with IAM Roles for EC2 ⏱️ 3 giờ
- **Mô tả**: Tạo IAM roles cho EC2, attach policies, sử dụng instance profiles
- **Kết quả**: EC2 instance có thể access S3 bucket thông qua IAM role
- **Tools/Tech**: IAM Roles, Instance Profiles, AWS CLI on EC2
- **Links**: IAM role configuration, EC2 instance profile setup

## 📚 Kiến thức học được

### 🔧 Technical Skills
- **AWS Services**: EC2 (Instances, AMIs, Security Groups), IAM Roles, Instance Profiles
- **Linux Administration**: SSH connection, basic Linux commands
- **Security**: Security groups vs NACLs, Key pair management

### 💡 Concepts & Theory
- **New Concepts**: EC2 instance lifecycle, Instance metadata service
- **Best Practices**: Security group configuration, IAM roles for services
- **Industry Knowledge**: Infrastructure as Code concepts

### 🤝 Soft Skills
- **Problem Solving**: Troubleshooting SSH connection issues
- **System Administration**: Managing cloud instances
- **Security Mindset**: Applying least privilege to EC2 access

## 🚧 Khó khăn và giải pháp

### Vấn đề 1: SSH Connection Timeout
- **Mô tả**: Không thể SSH vào EC2 instance sau khi launch
- **Impact**: Mất 1 giờ để troubleshoot
- **Root Cause**: Security group không allow SSH traffic từ my IP
- **Solution**: Update security group rules để allow port 22 từ my IP
- **Result**: SSH connection thành công
- **Lesson**: Always check security groups when có connectivity issues

### Vấn đề 2: IAM Role Permission Issues
- **Mô tả**: EC2 instance không thể access S3 bucket dù đã attach IAM role
- **Impact**: Mất 45 phút để debug
- **Root Cause**: Forgot to attach instance profile to EC2 instance
- **Solution**: Stop instance, modify IAM role, restart instance
- **Result**: EC2 có thể access S3 successfully
- **Lesson**: IAM roles cần instance profiles để work với EC2

## 💭 Reflection & Insights

### What went well today?
- Successfully launched và managed first EC2 instance
- Hiểu được relationship giữa IAM roles và EC2
- Nắm được security groups configuration

### What could be improved?
- Cần học thêm về different EC2 instance types
- Nên practice thêm với advanced EC2 features
- Cần cải thiện troubleshooting skills

### Key Insights
- EC2 là foundation của nhiều AWS services khác
- Security groups act như virtual firewall
- IAM roles provide secure way cho services to interact

---

*Worklog created by: Kieu Nguyen Thanh Binh*  
*Next review: 15/05/2025*
