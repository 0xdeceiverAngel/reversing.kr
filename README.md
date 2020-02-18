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

## Position
先看一下string ida 要設定
![](https://github.com/0xdeciverAngel/reversing.kr/raw/master/ida_string_setup.png)

點string 她會跳到rodata 再ctrl+x 跳到 `ref to` 然後decompile

```
signed int __stdcall sub_401740(int a1)
{
  int v1; // edi
  int v3; // esi
  int v4; // esi
  __int16 v5; // bx
  unsigned __int8 v6; // al
  unsigned __int8 v7; // ST2C_1
  unsigned __int8 v8; // al
  unsigned __int8 v9; // bl
  wchar_t *v10; // eax
  __int16 v11; // di
  wchar_t *v12; // eax
  __int16 v13; // di
  wchar_t *v14; // eax
  __int16 v15; // di
  wchar_t *v16; // eax
  __int16 v17; // di
  wchar_t *v18; // eax
  __int16 v19; // di
  unsigned __int8 v20; // al
  unsigned __int8 v21; // ST2C_1
  unsigned __int8 v22; // al
  unsigned __int8 v23; // bl
  wchar_t *v24; // eax
  __int16 v25; // di
  wchar_t *v26; // eax
  __int16 v27; // di
  wchar_t *v28; // eax
  __int16 v29; // di
  wchar_t *v30; // eax
  __int16 v31; // di
  wchar_t *v32; // eax
  __int16 v33; // si
  unsigned __int8 v34; // [esp+10h] [ebp-28h]
  unsigned __int8 v35; // [esp+10h] [ebp-28h]
  unsigned __int8 v36; // [esp+11h] [ebp-27h]
  unsigned __int8 v37; // [esp+11h] [ebp-27h]
  unsigned __int8 v38; // [esp+13h] [ebp-25h]
  unsigned __int8 v39; // [esp+13h] [ebp-25h]
  unsigned __int8 v40; // [esp+14h] [ebp-24h]
  unsigned __int8 v41; // [esp+14h] [ebp-24h]
  unsigned __int8 v42; // [esp+19h] [ebp-1Fh]
  unsigned __int8 v43; // [esp+19h] [ebp-1Fh]
  unsigned __int8 v44; // [esp+1Ah] [ebp-1Eh]
  unsigned __int8 v45; // [esp+1Ah] [ebp-1Eh]
  unsigned __int8 v46; // [esp+1Bh] [ebp-1Dh]
  unsigned __int8 v47; // [esp+1Bh] [ebp-1Dh]
  unsigned __int8 v48; // [esp+1Ch] [ebp-1Ch]
  unsigned __int8 v49; // [esp+1Ch] [ebp-1Ch]
  int name; // [esp+20h] [ebp-18h]
  int serial; // [esp+24h] [ebp-14h]
  char ans; // [esp+28h] [ebp-10h]
  int v53; // [esp+34h] [ebp-4h]

  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&name);
  v1 = 0;
  v53 = 0;
  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&serial);
  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&ans);
  LOBYTE(v53) = 2;
  CWnd::GetWindowTextW(a1 + 304, &name);
  if ( *(_DWORD *)(name - 12) == 4 )            // name 長度 4
  {
    v3 = 0;
    while ( (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&name, v3) >= 'a'
         && (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&name, v3) <= 'z' )
    {
      if ( ++v3 >= 4 )
      {
LABEL_7:
        v4 = 0;
        while ( 1 )
        {
          if ( v1 != v4 )
          {
            v5 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&name, v4);
            if ( (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&name, v1) == v5 )
              goto LABEL_2;
          }
          if ( ++v4 >= 4 )
          {
            if ( ++v1 < 4 )
              goto LABEL_7;
            CWnd::GetWindowTextW(a1 + 420, &serial);
            if ( *(_DWORD *)(serial - 12) == 11
              && (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 5) == 45 )
            {
              v6 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&name, 0);
              v7 = (v6 & 1) + 5;
              v48 = ((v6 >> 4) & 1) + 5;
              v42 = ((v6 >> 1) & 1) + 5;
              v44 = ((v6 >> 2) & 1) + 5;
              v46 = ((v6 >> 3) & 1) + 5;
              v8 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&name, 1);
              v34 = (v8 & 1) + 1;
              v40 = ((v8 >> 4) & 1) + 1;
              v36 = ((v8 >> 1) & 1) + 1;
              v9 = ((v8 >> 2) & 1) + 1;
              v38 = ((v8 >> 3) & 1) + 1;
              v10 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
              itow_s(v7 + v9, v10, 0xAu, 10);
              v11 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0);
              if ( (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 0) == v11 )
              {
                ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                v12 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                itow_s(v46 + v38, v12, 0xAu, 10);
                v13 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 1);
                if ( v13 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                {
                  ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                  v14 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                  itow_s(v42 + v40, v14, 0xAu, 10);
                  v15 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 2);
                  if ( v15 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                  {
                    ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                    v16 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                    itow_s(v44 + v34, v16, 0xAu, 10);
                    v17 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 3);
                    if ( v17 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                    {
                      ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                      v18 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                      itow_s(v48 + v36, v18, 0xAu, 10);
                      v19 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 4);
                      if ( v19 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                      {
                        ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                        v20 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&name, 2);
                        v21 = (v20 & 1) + 5;
                        v49 = ((v20 >> 4) & 1) + 5;
                        v43 = ((v20 >> 1) & 1) + 5;
                        v45 = ((v20 >> 2) & 1) + 5;
                        v47 = ((v20 >> 3) & 1) + 5;
                        v22 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&name, 3);
                        v35 = (v22 & 1) + 1;
                        v41 = ((v22 >> 4) & 1) + 1;
                        v37 = ((v22 >> 1) & 1) + 1;
                        v23 = ((v22 >> 2) & 1) + 1;
                        v39 = ((v22 >> 3) & 1) + 1;
                        v24 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                        itow_s(v21 + v23, v24, 0xAu, 10);
                        v25 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 6);
                        if ( v25 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                        {
                          ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                          v26 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                          itow_s(v47 + v39, v26, 0xAu, 10);
                          v27 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 7);
                          if ( v27 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                          {
                            ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                            v28 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                            itow_s(v43 + v41, v28, 0xAu, 10);
                            v29 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 8);
                            if ( v29 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                            {
                              ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                              v30 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                              itow_s(v45 + v35, v30, 0xAu, 10);
                              v31 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 9);
                              if ( v31 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                              {
                                ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                                v32 = (wchar_t *)ATL::CSimpleStringT<wchar_t,1>::GetBuffer(&ans);
                                itow_s(v49 + v37, v32, 0xAu, 10);
                                v33 = ATL::CSimpleStringT<wchar_t,1>::GetAt(&serial, 10);
                                if ( v33 == (unsigned __int16)ATL::CSimpleStringT<wchar_t,1>::GetAt(&ans, 0) )
                                {
                                  ATL::CSimpleStringT<wchar_t,1>::ReleaseBuffer(&ans, -1);
                                  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::~CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&ans);
                                  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::~CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&serial);
                                  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::~CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&name);
                                  return 1;
                                }
                              }
                            }
                          }
                        }
                      }
                    }
                  }
                }
              }
            }
            goto LABEL_2;
          }
        }
      }
    }
  }
LABEL_2:
  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::~CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&ans);
  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::~CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&serial);
  ATL::CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>::~CStringT<wchar_t,StrTraitMFC_DLL<wchar_t,ATL::ChTraitsCRT<wchar_t>>>(&name);
  return 0;
}
```
我知道看起來有點硬

```python
serial="76876-77776"
name=""
for i in range(ord('a'),ord('z')+1):
    for j in range(ord('a'),ord('z')+1):
        v6 = i
        v8 = j
        v7 = (v6 & 1) + 5
        v48 = ((v6 >> 4) & 1) + 5
        v42 = ((v6 >> 1) & 1) + 5
        v44 = ((v6 >> 2) & 1) + 5
        v46 = ((v6 >> 3) & 1) + 5
        
        v34 = (v8 & 1) + 1
        v40 = ((v8 >> 4) & 1) + 1
        v36 = ((v8 >> 1) & 1) + 1
        v9 = ((v8 >> 2) & 1) + 1
        v38 = ((v8 >> 3) & 1) + 1
        if (v7 + v9 == int(serial[0])) and (v46 + v38 == int(serial[1])) and (v42 + v40 == int(serial[2])) and (v44 + v34 == int(serial[3])) and (v48 + v36 == int(serial[4])):
            print (chr(i),chr(j))
print('------------')            
for i in range(ord('a'),ord('z')+1):
    for j in range(ord('a'),ord('z')+1):
        v20=i
        v22=j
		
        v21 = (v20 & 1) + 5
        v49 = ((v20 >> 4) & 1) + 5
        v43 = ((v20 >> 1) & 1) + 5
        v45 = ((v20 >> 2) & 1) + 5
        v47 = ((v20 >> 3) & 1) + 5
		
        v35 = (v22 & 1) + 1
        v41 = ((v22 >> 4) & 1) + 1
        v37 = ((v22 >> 1) & 1) + 1
        v23 = ((v22 >> 2) & 1) + 1
        v39 = ((v22 >> 3) & 1) + 1
		
        if v21 + v23 == int(serial[6]) and v47 + v39 == int(serial[7]) and v43 + v41 == int(serial[8]) and v45 + v35 == int(serial[9]) and v49 + v37 == int(serial[10]):
            print (chr(i),chr(j))
'''
b u
c q
f t
g p
------------
a y
b m
c i
e x
f l
g h
h u
i q
j e
k a
l t
m p
n d
'''
```
題目的readme 有提示說 結尾是p

所以答案 
>bump
cqmp
ftmp
gpmp
