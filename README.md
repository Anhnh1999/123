https://github.com/MicrosoftDocs/cpp-docs/blob/main/docs/windows/walkthrough-creating-windows-desktop-applications-cpp.md

## WinMain 
#### WNDCLASSEX struct 
- RegisterClassEx: đăng ký lớp cửa sổ 
- callback function `WndProc` - xử lý các message gửi tới cửa sổ   

#### CreateWindowEx
- tạo cửa sổ và gửi message `WM_CREATE` đến `WndProc`

#### hiển thị cửa sổ 
- sau khi tạo được cửa sổ thì sử dụng `ShowWindow` để hiển thị và `UpdateWindow` để gửi messeage `WM_PAINT` đến `WndProc`

#### DispatchMessage
- gửi message nhận được đến `WndProc`


## callback function WndProc
#### message WM_CREATE
- khởi tạo vị trí của hình tròn theo kích thước của cửa sổ 
- `GetClientRect` để lấy kích thước của cửa sổ, trả về 1 pointer RECT struct
#### message VM_PAINT 
https://docs.microsoft.com/en-us/windows/win32/gdi/using-filled-shapes
- sử dụng hàm Ellipse để vẽ hình tròn
- InvalidateRect để yêu cầu vẽ lại cửa sổ khi xử lý VM_PAINT message tiếp theo 
- sau khi vẽ sau hình tròn thì xử lý hướng bay và va chạm với cửa sổ với độ trễ 5s 
#### message WM_DESTROY
- xử lý khi cửa sổ bị hủy 
- sử dụng hàm PostQuitMessage với `nExitCode = 0`
#### DefWindowProc 
- xử lý các messeage khác 
