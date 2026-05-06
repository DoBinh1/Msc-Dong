# BÁO CÁO 2 — HƯỚNG DẪN THỰC THI MÔ PHỎNG TRÊN ANSYS

Báo cáo này hướng dẫn bạn chạy lại mô phỏng của thầy Liu trên **ANSYS Mechanical APDL 2024 R2** từ đầu, kèm bảng kiểm và các "bẫy" thường gặp.

---

## 0. Chuẩn bị

### 0.1 Phiên bản và license
- Bài báo viết bằng ANSYS 18.1; bạn đang dùng 2024 R2 → **lệnh APDL gần như tương thích ngược 100 %**, nhưng:
  - Element `SOLID226` / `PLANE223` từ R2020+ có thêm DOF cho **diffusion** (CONC) — cần bật KEYOPT(1) đúng và **license phải có Multifield Solver (Mechanical Enterprise hoặc Mechanical Premium + Multiphysics)**. Nếu chỉ có Academic Teaching, có thể không có module diffusion — kiểm tra bằng `*GET,LIC,LIC,…` hoặc thử lệnh `TB,MIGR`.
  - `TB,MIGR` (atomic migration) cần license đầy đủ. Nếu chạy ra lỗi "Material model MIGR is not allowed for this license" → đổi license bậc cao hơn.

### 0.2 Đơn vị
Bài dùng hệ **uMKS** (μm). Mọi số liệu trong script đã được quy đổi sẵn — **đừng** tự nhân/chia thêm khi viết lại.

### 0.3 Khởi động sạch
Chạy ANSYS Mechanical APDL → **File ▸ Change Jobname** → đặt `Dong` → **File ▸ Change Working Directory** → trỏ vào `d:/[Lab] HUST/Đông`. Sau đó:

```
FINISH
/CLEAR,START
/FILNAME,Dong,1
/TITLE,Void growth in SAC strip
```

> Lưu ý: nếu jobname không được set trước, lúc ANSYS gặp lỗi sẽ tạo file `ansabort.err` và `ansabort.rst` (đó là lý do bạn thấy `ansabort0.err … ansabort3.err`).

---

## 1. Định nghĩa phần tử và vật liệu (PREP7)

```
/PREP7
ET,1,PLANE223,100111   ! 2D, struc-therm-elec-diff
KEYOPT,1,1,100111
ET,2,SOLID226,100111   ! 3D, struc-therm-elec-diff
KEYOPT,2,1,100111
```

**Bảng kiểm 1:**
- [ ] `ET,1,PLANE223` đã có
- [ ] `ET,2,SOLID226` đã có
- [ ] KEYOPT(1) = `100111` cho cả hai → bật UX/UY/UZ (bit 1), TEMP (bit 2), VOLT (bit 3), CONC (bit 6)
- [ ] Có hằng số `kB` và `kB_eV` ở đầu file (dùng cho TBDATA)

Khai báo vật liệu Cu (mat 1) và SAC (mat 2) đúng như trong `4.txt` — bạn không cần đụng vào.

---

## 2. Dựng hình học

Có hai cách:

### 2.1 Giữ nguyên cách cũ (như `4.txt`)
- Dựng ở mm, mesh, mirror, extrude, scale × 1000, scale × 0.01.
- Ưu: y nguyên script gốc.
- Nhược: **fragile** — nếu bạn lỡ xóa/thêm 1 keypoint hay đổi thứ tự, các số hiệu area/keypoint sẽ lệch và những lệnh `DA,11,VOLT,0` sẽ áp lên area sai.

### 2.2 Khuyến nghị: viết lại sạch ở μm ngay từ đầu

```
*SET,Lhalf, 1000        ! nửa chiều dài (μm)
*SET,W,     50          ! nửa chiều rộng
*SET,Rn,    50          ! bán kính khía
*SET,th,    0.5         ! chiều dày Z
*SET,Lpad,  100         ! chiều dài pad Cu
```

Sau đó dùng RECTNG/CYL4/ASBA để tạo **một nửa** (đối xứng qua trục Y), mesh, rồi `ARSYM,X,…` để mirror.

---

## 3. Chia lưới

```
type,1
mat,2          ! SAC body
MSHAPE,1,2D    ! tam giác cho vùng có khía
MSHKEY,0
esize, 5       ! μm (mịn quanh khía)
amesh, <area_SAC_quanh_khia>

mat,2
MSHAPE,0,2D    ! tứ giác mapped cho vùng strip xa khía
esize, 30
amesh, <area_SAC_dau_strip>

mat,1
amesh, <area_Cu_pad>
```

**Bẫy:** `esize=250` trong file gốc là cho hình học chưa scale × 0.01 (lúc đó cỡ 10 000 μm). Nếu bạn dựng hình ngay ở μm thật, dùng `esize` cỡ 3–30 μm.

---

## 4. Extrude 2D → 3D

```
TYPE,2
EXTOPT,ESIZE,1,0    ! 1 layer theo chiều dày
EXTOPT,ACLEAR,1     ! xoá mesh 2D sau khi extrude
EXTOPT,ATTR,1,0,0
VEXT,ALL, , ,0,0,th
NUMMRG,NODE,1e-7
ALLSEL
```

**Bảng kiểm 4:**
- [ ] Tất cả thể tích (volume) sau VEXT có gán material đúng (Cu hay SAC)
- [ ] `NUMMRG,NODE` đã chạy để dán các nút trùng

---

## 5. Điều kiện biên

Để **không phụ thuộc vào số hiệu area cứng**, chọn theo toạ độ:

```
! VOLT = 0 ở mặt x = -Lhalf
NSEL,S,LOC,X,-Lhalf-0.001,-Lhalf+0.001
D,ALL,VOLT,0
ALLSEL

! Nối VOLT (couple) trên mặt x = +Lhalf để cấp dòng tổng
NSEL,S,LOC,X,Lhalf-0.001,Lhalf+0.001
CP,1,VOLT,ALL
*GET,Nmaster,NODE,0,NUM,MIN
F,Nmaster,AMPS, 0.012e12     ! 0.012 A trong uMKS = 0.012e12 pA
ALLSEL

! Convection trên các mặt biên (trừ hai mặt cấp dòng)
ASEL,S,EXT
ASEL,U,LOC,X,-Lhalf-0.001,-Lhalf+0.001
ASEL,U,LOC,X, Lhalf-0.001, Lhalf+0.001
SFA,ALL,1,CONV,21,26
ALLSEL

! Cơ học: chặn rigid body
NSEL,S,LOC,Y,0
D,ALL,UY,0
NSEL,S,LOC,X,0
D,ALL,UX,0
NSEL,S,LOC,Z,0
D,ALL,UZ,0
ALLSEL
```

Cách này **bền vững** với mọi thay đổi mesh / số hiệu area.

---

## 6. Bài toán transient

```
TOFFST,273
TREF,26
/SOLU
ANTYPE,TRANS
TRNOPT,FULL
NROPT,FULL
AUTOTS,ON
NEQIT,50           ! 50 là đủ; 270 là quá lớn → tốn thời gian khi không hội tụ
LNSRCH,ON          ! BẬT line search — rất hữu ích cho coupled-field
CNVTOL,F,, 1e-3    ! nới convergence cho lực
CNVTOL,HEAT,, 1e-3
CNVTOL,AMPS,, 1e-3
CNVTOL,RATE,, 1e-3 ! cho diffusion
```

### Load step 1 (khởi tạo trạng thái T = 26, C = 1)
```
TIME,1e-6
DELTIM,0.5e-6,0.5e-6,0.5e-6
KBC,1
TIMINT,OFF
D,ALL,TEMP,26
D,ALL,CONC,1
SOLVE
```

### Load step 2 (bật transient, xoá ràng buộc T và C)
```
TIME,2e-6
DELTIM,0.5e-6,0.5e-6,0.5e-6
KBC,0
TIMINT,ON
DDELE,ALL,TEMP
DDELE,ALL,CONC
SOLVE
```

### Load step 3 (cấp dòng và chạy tới gần TTF)
**KHUYẾN NGHỊ:** chia thành 2 sub-load step để tránh shock-load phá hội tụ:

```
! 3a: cấp dòng từ từ trong 1 giây
TIME,1
DELTIM,0.1,1e-3,0.5
KBC,0                    ! ramp
F,Nmaster,AMPS,0.012e12
SOLVE

! 3b: chạy dài
TIME,16300000
DELTIM,1000,100,500000
KBC,1
SOLVE
```

### Load step 4 (mịn dần trước EBD)
```
TIME,16354000
DELTIM,3000,300,3000
SOLVE
```

---

## 7. Vòng EBD (Element Birth & Death)

```
/POST1
SET,LAST
ESEL,S,LIVE
ETABLE,ERAS
ETABLE,CCONC,SMISC,1     ! SMISC,1 cho SOLID226 với CONC = nồng độ trung bình phần tử
ESEL,R,ETAB,CCONC,, 0.85
*GET,nKill,ELEM,0,COUNT
/COM, *** Số phần tử sẽ chết: %nKill% ***
CM,TO_KILL,ELEM
FINISH

/SOLU
ANTYPE,,REST              ! restart thay vì khởi tạo mới
CMSEL,S,TO_KILL
EKILL,ALL
ALLSEL
TIME,16357000
DELTIM,3000,300,3000
SOLVE
```

Lặp lại 8–9 lần với `TIME` tăng thêm 3000 s mỗi lần. Trong file gốc dùng `*DO`. Cách an toàn hơn là viết một **macro file** `kill_step.mac` và gọi `*USE,kill_step,<TBASE>` cho mỗi bước — dễ debug hơn.

---

## 8. Trích xuất kết quả

```
/POST1
SET,LAST

! Bản đồ trường
PLNSOL,VOLT
PLNSOL,TEMP
PLNSOL,S,Y
PLNSOL,CONC

! Vector dòng điện
PLVECT,JC,,,VECT,ELEM,ON,0

! Điện trở strip = V_master / I_total
*GET,Vmaster,NODE,Nmaster,VOLT
*GET,Itot,NODE,Nmaster,RF,AMPS  ! reaction current
Rstrip = Vmaster / Itot
/COM, *** Rstrip = %Rstrip% TΩ (uMKS) = %Rstrip*1e6% Ω ***
```

Lặp `SET,nset` cho từng substep và lưu giá trị `Rstrip` vào array → vẽ Fig. 12, 13, Bảng II như trong bài báo.

---

## 9. Bảng kiểm tổng (Pre-flight)

| # | Mục | OK |
|---|---|---|
| 1 | Jobname đã đặt = `Dong` | ☐ |
| 2 | Working dir đã đúng | ☐ |
| 3 | License đủ Multiphysics + Diffusion | ☐ |
| 4 | KEYOPT(1) = 100111 cho cả PLANE223 lẫn SOLID226 | ☐ |
| 5 | Đơn vị nhất quán uMKS, mọi số liệu đã quy đổi | ☐ |
| 6 | Hình học đúng kích thước thực (μm) | ☐ |
| 7 | Mesh có ít nhất 4 lớp phần tử quanh khía | ☐ |
| 8 | NUMMRG đã chạy sau VEXT và sau ARSYM | ☐ |
| 9 | BC cấp dòng & nối đất ở **đúng** mặt | ☐ |
| 10 | TOFFST,273 và TREF,26 đã đặt | ☐ |
| 11 | TIMINT,OFF cho LS1, TIMINT,ON cho LS2+ | ☐ |
| 12 | DDELE đã xoá BC tạm sau LS1 | ☐ |
| 13 | CNVTOL được nới hợp lý | ☐ |
| 14 | LS3 dùng ramped current ít nhất ở giai đoạn đầu | ☐ |
| 15 | Restart đúng cú pháp `ANTYPE,,REST` cho mỗi bước EBD | ☐ |

Khi đủ 15 dấu tích, **rất có khả năng** mô phỏng chạy thành công.

---

## 10. Khi chạy lỗi — 3 việc đầu tiên

1. Mở `*.err` → tìm dòng `*** ERROR ***` đầu tiên (không phải dòng cuối). Lỗi sau thường là hệ quả của lỗi trước.
2. Mở `Dong.out` (hoặc `file.out`) → xem cùng thời điểm có warning gì về convergence, negative pivot, time step bisection.
3. Trong GUI, mở **Solution Information** → tab **Solution Output** → sẽ thấy thông báo "Substep XX did not converge" hoặc "Solution diverged" ở bước nào.

Phần "Sai và Sửa" ở file `SaiVaSua.md` đi vào chi tiết các lỗi cụ thể trong script `4.txt` của bạn.
