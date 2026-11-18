# Thông tin học viên

- Họ và tên: Đào Duy Hiếu
- MSHV: 2A202603235
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: HIEU2460
- Repo bài nộp (URL): https://github.com/HIEU2460/K5-L23-Day08-DaoDuyHieu-2A202603235-AI-Assisted-Labeling.git
- Commit chốt bài (hash): a9b2d30a5e8e22c922c312680cd508ae8b50e04d
- Máy: Windows, máy local không dùng CUDA; model chạy bằng CPU trên Google Colab
- CVAT (Docker): CVAT local bằng Docker, không lưu lại số phiên bản lúc làm bài

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 314 / 18 / 33 (mẫu số: 365 box máy); added: 118 (mẫu số: 450 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 72.4% / 91.3% (46 đối tượng gold)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 3.75 phút/ảnh; duyệt có pre-label 3.00 phút/ảnh
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/85)
- **Quyết định:** Rework — bắt bẫy 8/12 chưa đạt ngưỡng 10/12, nên chưa đủ tin cậy để Release
