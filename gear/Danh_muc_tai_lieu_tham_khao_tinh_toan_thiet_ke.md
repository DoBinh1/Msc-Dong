# DANH MỤC TÀI LIỆU THAM KHẢO

*Tài liệu tham khảo cho tính toán thiết kế bộ thí nghiệm phát hiện vết nứt bánh răng. Trích dẫn theo số thứ tự [1], [2]… trong báo cáo.*

---

## 1. Giáo trình & sổ tay thiết kế

[1] Trịnh Chất, Lê Văn Uyển. *Tính toán thiết kế hệ dẫn động cơ khí*, Tập 1–2. NXB Giáo dục.
— Quy trình chuẩn: chọn động cơ, trục, then, ổ lăn, khớp nối.

[2] Nguyễn Trọng Hiệp. *Chi tiết máy*, Tập 1–2. NXB Giáo dục.
— Cơ sở lý thuyết độ bền trục, ổ, bánh răng.

[3] R. G. Budynas, J. K. Nisbett. *Shigley's Mechanical Engineering Design*, McGraw-Hill.
— Thiết kế trục, kiểm bền mỏi, ổ lăn, bánh răng.

[4] Ninh Đức Tốn. *Dung sai và lắp ghép*. NXB Giáo dục.
— Lắp ghép k6/H7, dung sai hình học, độ đảo.

---

## 2. Tiêu chuẩn thiết kế

[5] ISO 6336. *Calculation of load capacity of spur and helical gears* — kiểm bền răng.

[6] ISO 1328. *Cylindrical gears — ISO tolerance classification* — cấp chính xác, backlash.

[7] ISO 281. *Rolling bearings — Dynamic load ratings and rating life* — tuổi thọ ổ lăn.

[8] ISO 286. *Hệ thống dung sai lắp ghép* — kiểu lắp ổ–trục–vỏ.

[9] ISO 21940-11. *Rotor balancing — Procedures and tolerances* — cân bằng động rotor.

---

## 3. Bánh răng & vết nứt (TVMS)

[10] Yang, D.C.H., Lin, J.Y. (1987). *Hertzian damping, tooth friction and bending elasticity in gear impact dynamics*. ASME J. Mech. Des., 109(2), 189–196.
— Nền tảng phương pháp thế năng tính TVMS.

[11] Mohammed, O.D., Rantatalo, M., Aidanpää, J.O. (2013). *Improving mesh stiffness calculation of cracked gears for vibration-based fault analysis*. Eng. Failure Analysis, 34, 235–251.
— Tài liệu cốt lõi về TVMS bánh răng nứt.

[12] Sainsot, P., Velex, P., Duverger, O. (2004). *Contribution of gear body to tooth deflections*. ASME J. Mech. Des., 126(4), 748–752.
— Công thức độ cứng nền răng kf.

[13] Chen, Z., Shao, Y. (2011). *Dynamic simulation of spur gear with tooth root crack…*. Eng. Failure Analysis, 18, 2149–2164.
— Mô hình vết nứt 3D theo chiều rộng răng.

[14] Jiang, F., et al. (2024). *Two novel indicators for gear crack diagnosis based on vibration responses*. Mechanism and Machine Theory.
— Đường nứt parabol thực tế; chỉ số chẩn đoán mới.

---

## 4. Động lực học & phân tích rung

[15] S. S. Rao. *Mechanical Vibrations*, Pearson.
— Tần số riêng, tốc độ tới hạn, mô hình dao động.

[16] G. Genta. *Dynamics of Rotating Systems*, Springer.
— Rotordynamics: ảnh hưởng độ cứng ổ/vỏ tới tần số riêng.

[17] *Harris' Shock and Vibration Handbook*, McGraw-Hill.
— Cách rung: truyền suất, độ võng tĩnh, chọn gối.

[18] R. B. Randall. *Vibration-based Condition Monitoring*, Wiley.
— TSA, FFT, cepstrum, giải điều chế chẩn đoán.

[19] Zakrajsek, J.J., et al. (1993). *Gear fault detection methods*. NASA TM-105950.
— Chỉ số NA4, FM0/FM4.

---

## 5. Phần mềm & bộ dữ liệu

[20] ANSYS Mechanical — *Theory Reference & Fracture Tools*.
— Kiểm chứng TVMS bằng FEM; mô phỏng phát triển nứt.

[21] MATLAB *Signal Processing Toolbox* & Simulink — mô hình 6-DOF, xử lý tín hiệu.

[22] Python: `PyEMD`, `vmdpy`, `scipy.signal`, `scikit-learn` — EMD/VMD, pipeline ML.

[23] Bộ dữ liệu công khai: PHM Society 2009 Gearbox; UNSW Gear Vibration Database; Mendeley Data (Mohammed 2013).

---

> **Lưu ý:** Khi trích dẫn tiêu chuẩn ISO/AGMA, ghi rõ năm phiên bản hiện hành; nên kèm TCVN tương đương khi nộp tại Việt Nam.
