kiểm tra file thấy đây là file thực thi PE32 của windows
![](file.png)

thực hiện chạy thử file thấy file yêu cầu nhập password, nếu nhập sai thì sẽ hiện ra `oh, no!`
![](run.png)

load file vào ida32
### IDA
hàm main của chương trình 
![](ida1.png)
- chương trình yêu cầu nhập vào tối thiểu `294`ký tự của input rồi input được truyền tham số vào hàm `encrypt`


#### hàm `encrypt` 
- hàm `encrypt` nhận 1 tham số là input nhập vào 
![](ida2.png)
- hàm `encrypt` gọi hàm `dump_shellcode` 122 lần với tham số truyền vào trong mảng `v3`


#### hàm `dump_shellcode`
- biến `a1` có thể là `1`, `2`, `3`, `4` là 4 giá trị của switch để xor ra shellcode trong 4 trường hợp, debug tại đây để dump ra 4 đoạn shellcode 
- biến `a2` là vị trí của inp
- biến `a3` là input sau khi đã bị được encrypt  

#### `shellcode 1`
shellcode 1 encode input bằng cách xor
![](shellcode1.png)

#### `shellcode 2`
shellcode 2 thực hiện ghép 2 ký tự liền kề của input lại làm 1 word rồi dịch bit rồi xor vơi `0x1693`
![](shellcode2.png)

#### `shellcode 3`
shellcode 3 thực hiện encode input bằng base64 với custom mapping table `ABDCEHGFIJKLUNOPYRTSMVWXQZajcdefohibkmlngpqrstuv4xzy8123w56709+/`
![](shellcode3.png)


#### `shellcode 4`
shellcode 4 thực hiện encrypt input bằng thuật toán `RC4` với key là `susan`
![](shellcode4_1.png)
![](shellcode4_2.png)
![](shellcode4_3.png)

từ 4 shellcode trên, viết được payload để decrypt ra input nhập vào 
``` python 
import base64
from Crypto.Cipher import ARC4

shellcode_enc = [0x00000001, 0x00000000, 0xCCCCCC74, 0x00000001, 0x00000001, 0xCCCCCC48, 0x00000001, 0x00000002, 0xCCCCCC3B, 0x00000002, 0x00000088, 0xCCCC2481, 0x00000003, 0x0000006F, 0x6F593363, 0x00000002, 0x00000084, 0xCCCC0101, 0x00000002, 0x0000000A, 0xCCCCAF35, 0x00000001, 0x000000D0, 0xCCCCCC33, 0x00000003, 0x0000000F, 0x76784D64, 0x00000004, 0x00000012, 0x1AB120DD, 0x00000001, 0x00000106, 0xCCCCCC0C, 0x00000003, 0x000000E8, 0x3542446A, 0x00000002, 0x0000001D, 0xCCCC21A6, 0x00000002, 0x0000001F, 0xCCCC8ABE, 0x00000001, 0x00000021, 0xCCCCCC4C, 0x00000002, 0x00000022, 0xCCCC0E26, 0x00000001, 0x00000024, 0xCCCCCC35, 0x00000001, 0x0000005D, 0xCCCCCC3B, 0x00000003, 0x0000002B, 0x7539456A, 0x00000004, 0x00000016, 0x0DED3F88, 0x00000003, 0x000000EB, 0x7A56336A, 0x00000002, 0x00000032, 0xCCCCAF35, 0x00000002, 0x00000030, 0xCCCC2FAB, 0x00000002, 0x00000008, 0xCCCC3681, 0x00000003, 0x00000034, 0x75636C6A, 0x00000001, 0x00000037, 0xCCCCCC00, 0x00000002, 0x00000038, 0xCCCC3C25, 0x00000003, 0x00000053, 0x67524649, 0x00000001, 0x0000003E, 0xCCCCCC21, 0x00000001, 0x0000003F, 0xCCCCCC54, 0x00000001, 0x00000092, 0xCCCCCC37, 0x00000001, 0x00000086, 0xCCCCCC74, 0x00000004, 0x00000109, 0x13FD36C0, 0x00000001, 0x0000004A, 0xCCCCCC00, 0x00000001, 0x0000010D, 0xCCCCCC52, 0x00000004, 0x0000004D, 0x43BA3DC1, 0x00000003, 0x0000011C, 0x6F773264, 0x00000004, 0x0000003A, 0x0CB073CD, 0x00000004, 0x00000056, 0x06B973CD, 0x00000003, 0x0000005A, 0x796D3251, 0x00000004, 0x0000010F, 0x10AE36CB, 0x00000001, 0x0000005E, 0xCCCCCC3D, 0x00000002, 0x0000005F, 0xCCCC2101, 0x00000004, 0x00000068, 0x06B52788, 0x00000001, 0x000000BA, 0xCCCCCC44, 0x00000004, 0x0000009A, 0x06BA3ADC, 0x00000003, 0x0000006C, 0x6D4A4649, 0x00000004, 0x00000004, 0x43AE6288, 0x00000004, 0x00000072, 0x0EFD20C1, 0x00000002, 0x000000C7, 0xCCCC863D, 0x00000001, 0x0000007A, 0xCCCCCC2B, 0x00000004, 0x0000007B, 0x0DB82788, 0x00000001, 0x000000C6, 0xCCCCCC0E, 0x00000001, 0x00000083, 0xCCCCCC2B, 0x00000001, 0x00000003, 0xCCCCCC01, 0x00000004, 0x00000093, 0x13BC2388, 0x00000004, 0x000000A9, 0x43B23788, 0x00000003, 0x0000000C, 0x436D475A, 0x00000004, 0x0000008A, 0x11BC36CE, 0x00000004, 0x0000008E, 0x11BC73DB, 0x00000004, 0x000000AF, 0x0AB527D1, 0x00000002, 0x00000044, 0xCCCCA525, 0x00000001, 0x00000097, 0xCCCCCC37, 0x00000002, 0x00000098, 0xCCCC2F01, 0x00000004, 0x00000061, 0x02FD3CDC, 0x00000001, 0x0000009E, 0xCCCCCC52, 0x00000004, 0x0000009F, 0x3AFD7DDB, 0x00000004, 0x00000046, 0x17B126CB, 0x00000001, 0x000000A7, 0xCCCCCC33, 0x00000001, 0x0000002F, 0xCCCCCC00, 0x00000001, 0x00000087, 0xCCCCCC48, 0x00000001, 0x000000AD, 0xCCCCCC33, 0x00000001, 0x000000AE, 0xCCCCCC4E, 0x00000004, 0x000000C9, 0x00FD26C7, 0x00000002, 0x000000B3, 0xCCCCA122, 0x00000001, 0x000000B5, 0xCCCCCC00, 0x00000004, 0x000000B6, 0x43A83CD1, 0x00000003, 0x00000065, 0x73593351, 0x00000001, 0x000000BB, 0xCCCCCC37, 0x00000001, 0x000000BC, 0xCCCCCC31, 0x00000004, 0x000000D2, 0x0CA973DC, 0x00000003, 0x000000BF, 0x3842545A, 0x00000002, 0x000000C2, 0xCCCC2181, 0x00000001, 0x000000DA, 0xCCCCCC4E, 0x00000004, 0x0000007F, 0x17B430C9, 0x00000004, 0x00000076, 0x0FB821CD, 0x00000004, 0x000000FB, 0x11AD73CD, 0x00000002, 0x000000CD, 0xCCCC26A6, 0x00000001, 0x000000CF, 0xCCCCCC00, 0x00000002, 0x0000004B, 0xCCCC2C25, 0x00000001, 0x000000D1, 0xCCCCCC31, 0x00000002, 0x000000BD, 0xCCCC22A3, 0x00000003, 0x00000113, 0x796D4749, 0x00000002, 0x000000C4, 0xCCCCA426, 0x00000003, 0x000000EF, 0x6C6D476A, 0x00000004, 0x000000DC, 0x0DBC73CD, 0x00000001, 0x000000EE, 0xCCCCCC00, 0x00000004, 0x000000E4, 0x0CAF27C6, 0x00000003, 0x0000001A, 0x5539315A, 0x00000001, 0x0000002E, 0xCCCCCC35, 0x00000004, 0x000000E0, 0x0CBE73CC, 0x00000001, 0x000000DB, 0xCCCCCC35, 0x00000002, 0x000000F2, 0xCCCCA48C, 0x00000004, 0x000000F4, 0x07B33288, 0x00000002, 0x00000025, 0xCCCC39A7, 0x00000004, 0x00000040, 0x05B43788, 0x00000001, 0x000000FF, 0xCCCCCC3D, 0x00000003, 0x00000100, 0x6B563251, 0x00000003, 0x00000103, 0x6D4A5864, 0x00000004, 0x00000027, 0x3CBA1DC7, 0x00000002, 0x00000107, 0xCCCC062B, 0x00000004, 0x000000A3, 0x00FD26C7, 0x00000001, 0x000000A8, 0xCCCCCC4E, 0x00000001, 0x0000010E, 0xCCCCCC3D, 0x00000003, 0x000000F8, 0x67524649, 0x00000004, 0x000000D6, 0x02B53088, 0x00000002, 0x00000116, 0xCCCC8625, 0x00000004, 0x00000118, 0x0CFD20DC, 0x00000002, 0x00000051, 0xCCCCA2A8, 0x00000003, 0x0000011F, 0x33566C63, 0x00000003, 0x00000122, 0x6B4A5851, 0x00000001, 0x00000125, 0xCCCCCC0E]
temp = [0] * 366
flag = [0]*294

def custom_base64(x):
    x = x.to_bytes(4,"little")
    x = x.decode("utf-8")
    std_base64chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    custom = "ABDCEHGFIJKLUNOPYRTSMVWXQZajcdefohibkmlngpqrstuv4xzy8123w56709+/"
    y = bytes(str(x).translate(str(x)[2:-1].maketrans(custom, std_base64chars)), 'utf-8')
    x = base64.b64decode(y)
    return x.decode()

def decode(a, b, c):
    global flag
    d = 0
    if a == 1:
        aa = ((c & 0xFF)^ 0x20) 
        if aa % 2 == 0:
            flag[b] = aa
        else:
            aa = ((c & 0xFF)^ 0x52)
            flag[b] = aa
    if a == 2:
        d = c & 0xFFFF
        for i in range(5,0, -1):
            d = d ^ 0x1693
            d = (d << (16 - i)) | (d >> i)
            d = d & 0xFFFF
        dd = d >> 8
        flag[b] = dd 
        flag[b + 1] = (d & 0xff)
    if a == 3:
        a3 = custom_base64(c)
        for i in range(len(a3)):
            flag[b + i] = ord(a3[i])

    if a == 4:
        key = b"susan"
        cipher = ARC4.new(key)
        c = c.to_bytes(4,"little")
        ddd = cipher.decrypt(c)
        for i in range(len(ddd)):
            flag[b + i] = ddd[i]
        
for i in range(0, 365, 3):
    decode(shellcode_enc[i], shellcode_enc[i+1], shellcode_enc[i+2])

for i in range(len(flag)):
    print(chr(flag[i]),end='')
```

#### input: `ThiS 1s A rIdiCuLously l0ng_Lon9_l0ng_loNg_lOng strIng. The most difficult thing is the decision to act, the rest is merely tenacity. The fears are paper tigers. You can do anything you decide to do. You can act to change and control your life; and the procedure, the process is its own reward.`

từ input nhập vào, có được flag
#### flag: `vcstraining{Aw3s0me_D4ta_tran5Form4t1oN_Kak4}`






