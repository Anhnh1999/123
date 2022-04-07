load file vào ida32
### IDA
hàm main của chương trình 

![](ida1.png)

- hàm main có 1 đoạn code xảy ra `exception divide by zero` và một đoạn code không hợp lệ nên để thực thi được phải patch 2 đoạn này bằng lệnh `nop`

![](raw1.png)

- patch đoạn code 

![](patch1.png)



#### hàm `DialogFunc` 
- phân tích callback function `DialogFunc` của `DialogBoxParamA` 

##### với message `WM_INITDIALOG`, luồng thực thi sẽ gặp 3 kỹ thuật anti-debug `IsDebuggerPresent`, `NtGlobalFlag` và `ProcessHeap` trong `PEB`

![](raw2.png)

- để debug được chương trình cần patch 3 kỹ thuật này lại 
    + `IsDebuggerPresent` nhảy tới `ExitProcess` nếu sử dụng debugger  
    + `NtGlobalFlag` kiểm tra nếu khác 0 thì sẽ nhảy tới `ExitProcess`
    + `ProcessHeap` kiểm tra nếu `Flag` không có `HEAP_GROWABLE`(0x00000002) và nếu ForceFlags nếu khác 0 thì sẽ phát hiện debugger và nhảy tới `ExitProcess`  
        []()https://www.apriorit.com/dev-blog/367-anti-reverse-engineering-protection-techniques-to-use-before-releasing-software#p4
    + ngoài ra ở hàm `sub_401757` kiểm tra xem opcode từ `loc_4011AC` có bị thay đổi hay không, patch luôn hàm này để có thể patch code đoạn sau mà không bị `ExitProcess`

- patch bằng cách `nop` đoạn code nhảy tới ExitProcess của cả 3 kỹ thuật 

![](patch2.png)

##### với message `WM_COMMAND`, luồng thực thi sẽ gặp kỹ thuật anti-debug sử dụng `NtQueryInformationProcess` với `ProcessInformationClass` là `ProcessDebugPort`

![](raw3.png)

- `NtQueryInformationProcess`  sẽ chuyển `ProcessInformation` thành giá trị khác 0 nếu có debugger 
- nếu `ProcessInformation` có giá trị khác 0 thì opcode tại `loc_4010A4` sẽ bị thay đổi bằng cách cộng từng byte với giá trị trong `ProcessInformation`
- để debug được thì sau khi gọi hàm `NtQueryInformationProcess` thì sửa đổi giá trị trong `ProcessInformation` thành 0 để opcode trong `loc_4010A4` không đổi
- luồng thực thi tiếp theo là đến lời gọi hàm `sub_401284`

#### hàm `sub_401284`
##### luồng thực thi đầu tiên sẽ gặp 1 kỹ thuật anti-debug sử dụng `OutputDebugStringA` và `GetLastError`
- nếu chạy trên win vista trở xuống thì sẽ bị check debugger với 2 hàm `OutputDebugStringA` và `GetLastError` vì vậy có thể patch hoặc chạy trên windows phiên bản cao để bypass
- sau kỹ thuật `OutputDebugStringA` và `GetLastError`, có thêm 1 kỹ thuật anti-debug sử dụng `NtSetInformationThread` gây crash chương trình nếu gặp breakpoint, bypass hàm này để tiếp tục debug chương trình 









