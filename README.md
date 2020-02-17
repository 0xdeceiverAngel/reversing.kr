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

## Music_Player

這題是用VB寫的
函數根本不熟 ㄏㄏ
動態分析一下

有兩個地方要過
第一個是 1min 限制
第二個是 error視窗

第一個改成 
>00404563 || cmp eax,EA60                 ;0xEA60==6000
00404568 || mov dword ptr ss:[ebp-18],eax  
0040456B || jmp music_player1.4045FE     

第二個地方 因為我用x32dbg
會直接跳到ntdll裡面
右鍵-前往-上一個引用
找到
>004046B9 | call dword ptr ds:[<&__vbaHresultCheckObj>]  

直接用nop填掉

之後重新patch存檔 重新執行就看到了


LIstenCare
## replace 
![](https://github.com/0xdeciverAngel/reversing.kr/raw/master/replace1.png)

>eax存到 0x4084d0
不重要運算
+1
+1
+0x601605C7
不重要運算
+1
+1

![](https://github.com/0xdeciverAngel/reversing.kr/raw/master/replace2.png)

加起來等於 0x601605cb

0x401071 0x401072 要改成 nop 不然跳不到 correct

又因為 

把0x4084d0 值取出來到 eax
所以我們 可以`輸入+0x601605cb==0x401071` 讓eax 變成 0x401071 就可以改成nop 

又剛好執行2次 71 72 都可以改成nop

>python 0x401072-0x601605CB+0xFFFFFFFF==2687109798

這邊要 bufflow

2687109798

註:
![](https://github.com/0xdeciverAngel/reversing.kr/raw/master/replace3.png)

## ImagePrc
打開就是可以畫畫
問題是你還要畫到跟byte 都一樣 

先用動態 找到 `wrong`字串出現在 是 0x4013DB

用ida 跳到位置上 decompile

```
int __stdcall sub_401130(HWND hWnd, UINT Msg, WPARAM wParam, LPARAM lParam)
{
  HDC v4; // eax
  int result; // eax
  HGDIOBJ v6; // eax
  HDC v7; // esi
  void *v8; // esi
  HRSRC v9; // eax
  HGLOBAL v10; // eax
  _BYTE *v11; // eax
  signed int v12; // edi
  _BYTE *v13; // ecx
  int v14; // eax
  char pv; // [esp+8h] [ebp-80h]
  LONG v16; // [esp+Ch] [ebp-7Ch]
  UINT cLines; // [esp+10h] [ebp-78h]
  struct tagBITMAPINFO bmi; // [esp+20h] [ebp-68h]

  if ( Msg <= 0x111 )
  {
    if ( Msg != 273 )
    {
      switch ( Msg )
      {
        case 1u:
          v7 = GetDC(hWnd);
          hbm = CreateCompatibleBitmap(v7, 200, 150);  //size  
          hdc = CreateCompatibleDC(v7);
          h = SelectObject(hdc, hbm);
          Rectangle(hdc, -5, -5, 205, 205);
          ReleaseDC(hWnd, v7);
          ::wParam = (WPARAM)CreateFontA(12, 0, 0, 0, 400, 0, 0, 0, 0x81u, 0, 0, 0, 0x12u, pszFaceName);
          dword_4084E0 = (int)CreateWindowExA(
                                0,
                                ClassName,
                                WindowName,
                                0x50000000u,
                                60,
                                85,
                                80,
                                28,
                                hWnd,
                                (HMENU)0x64,
                                hInstance,
                                0);
          SendMessageA((HWND)dword_4084E0, 0x30u, ::wParam, 0);
          return 0;
        case 2u:
          v6 = SelectObject(hdc, h);
          DeleteObject(v6);
          DeleteDC(hdc);
          PostQuitMessage(0);
          return 0;
        case 0xFu:
          v4 = BeginPaint(hWnd, (LPPAINTSTRUCT)bmi.bmiColors);
          BitBlt(v4, 0, 0, 200, 150, hdc, 0, 0, 0xCC0020u);
          EndPaint(hWnd, (const PAINTSTRUCT *)bmi.bmiColors);
          return 0;
      }
      return DefWindowProcA(hWnd, Msg, wParam, lParam);
    }
    if ( wParam == 100 )
    {
      GetObjectA(hbm, 24, &pv);
      memset(&bmi, 0, 0x28u);
      bmi.bmiHeader.biHeight = cLines;
      bmi.bmiHeader.biWidth = v16;
      bmi.bmiHeader.biSize = 40;
      bmi.bmiHeader.biPlanes = 1;
      bmi.bmiHeader.biBitCount = 24;
      bmi.bmiHeader.biCompression = 0;
      GetDIBits(hdc, (HBITMAP)hbm, 0, cLines, 0, &bmi, 0);
      v8 = operator new(bmi.bmiHeader.biSizeImage);
      GetDIBits(hdc, (HBITMAP)hbm, 0, cLines, v8, &bmi, 0);
      v9 = FindResourceA(0, (LPCSTR)0x65, (LPCSTR)0x18);
      v10 = LoadResource(0, v9);
      v11 = LockResource(v10);
      v12 = 0;
      v13 = v8;
      v14 = v11 - (_BYTE *)v8;
      while ( *v13 == v13[v14] )
      {
        ++v12;
        ++v13;
        if ( v12 >= 90000 )
        {
          sub_401500(v8);
          return 0;
        }
      }
      MessageBoxA(hWnd, Text, Caption, 0x30u);
      sub_401500(v8);
      return 0;
    }
    return 0;
  }
  switch ( Msg )
  {
    case 0x200u:
      if ( dword_47D7F8 )
      {
        MoveToEx(hdc, x, y, 0);
        LineTo(hdc, (unsigned __int16)lParam, (unsigned int)lParam >> 16);
        x = (unsigned __int16)lParam;
        y = (unsigned int)lParam >> 16;
        InvalidateRect(hWnd, 0, 0);
      }
      return 0;
    case 0x201u:
      dword_47D7F8 = 1;
      y = (unsigned int)lParam >> 16;
      x = (unsigned __int16)lParam;
      result = 0;
      break;
    case 0x202u:
      dword_47D7F8 = 0;
      result = 0;
      break;
    default:
      return DefWindowProcA(hWnd, Msg, wParam, lParam);
  }
  return result;
}
```
然後用 reshacker 把圖片的hex 抓出來
開小畫家 先存一個size 一樣的bmp 再用 hexeditor 把hex 填回去 就會得到圖片

或是 可以跟進去
![](https://github.com/0xdeciverAngel/reversing.kr/raw/master/imageprc.png)

從 ecx 或是 eax+ecx 撈資料

GOT
