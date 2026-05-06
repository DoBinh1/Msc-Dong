# BÁO CÁO 1 — HIỂU BÀI BÁO

**Bài báo:** *Numerical Simulation of Void Growth in Electrical Solder Material* — Thomas Jin-Chee Liu (Ming Chi University of Technology, Taiwan), CPEEE 2025.

**Người đọc:** Đông (HUST) — học viên thạc sỹ.

**Mục tiêu báo cáo:** Tóm tắt vật lý, mô hình toán, dữ liệu vật liệu, quy trình mô phỏng và các kết quả chính, để hiểu *tại sao* thầy lại làm từng bước như trong file `4.txt`.

---

## 1. Bối cảnh và động lực

Trong các vi mạch điện tử (chip packaging, solder joints, solder balls, solder wires), mối hàn **SAC (Sn–Ag–Cu)** là điểm yếu hay hỏng nhất. Dòng điện chạy qua mối hàn lâu ngày sẽ làm nguyên tử Sn dịch chuyển, tạo lỗ rỗng (*void*) ở phía dòng điện tử đi vào và đùn nguyên tử (*hillock*) ở phía đối diện. Khi void lớn dần, tiết diện dẫn điện thu hẹp lại, điện trở tăng và cuối cùng mối hàn đứt.

Hiện tượng này được dùng để định nghĩa **Time To Failure (TTF)**: thời điểm điện trở tăng 15 % so với ban đầu (theo tiêu chuẩn của tham khảo [1]).

---

## 2. Bốn cơ chế dịch chuyển nguyên tử

Tổng dòng nguyên tử (atomic flux) là tổ hợp của bốn nguồn:

$$ \mathbf{J}_t = \mathbf{J}_{em} + \mathbf{J}_{th} + \mathbf{J}_{st} + \mathbf{J}_d $$

| Cơ chế | Công thức | Động lực |
|---|---|---|
| Electromigration (EM) | $\mathbf{J}_{em} = -\dfrac{C Z^* e D}{k_B T}\nabla V$ | Gradient điện thế $\nabla V$ |
| Thermomigration (TM) | $\mathbf{J}_{th} = -\dfrac{C Q^* D}{k_B T^2}\nabla T$ | Gradient nhiệt $\nabla T$ |
| Stress migration (SM) | $\mathbf{J}_{st} = \dfrac{C \Omega D}{k_B T}\nabla H$ | Gradient ứng suất thủy tĩnh $\nabla H$ |
| Pure diffusion | $\mathbf{J}_d = -D\,\nabla C$ | Gradient nồng độ $\nabla C$ |

Hệ số khuếch tán phụ thuộc nhiệt độ (Arrhenius):

$$ D = D_0 \exp\!\left(-\frac{E_a}{k_B T}\right) = D_0 \exp\!\left(-\frac{E_{am}}{R T}\right) $$

→ Bài toán là **electro–thermo–structural–diffusion coupled-field** (4 trường vật lý đồng thời). Trong ANSYS, đây chính là lý do phải dùng phần tử **PLANE223 / SOLID226** với KEYOPT(1) = `100111` (cờ bật cho 4 DOF: UX/UY/UZ, TEMP, VOLT, CONC).

---

## 3. Mô hình hình học

- Mảnh SAC dạng dải (strip) chữ nhật, có **khía nửa hình tròn** ở mép, mô phỏng vết khía nơi tập trung dòng điện và ứng suất.
- Kích thước: $L = 2000\,\mu m$, $W = 100\,\mu m$, $e = 0.5\,\mu m$, $R = 50\,\mu m$ (dài × rộng × dày × bán kính khía).
- Hai đầu strip có pad đồng (Cu) để cấp dòng.

Trong file `4.txt`, hình học được dựng ở đơn vị **mm**, sau đó scale × 1000 (mm → μm), rồi extrude 2D → 3D, mirror đối xứng qua trục Y, và scale lần cuối × 0.01 để khớp kích thước thật. Đây là cách rất "GUI-recorded" — không clean nhưng vẫn ra hình đúng.

---

## 4. Hệ đơn vị (rất quan trọng để khỏi sai số liệu)

Bài dùng hệ **uMKS** (μm-based MKS):

| Đại lượng | SI | uMKS |
|---|---|---|
| Chiều dài | m | μm |
| Lực | N | μN |
| Ứng suất | Pa | MPa |
| Nhiệt độ | K hoặc °C | °C (offset 273) |
| Năng lượng | J | pJ ($10^{-12}$ J) |
| Dòng điện | A | pA |
| Điện áp | V | V |
| Điện trở | Ω | TΩ |
| Khối lượng riêng | kg/m³ | kg/μm³ ($\times 10^{-18}$) |

Vì thế trong script bạn thấy các phép nhân kiểu `393*1e6`, `2.38e-8*1e-6`, `127.7e9*1e-6`, `8900*1e-18` — đó **không phải** lỗi đánh máy mà là phép quy đổi đơn vị.

---

## 5. Dữ liệu vật liệu

### 5.1 Đồng (Cu — pad cấp dòng)

| Thông số | Giá trị |
|---|---|
| Điện trở suất ρ @ 200 °C | 2.38 × 10⁻⁸ Ω·m |
| Pre-exp diffusivity D₀ | 7.8 × 10⁻⁵ m²/s |
| Activation energy Qₐ | 210 × 10³ J/(mol·K) |
| Charge number Z* | −4 |
| Atomic volume Ω | 1.182 × 10⁻²⁹ m³ |
| Young's modulus E | 127.7 GPa |
| Poisson ν | 0.31 |
| α (thermal expansion) | 17.1 × 10⁻⁶ 1/K |

### 5.2 SAC (Sn–Ag–Cu — mối hàn)

| Thông số | Giá trị |
|---|---|
| Điện trở suất ρ @ 200 °C | 20.75 × 10⁻⁸ Ω·m |
| Pre-exp diffusivity D₀ | 4.1 × 10⁻⁵ m²/s |
| Activation energy Eₐ | 0.98 eV |
| Heat of transport Q* | 0.0094 eV |
| Charge number Z* | −23 |
| Atomic volume Ω | 2.71 × 10⁻²⁹ m³ |
| Young's modulus E | 26.2 GPa |
| Poisson ν | 0.35 |
| α | 23 × 10⁻⁶ 1/K |
| Cref | 1 |
| β (diffusion expansion) | 1 × 10⁻⁵ |

(Bảng I trong bài báo có nhiều con số bị OCR sai khi đổi sang hệ uMKS — script trong `4.txt` mới là nguồn đúng.)

Trong APDL, mô hình migration được khai báo bằng:

```
TB,MIGR,<mat>
TBDATA,1, Eₐ/k_B    ! diffusivity Arrhenius
TBDATA,2, Ω/k_B     ! stress migration
TBDATA,3, Q*/k_B    ! thermomigration  (chỉ cho SAC)
TBDATA,4, Z*/k_B    ! electromigration
```

---

## 6. Điều kiện biên và tải

- **Điện**: nối đất (V = 0) ở một đầu strip (DA … VOLT,0); cấp dòng I₀ = 0.012 A ở đầu kia (FK … AMPS, TOC).
- **Nhiệt**: đối lưu trên mọi mặt ngoài, h = 21 W/(m²·°C), T∞ = 26 °C (SFA … CONV).
- **Cơ**: ràng buộc UZ ở cạnh dưới, UY = 0 ở mặt đáy, UX = 0 ở vài đường để khử rigid-body motion.
- **Khuếch tán**: điều kiện đầu C = 1 trên toàn miền (D,all,,1,…,CONC), nhiệt độ đầu T = 26 °C.

---

## 7. Quy trình giải (transient, 4 trường ghép)

| Load step | TIME | DELTIM / NSUBST | KBC | Mục đích |
|---|---|---|---|---|
| 1 (initial steady) | 1 × 10⁻⁶ s | 0.5e-6 | 1 (stepped) | Áp T = 26, C = 1 ở mọi nút, **TIMINT,OFF** để khởi tạo trường ban đầu |
| 2 | 2 × 10⁻⁶ s | 0.5e-6 | 0 (ramped) | Bật **TIMINT,ON**; xoá ràng buộc T và C để chúng được tính tự do |
| 3 | 16,300,000 s (≈ 4528 h) | NSUBST 30 | 1 | Áp dòng 0.012 A và chạy tới gần thời điểm bắt đầu void |
| 4 | 16,354,000 s | DELTIM 3000 | — | Chạy mịn trước EBD |
| 5+ | Vòng *DO 9 bước, mỗi bước +3000 s | DELTIM 3000 | — | **EBD**: kill các phần tử thoả C ≤ 0.85, solve lại |

EBD ("Element Birth & Death") chính là cách thầy mô phỏng void lớn dần: sau mỗi bước, những phần tử nào có nồng độ chuẩn hoá $C \le C_{void} = 0.85$ thì *kill* (bị loại khỏi mô hình), tạo nên hình dạng void thay đổi theo thời gian.

---

## 8. Kết quả mong đợi (đối chiếu được với bài báo)

1. **Trường dòng điện $J_y$** tập trung quanh khía bán nguyệt (Fig. 7).
2. **Trường nhiệt độ** có hot-spot quanh khía (Fig. 8).
3. **Ứng suất pháp $\sigma_y$** tập trung quanh khía (Fig. 9).
4. **Trường nồng độ chuẩn hoá $C$** thấp ở thượng nguồn dòng electron (Fig. 10).
5. **Lịch sử void** (Fig. 11): void xuất hiện ~ t = 16,357,000 s (≈ 4544 h) và lớn dần.
6. **Thể tích void** vs. thời gian (Fig. 12): tốc độ tăng nhanh dần.
7. **Width còn lại $w_n$** giảm theo thời gian (Fig. 13).
8. **Điện trở strip** tăng từ 8.71 Ω lên 10.38 Ω; vượt mức 15 % giữa 16,378,000 s và 16,381,000 s — đó là **TTF**.

---

## 9. Vì sao cần làm cẩn thận khi viết lại?

- Mô hình **cực kỳ đa vật lý** — sai một KEYOPT, một đơn vị, một dấu trừ là toàn bộ kết quả vô nghĩa.
- Hình học được **scale 2 lần** (× 1000 rồi × 0.01) → dễ nhầm khi viết lại sạch hơn.
- Số hiệu area / keypoint / line phụ thuộc thứ tự lệnh — script gốc cứng nhắc dùng số hiệu (DA,11; FK,11; ASEL…,13) → khi sửa hình học là phải gán lại.
- Vòng EBD bằng *DO/CMWRITE/EKILL có nhiều cạm bẫy nếu lần solve đầu không hội tụ.

Phần **báo cáo 2** sẽ trình bày thứ tự thao tác và bảng kiểm để chạy được mô phỏng từ đầu, kể cả khi bạn dùng ANSYS 2024 R2 (không phải 18.1 như bài báo).
