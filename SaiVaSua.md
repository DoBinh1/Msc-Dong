# SAI VÀ SỬA — Phân tích lỗi trong script `4.txt`

Báo cáo này chỉ ra **những điểm sai/yếu cụ thể** trong script `4.txt` và **cách sửa**. Tôi đối chiếu cả 4 file `ansabort*.err` để biết ANSYS đã chết ở đâu.

---

## A. Bằng chứng từ file lỗi

Cả `ansabort1.err`, `ansabort2.err`, `ansabort3.err` chỉ chứa 2 dòng banner (ngày giờ chạy) → có nghĩa là 3 lần chạy gần nhất, ANSYS **chết quá sớm, trước khi kịp ghi error message thật**. Đó là dấu hiệu của:
- File jobname chưa được set (nên file lỗi đổ về tên mặc định `ansabort.err`)
- Hoặc lệnh đầu tiên trong script đã sai cú pháp / thiếu license và ANSYS thoát ngay.

Còn `ansabort0.err` (lần chạy cũ hơn) cho thấy:
```
*** WARNING *** No element table items stored for the selected item group.
                The PRETAB command is ignored.
*** ERROR ***   An error occurred while attempting to open the results file
                ansabort.rst.
*** ERROR ***   No results file available.
```

→ Lần đó ANSYS chạy được tới phần `/POST1` nhưng **không có file `.rst`**. Tức là toàn bộ phần `/SOLU` không sản sinh ra kết quả nào — `SOLVE` đã thất bại nhưng script vẫn tiếp tục.

---

## B. Các lỗi cụ thể trong `4.txt` (theo thứ tự xuất hiện)

### B.1 Thiếu `/FILNAME` đặt jobname

Dòng `/filename,Dong` (line 12) — tên lệnh đúng phải là **`/FILNAME`** (chỉ 1 chữ "n"), không phải `/FILENAME`. Bạn dùng `/filename` → ANSYS coi như lệnh không hợp lệ và **bỏ qua**, jobname rơi về mặc định `ansabort` hoặc `file`.

**Sửa:**
```apdl
/FILNAME,Dong,1
```

### B.2 KEYOPT(1) khai báo trùng lặp

```
ET,1,PLANE223,100111
KEYOPT,1,1,100111      ! lặp lại
```

Không sai nhưng dư. Bỏ một trong hai. Ngoài ra với PLANE223 trong ANSYS 2024 R2, KEYOPT(1) chấp nhận tổ hợp tới `110011`/`100111`. **Hãy verify** trong Element Reference: vào *Help ▸ Mechanical APDL ▸ Element Reference ▸ PLANE223 ▸ KEYOPT(1) Settings*. Nếu R2024 yêu cầu `1000111` (7 chữ số) thì sửa cả hai chỗ.

### B.3 KEYOPT(1) cho **PLANE223** có thể không cần CONC

Bạn dùng PLANE223 chỉ để mesh 2D rồi **EXTRUDE** sang SOLID226. Sau khi EXTOPT/VEXT, mesh 2D bị xoá. Phần tử thực giải là SOLID226. Vì thế KEYOPT(1) của PLANE223 không quá quan trọng — nhưng nếu sai có thể chặn việc mesh. Để an toàn, để giống nhau ở cả hai.

### B.4 Hình học scale × 1000 rồi × 0.01 — kích thước cuối là **1/10** mong muốn

- Bạn dựng ở mm với chiều dài tổng (sau mirror) = 20 mm × 100 mm.
- `ARSCALE,...,1000,1000,1000` → 20 000 × 100 000 μm.
- `VLSCAL,...,0.01,0.01,0.01` → 200 × 1000 μm.

Trong khi bài báo ghi $L = 2000\,\mu m$, $W = 100\,\mu m$ → bạn đang **chia đôi nhầm** chiều dài và **gấp đôi** chiều rộng. Dù vẫn ra hình "strip có khía" nhưng tỉ lệ và do đó các trường gradient sẽ khác bài báo.

**Sửa:** bỏ luôn `VLSCAL` (line 291–294) và đổi `ARSCALE` thành scale = 100 thay vì 1000:
```
ARSCALE,ALL,,,100,100,100,,0,1
```
hoặc tốt hơn — viết lại hình học trực tiếp ở μm (xem `BaoCao_2_HuongDanThucThi.md` mục 2.2).

### B.5 BC điện áp dùng số hiệu area cứng

```
ASEL,S, , ,13      ! area 13 — chắc gì có sau khi mirror + extrude?
NSLA,S,1
CP,1,VOLT,all
DA,11,VOLT,0       ! area 11
```

Số hiệu area `11`, `13` là **giả định** — chỉ đúng với đúng thứ tự lệnh và đúng phiên bản ANSYS gốc (18.1). Trong R2024 bộ đánh số có thể khác.

**Cách kiểm tra nhanh:** sau khi extrude, chạy:
```
APLOT
/PNUM,AREA,1
/REPLOT
```
rồi nhìn xem area cấp dòng và area nối đất là area nào. Sửa số hiệu cho đúng.

**Tốt hơn:** chọn theo toạ độ (xem báo cáo 2 mục 5).

### B.6 `D, all, , 1, , , ,conc` áp lên **mọi node**

Lệnh này áp `CONC = 1` (Dirichlet) trên **mọi node**, không chỉ node biên. Đây là cách rất "thô" để khởi tạo trường ban đầu, kết hợp với `TIMINT,OFF` trong LS1. Sau LS1, `DDELE,ALL,CONC` xoá hết.

Vấn đề: nếu LS1 không hội tụ, `DDELE` không kịp chạy, và LS2 vẫn giữ ràng buộc CONC = 1 trên mọi node → mô hình trở thành **trường C cố định** → không có khuếch tán → không có void!

**Sửa:** dùng `IC,ALL,CONC,1` và `IC,ALL,TEMP,26` thay vì `D` để đặt **initial condition** (không phải Dirichlet). Đó là cách đúng và an toàn:
```
IC,ALL,TEMP,26
IC,ALL,CONC,1
```
Khi đó không cần `TIMINT,OFF / TIMINT,ON / DDELE` nữa — LS1 có thể bỏ luôn.

### B.7 LS3 quá thô — gần như chắc chắn không hội tụ

```
TIME,16300000
nsubst,30,30,30   ! 30 substep cho 16.3 triệu giây ⇒ ~543 000 s/substep
KBC,1             ! step load: nhảy dòng 0.012 A trong 1 substep
```

Một sub-step dài 543 000 s với electromigration là **khổng lồ**, AUTOTS sẽ liên tục bisect, NEQIT 270 sẽ bị hết, và cuối cùng SOLVE thất bại.

**Sửa:** chia nhỏ:
```
! Pha 1: ramp dòng trong 1 s
TIME,1
DELTIM,0.05,1e-3,0.5
KBC,0
F,Nmaster,AMPS,0.012e12
SOLVE

! Pha 2: chạy nhanh tới 1e6 s
TIME,1e6
DELTIM,1000,100,100000
KBC,1
SOLVE

! Pha 3: chạy mịn dần tới 16.3e6
TIME,16300000
DELTIM,10000,1000,500000
SOLVE
```

### B.8 Vòng `*DO` có lỗi logic về biến `i`

```
i = 1
*DO,LS,1,NSTEP,1
   /SOLU
   /INPUT,TO_KILL_%i%,cm   ! đọc lại file (vô nghĩa, component đã có sẵn)
   ALLSEL
   CMSEL,S,TO_KILL_%i%
   EKILL,ALL
   …
   SOLVE
   i = i + 1
   FINISH
   /POST1
   …
   CM,TO_KILL_%i%,ELEM
   CMWRITE,TO_KILL_%i%,cm
*ENDDO
```

Sửa lỗi/yếu:
1. **`/INPUT,TO_KILL_%i%,cm` là dư**. Component `TO_KILL_i` đã được CMWRITE từ trước, đồng thời cũng vẫn còn trong database. Bỏ dòng này.
2. **`FINISH` trong vòng *DO** rồi quay lại `/SOLU` ở vòng sau là **không cần** restart đúng cách. Phải có `ANTYPE,,REST` để báo ANSYS đây là restart, nếu không nó coi là analysis mới và **xoá kết quả cũ**.
3. **`i = i + 1` trước khi ghi component mới** → đúng. Nhưng tốt hơn dùng `LS+1` thay biến tách rời `i`:
   ```
   CM,TO_KILL_%(LS+1)%,ELEM
   ```
4. **Không kiểm tra rỗng**: nếu `ESEL,R,ETAB,CCONC,, REFC` không chọn được phần tử nào, `CM,TO_KILL_x` vẫn tạo component **rỗng**, và lần sau `EKILL,ALL` không kill gì → vòng lặp tiếp tục vô tích sự cho tới khi vượt NSTEP.

   **Sửa:** kiểm tra:
   ```
   *GET,nKill,ELEM,0,COUNT
   *IF,nKill,EQ,0,EXIT
   ```

### B.9 Thiếu `LSWRITE` / `LSSOLVE`

Bài có nhiều load step. Cách sạch là dùng `LSWRITE,n` cho mỗi LS rồi `LSSOLVE,1,4` ở cuối — dễ debug khi load step nào fail. Hiện tại bạn `SOLVE` ngay sau mỗi LS, nếu fail thì rất khó re-run riêng từng LS.

### B.10 `mat,2 amesh,6` rồi `mat,2 amesh,1` (line 166–169)

Cả hai đều là material 2. Lý do: bạn muốn cả strip ngoài lẫn vùng quanh khía đều là SAC. Đúng nhưng dòng `mat,2` đầu (line 166) là dư — bạn vừa set `mat,2` ở line 164. Dù không sai, đọc rất rối. Bỏ một dòng.

### B.11 Tham chiếu `ETABLE,CCONC,SMISC,1` chưa chắc đúng cho R2024

ANSYS đôi khi **đổi index SMISC/NMISC** giữa các phiên bản. SMISC,1 cho SOLID226 trong v18.1 có thể là "concentration", nhưng trong R2024 có thể là "Force-X" của structural DOF.

**Cách kiểm tra:** trong ANSYS R2024 GUI, vào *Help ▸ Element Reference ▸ SOLID226 ▸ Output Data ▸ Table 226.x (NMISC + SMISC)*, tìm cột "Concentration".

**Tạm thời:** thay bằng `PLNSOL,CONC` để lấy nodal solution, rồi convert sang element bằng `ETABLE,CCONC,NMISC,n` với n đúng.

### B.12 `tref,25` (line 53) vs `TREF,26` (line 310)

Bạn set hai lần với hai giá trị khác nhau. Lệnh sau đè lên lệnh trước → cuối cùng dùng 26. Nhưng nếu một số lệnh khác đọc TREF giữa hai lệnh này (như `BETA` cho diffusion expansion?), sẽ có sai số. **Để 26 ở cả hai chỗ** cho nhất quán với `Tinf=26`.

### B.13 Kết quả hậu xử lý dùng `ESEL,S,LIVE,,1` lạ

Cú pháp đúng là `ESEL,S,LIVE` (không có tham số sau). Phần `,,1` không gây lỗi nhưng không có tác dụng. Để gọn:
```
ESEL,S,LIVE
```

### B.14 `OUTRES,ALL,1` ghi mọi substep

Với mô phỏng dài (16 triệu giây, hàng trăm substep), `OUTRES,ALL,1` sẽ tạo file `.rst` **rất lớn** (vài chục GB). Có thể là một nguyên nhân ANSYS chết do hết dung lượng đĩa.

**Sửa:** chỉ ghi result cho substep cuối của mỗi LS:
```
OUTRES,ALL,LAST
OUTRES,ESOL,LAST
OUTRES,NSOL,LAST
```
Nếu cần lưu lịch sử để vẽ đồ thị Fig. 12-13, dùng `*GET` + array trong vòng *DO thay vì lưu .rst.

---

## C. Thứ tự fix — **làm đúng 6 việc dưới đây trước khi chạy lại**

1. Sửa `/filename` → `/FILNAME,Dong,1` ngay đầu file.
2. Bỏ `VLSCAL × 0.01`, đổi `ARSCALE × 1000` thành `× 100`.
3. Đổi BC điện áp/dòng từ `ASEL,…,13` và `DA,11` sang chọn theo toạ độ (NSEL,LOC).
4. Thay `D,ALL,…,TEMP/CONC` (line 319-320) bằng `IC,ALL,TEMP,26` và `IC,ALL,CONC,1`. Bỏ luôn LS1 và `TIMINT,OFF/ON`.
5. Chia LS3 thành 3 sub-load step (ramp 1 s → 1e6 s → 16.3e6 s) như mục B.7.
6. Trong vòng `*DO`, thêm `ANTYPE,,REST` ngay sau `/SOLU`, và kiểm tra `nKill > 0` trước khi `EKILL`.

Sau đó chạy thử lại. Mở `Dong.out` ngay sau khi chạy; nếu fail, gửi lại đoạn `*** ERROR ***` đầu tiên cho tôi.

---

## D. Một check rất nhanh trước khi solve

Sau PREP7, trước /SOLU, chạy:
```
ALLSEL
CHECK
```
Lệnh `CHECK` của ANSYS sẽ chỉ ra ngay những area thiếu material, element type chưa gán, hoặc node trùng lặp.

---

## E. Ghi chú về license / phiên bản

`TB,MIGR` (atomic migration material model) đã có từ ANSYS 18.1, nhưng cần license **Mechanical Enterprise** hoặc **Multiphysics**. Nếu chỉ có **Mechanical Pro / Mechanical Premium thông thường / Academic Teaching**, lệnh sẽ bị từ chối với:
```
*** ERROR *** The MIGR material model is not allowed by your current license.
```
Đó cũng là một khả năng giải thích vì sao 3 file `ansabort1-3.err` chỉ có banner — license fail làm script chết ngay từ lệnh `tb,migr,1`.

**Cách kiểm tra license:** mở ANSYS với `–p ane3fl` (Mechanical Enterprise) hoặc kiểm tra trong **Tools ▸ License Preferences** xem có check vào "Multiphysics" không.

---

Khi bạn fix xong 6 mục trong phần C và phần A (jobname), gửi lại file `Dong.err` mới — tôi sẽ giúp đọc tiếp.
