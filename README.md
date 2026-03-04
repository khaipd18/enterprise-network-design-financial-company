# Thiết Kế Hệ Thống Mạng Doanh Nghiệp & Hybrid Cloud - FastPay Financial

**Đồ án môn học:** NT113.Q11 – Thiết kế mạng  
**Trường:** Đại học Công nghệ Thông tin, ĐHQG-HCM  
**GVHD:** ThS. Bùi Thanh Bình  
**Nhóm thực hiện:** Nhóm 01  

| STT | Sinh viên | MSSV | Vai trò |
|:---:|:---|:---:|:---|
| 1 | Phan Đình Khải | 23520678 | Trưởng nhóm |
| 2 | Đoàn Quốc An | 23520006 | Thành viên |
| 3 | Lê Chánh Ân | 23520007 | Thành viên |
| 4 | Nguyễn Đoàn Hồng Huy | 23520628 | Thành viên |

---

## 📖 Mục Lục

1.  [Giới Thiệu Dự Án](#1-giới-thiệu-dự-án)
2.  [Mục Tiêu & Yêu Cầu Kỹ Thuật](#2-mục-tiêu--yêu-cầu-kỹ-thuật)
3.  [Kiến Trúc Mạng (Topology)](#3-kiến-trúc-mạng-topology)
4.  [Công Nghệ Sử Dụng](#4-công-nghệ-sử-dụng)
5.  [Quy Hoạch Địa Chỉ IP](#5-quy-hoạch-địa-chỉ-ip-ip-schema)
6.  [Giải Pháp Bảo Mật (Zero Trust)](#6-giải-pháp-bảo-mật-zero-trust)
7.  [Triển Khai Hybrid Cloud (AWS)](#7-triển-khai-hybrid-cloud-aws)
8.  [Kế Hoạch Triển Khai (PDIOO)](#8-kế-hoạch-triển-khai-pdioo)
9.  [Ngân Sách & Chi Phí Vận Hành](#9-ngân-sách--chi-phí-vận-hành)
10. [Kết Luận & Hướng Phát Triển](#10-kết-luận--hướng-phát-triển)
11. [Tài Liệu Tham Khảo](#11-tài-liệu-tham-khảo)

---

## 1. Giới Thiệu Dự Án

**FastPay** là công ty Fintech cung cấp dịch vụ cổng thanh toán và ví điện tử. Dự án thiết kế hạ tầng mạng kết nối Trụ sở chính (HQ - TP.HCM) với 02 chi nhánh (Hà Nội, Đà Nẵng) và tích hợp **Hybrid Cloud**.

**Thách thức:**
* Bảo vệ dữ liệu tài chính nhạy cảm.
* Đảm bảo giao dịch thời gian thực (Real-time).
* Khả năng mở rộng linh hoạt (Auto Scaling).

---

## 2. Mục Tiêu & Yêu Cầu Kỹ Thuật

* **Security First (Zero Trust):** Phân vùng mạng nghiêm ngặt, kiểm soát truy cập đa lớp.
* **High Availability:** Dự phòng Gateway (HSRP), Đa đường truyền (Dual-ISP), Site-to-Site VPN.
* **Flexibility:** Core Banking tại On-premise (HQ), Web App trên AWS Cloud.

---

## 3. Kiến Trúc Mạng (Topology)

### 3.1. Mô hình Hub-and-Spoke
* **Hub:** HQ (TP.HCM) - Trung tâm dữ liệu.
* **Spokes:** Chi nhánh Hà Nội & Đà Nẵng.
* **Cloud:** AWS Region Singapore.

> <img width="1818" height="732" alt="image" src="https://github.com/user-attachments/assets/7b718b31-6089-491f-950f-a02fcc744f10" />
*Hình 1. Mô hình mạng tổng quan*

### 3.2. Mô hình Phân cấp (3-Layer Model)
* **Core Layer:** Router Cisco C8500 chịu tải cao.
* **Distribution Layer:** Switch Cisco C9500, định tuyến VLAN, Policy.
* **Access Layer:** Switch Cisco C9300, PoE cho Access Points.

> <img width="1496" height="709" alt="image" src="https://github.com/user-attachments/assets/611e316f-a51c-411c-9c4a-2a4ab957abbb" />
*Hình 2. Sơ đồ chi tiết và phân vùng VLAN tại Trụ sở (HQ)*

> <img width="1142" height="699" alt="image" src="https://github.com/user-attachments/assets/34f5d427-4422-496e-9301-c2f68da6df5c" />
*Hình 3. Kiến trúc mạng tại Chi nhánh Hà Nội & Đà Nẵng*
---

## 4. Công Nghệ Sử Dụng

| Hạng mục | Công nghệ / Giao thức | Vai trò |
|:---|:---|:---|
| **Routing** | OSPF (Multi-area) | Định tuyến nội bộ, hội tụ nhanh. |
| **Redundancy** | HSRP | Dự phòng Gateway (Active/Standby). |
| **Switching** | VLAN, 802.1Q | Phân chia mạng logic. |
| **WAN** | IPSec VPN (IKEv2) | Kết nối Site-to-Site mã hóa AES-256. |
| **Internet** | Dual-ISP | Load Balancing (Viettel + FPT). |
| **Wireless** | Wi-Fi 6 (802.11ax) | WPA2-Enterprise (802.1X). |
| **Monitoring** | Zabbix, Wazuh | Giám sát hiệu suất & SIEM. |

---

## 5. Quy Hoạch Địa Chỉ IP (IP Schema)

### 5.1. Mạng Nội Bộ (LAN)

| Vị trí | VLAN | Tên | Subnet | Gateway | Ghi chú |
|:---|:---:|:---|:---|:---|:---|
| **HQ** | 10 | Kế toán | `10.10.10.0/24` | `.1` | Bảo mật cao |
| | 20 | Risk Mgmt | `10.10.20.0/24` | `.1` | Analytics |
| | 30 | IT / Admin | `10.10.30.0/24` | `.1` | Full Access |
| | 50 | Staff Wi-Fi | `10.10.50.0/24` | `.1` | |
| | - | **Data Center** | **`172.16.70.0/24`** | **`.1`** | **Core Services** |
| | - | Guest Wi-Fi | `172.20.10.0/24` | `.1` | Internet Only |
| **Đà Nẵng** | 10 | IT Branch | `10.20.10.0/24` | `.1` | |
| | 20 | Staff Branch | `10.20.20.0/24` | `.1` | |
| **Hà Nội** | 10 | IT Branch | `10.30.10.0/24` | `.1` | |
| | 20 | Staff Branch | `10.30.20.0/24` | `.1` | |

### 5.2. Kết nối VPN (Tunnels)

| Tunnel | Kết nối | IP Network (/30) |
|:---:|:---|:---|
| Tu1, Tu2 | HQ - Đà Nẵng | `192.168.10.0`, `192.168.10.4` |
| Tu5, Tu6 | HQ - Hà Nội | `192.168.10.16`, `192.168.10.20` |
| Tu9 - Tu12 | HQ - AWS Cloud | `192.168.10.32` ... |

---

## 6. Giải Pháp Bảo Mật (Zero Trust)

Hệ thống áp dụng mô hình **Defense-in-Depth**:

1.  **Perimeter Security (Router Biên):**
    * **Stateful ACL:** Giả lập Firewall, chỉ cho phép traffic phản hồi (`established`) từ Internet.
    * **NAT Exemption:** Không NAT traffic VPN để đảm bảo định tuyến.
2.  **Internal Segmentation (Switch Phân phối):**
    * **Chính sách "Một chiều" (Branch):** IT truy cập được Kế toán, nhưng Kế toán KHÔNG THỂ khởi tạo kết nối sang IT.
    * **VLAN Isolation (HQ):** Kế toán chỉ được truy cập DB Server (Port 1433); Risk chỉ truy cập Analytics Server (Port 3306).
3.  **Authentication:**
    * **802.1X / RADIUS:** Xác thực người dùng truy cập mạng nội bộ.

---

## 7. Triển Khai Hybrid Cloud (AWS)

Mở rộng hạ tầng lên **AWS Region Singapore (ap-southeast-1)**.

### Kiến trúc AWS
* **VPC:** Mạng ảo riêng biệt.
* **Public Subnet:** NAT Gateway, Application Load Balancer (ALB).
* **Private Subnet:** EC2 Instances (Web Servers) - Không Public IP.
* **Security:** AWS WAF (Chặn SQLi, XSS), Security Groups.
* **Connectivity:** AWS Site-to-Site VPN kết nối về HQ.

> <img width="2647" height="1101" alt="NT113 drawio" src="https://github.com/user-attachments/assets/f628d8db-c693-43e6-b571-57de0ccc72f7" />

*Hình 4. Sơ đồ triển khai AWS*

---

## 8. Kế Hoạch Triển Khai (PDIOO)

Dự án tuân thủ vòng đời **PPDIOO** của Cisco:

1.  **Prepare (Chuẩn bị):** Khảo sát hạ tầng điện, lạnh, không gian tủ Rack. Đặt hàng thiết bị (BoM).
2.  **Plan (Lập kế hoạch):** Xác định yêu cầu mạng, quy hoạch IP, thiết kế High Level (HLD).
3.  **Design (Thiết kế):** Thiết kế chi tiết (LLD), kịch bản định tuyến, chính sách bảo mật, cấu hình mẫu.
4.  **Implement (Triển khai):**
    * Lắp đặt phần cứng (Rack & Stack).
    * Cấu hình Underlay (Internet/VLAN).
    * Triển khai Overlay (VPN/SD-WAN).
    * Kiểm thử UAT và Cutover.
5.  **Operate (Vận hành):** Giám sát 24/7 qua Dashboard, xử lý sự cố (Troubleshooting).
6.  **Optimize (Tối ưu):** Tinh chỉnh Load Balancing, cập nhật Firmware, nâng cấp băng thông.

---

## 9. Ngân Sách & Chi Phí Vận Hành

### 9.1. Chi Phí Đầu Tư Ban Đầu (CAPEX)

| Hạng mục | Chi tiết | Ước tính (USD) |
|:---|:---|:---:|
| **Router** | Cisco C8500 (HQ & Branch) | $283,804 |
| **Switch** | Cisco Nexus 7000 (DC) & C9500/C9300 | $153,467 |
| **Wireless** | WLC 9800-L & AP Cat 9120/9115 | $33,032 |
| **Cabling** | Cáp quang OM4/OS2, Cat6A | ~$10,675 |
| **TỔNG CỘNG** | | **~$481,000** |

### 9.2. Chi Phí Vận Hành Hàng Tháng (OPEX)

| Hạng mục | Chi tiết | Chi phí (VND/Tháng) | Chi phí (USD/Tháng) |
|:---|:---|:---:|:---:|
| **Internet HQ** | Viettel VIP600 + FPT Super 800 | 11,800,000 | ~$450 |
| **Internet Branch** | Viettel VIP500 + FPT Super 500 | 8,280,000 | ~$312 |
| **AWS Cloud** | EC2, ALB, WAF, VPN, CloudWatch | ~19,650,000 | ~$746 |
| **TỔNG CỘNG** | | **~39,730,000** | **~$1,508** |

---

## 10. Kết Luận & Hướng Phát Triển

### Kết quả đạt được
* ✅ **Unified Connectivity:** Hợp nhất hạ tầng mạng rời rạc thành một hệ thống SD-WAN Overlay đồng nhất.
* ✅ **Performance:** Cơ chế Active-Active tận dụng 100% băng thông đường truyền.
* ✅ **Security:** Mã hóa toàn trình dữ liệu, tuân thủ chuẩn bảo mật tài chính.
* ✅ **Hybrid Ready:** Kết nối thành công và an toàn với AWS Cloud.

### Hướng phát triển
1.  **Nâng cấp phần cứng:** Chuyển sang các dòng High-end khi nhân sự tăng trưởng.
2.  **Spine-Leaf Architecture:** Áp dụng cho Data Center khi mở rộng Server Farm.
3.  **AWS Direct Connect:** Thay thế VPN qua Internet bằng đường truyền vật lý riêng biệt để giảm độ trễ tối đa.

---

## 11. Tài Liệu Tham Khảo

1.  Bài giảng Thiết kế mạng, Khoa MMT&TT, UIT (2025–2026).
2.  Cisco Systems, *Cisco Catalyst SD-WAN Design Guide*.
3.  Amazon Web Services, *AWS VPC & VPN Documentation*.
4.  GeeksforGeeks, *Three Layer Hierarchical Model*.
5.  AlgoSec, *Network Lifecycle Management*.

---
*© 2025 Nhóm 01 - Khoa Mạng máy tính & Truyền thông - UIT*
