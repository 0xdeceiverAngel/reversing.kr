# reversing kr write up
## Easy_CrackMe
就是呢 上圖自己看
只是呢 不知道why `<&GetDlgItemTextA>` 字串進去 前面會空4個 value 應該是殘值

![](https://github.com/0xdeciverAngel/reversing.kr/raw/master/Easy_CrackMe.png)

Ea5yR3versing


## Easy_KeygenMe
上 ida 
這樣就OK拉 
![](https://github.com/0xdeciverAngel/reversing.kr/raw/master/Easy_KeygenMe.png)
```python
key=[0x10,0x20,0x30]
serial="5B134977135E7D13"
serial=bytes.fromhex(serial).decode('utf-8')
print(serial)
name=""
for i in range(0,len(serial)):
    name+=chr(ord(serial[i])^key[i%3])
print(name)
```

K3yg3nm3
## Easy_UnpackMe
脫殼脫起來
>OEP 的全名是Original Entry Point．表示程式的入口點．
如果程式有加殼的話，會改變真正的OEP．要透過脫殼才可以找到真正的OEP(來源網上)

丟ida或是動態都可以
丟動態要多一點耐心

![](https://github.com/0xdeciverAngel/reversing.kr/raw/master/Easy_UnpackMe.png)

401150