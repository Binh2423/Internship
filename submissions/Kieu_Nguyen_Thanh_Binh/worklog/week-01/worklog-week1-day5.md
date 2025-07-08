# Worklog - Ngày 16/05/2025

## 📅 Thông tin cơ bản
- **Ngày**: 16/05/2025
- **Thứ**: Thứ Sáu
- **Tuần thực tập**: Tuần thứ 1/12
- **Thời gian làm việc**: 8:00 - 17:00
- **Mood**: 😊 + Hào hứng với database services

## 🎯 Mục tiêu ngày hôm nay
- [x] Tìm hiểu về Amazon RDS và tạo database instance
- [x] Khám phá Amazon Lightsail cho simplified computing
- [x] So sánh RDS vs self-managed database

## 💼 Công việc đã thực hiện

### 1. Database Essentials with Amazon RDS ⏱️ 4 giờ
- **Mô tả**: Tạo MySQL RDS instance, configure security groups, connect từ EC2
- **Kết quả**: RDS instance running, có thể connect và query từ application
- **Tools/Tech**: Amazon RDS (MySQL), MySQL Workbench, Security Groups
- **Links**: RDS configuration, Database connection strings

### 2. Simplified Computing with Amazon Lightsail ⏱️ 3 giờ
- **Mô tả**: Tạo Lightsail instance, so sánh với EC2, deploy simple application
- **Kết quả**: Lightsail instance với WordPress deployed successfully
- **Tools/Tech**: Amazon Lightsail, WordPress, Lightsail Console
- **Links**: Lightsail instance URL, WordPress admin panel

## 📚 Kiến thức học được

### 🔧 Technical Skills
- **AWS Services**: RDS (MySQL), Lightsail, Database Security Groups
- **Database**: SQL basics, Database connectivity, Backup strategies
- **System Administration**: Managing database instances

### 💡 Concepts & Theory
- **New Concepts**: Managed vs self-managed databases, Database as a Service
- **Best Practices**: Database security, backup and recovery
- **Industry Knowledge**: When to use RDS vs Lightsail vs EC2

### 🤝 Soft Skills
- **Decision Making**: Choosing right service for use case
- **Analytical Thinking**: Comparing different AWS services
- **Documentation**: Recording database configurations

## 🚧 Khó khăn và giải pháp

### Vấn đề 1: RDS Connection Issues
- **Mô tả**: Không thể connect tới RDS instance từ EC2
- **Impact**: Mất 1.5 giờ để troubleshoot
- **Root Cause**: RDS security group không allow inbound từ EC2 security group
- **Solution**: Update RDS security group để allow MySQL port từ EC2
- **Result**: Connection established successfully
- **Lesson**: Database security groups cần careful configuration

### Vấn đề 2: Lightsail vs EC2 Confusion
- **Mô tả**: Không hiểu rõ khi nào nên dùng Lightsail vs EC2
- **Impact**: Mất thời gian research và so sánh
- **Root Cause**: Chưa hiểu rõ use cases của từng service
- **Solution**: Đọc documentation và thực hành với cả hai
- **Result**: Hiểu rõ trade-offs giữa simplicity và flexibility
- **Lesson**: Each AWS service has specific use cases

## 💭 Reflection & Insights

### What went well today?
- Successfully deployed managed database với RDS
- Hiểu được benefits của managed services
- Nắm được khi nào nên dùng Lightsail

### What could be improved?
- Cần học thêm về database optimization
- Nên practice thêm với different database engines
- Cần hiểu rõ hơn về database backup strategies

### Key Insights
- Managed services save significant operational overhead
- Lightsail is great for simple use cases
- Database security requires multiple layers

---

*Worklog created by: Kieu Nguyen Thanh Binh*  
*Next review: 19/05/2025*
