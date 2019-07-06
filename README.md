# LidLocker
Windows application that locks the laptop screen when the lid is closed.  This is useful for securing the desktop where applications need to continue running and "sleep" or "hibernate" are not desirable.

Compile the following program `lidlocker.c` and run as the logged on user at startup.<br />
`i686-w64-mingw32-gcc -mwindows -o lidlocker.exe lidlocker.c`

The power policy for "When I close the lid" should be set to "Do nothing".

```c
#include <windows.h>

GUID GLSC = {0xBA3E0F4D, 0xB817, 0x4094, 0xA2, 0xD1, 0xD5, 0x63, 0x79, 0xE6, 0xA0, 0xF3};

LRESULT CALLBACK LLWndProc(HWND, UINT, WPARAM, LPARAM);

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
{
   WNDCLASS wnd;
   LPSTR szClassName = "LidLockerClass";
   MSG msg;
   HWND hwnd;
   HPOWERNOTIFY hpn;

   wnd.style = CS_HREDRAW | CS_VREDRAW | WS_DISABLED;
   wnd.lpfnWndProc = LLWndProc;
   wnd.cbClsExtra = 0;
   wnd.cbWndExtra = 0;
   wnd.hInstance = hInstance;
   wnd.hIcon = NULL;
   wnd.hCursor = NULL;
   wnd.hbrBackground = (HBRUSH)(COLOR_BACKGROUND + 1);
   wnd.lpszMenuName = NULL;
   wnd.lpszClassName = szClassName;

   if(!RegisterClass(&wnd))
   {
      return -1;
   }

   if((hwnd = CreateWindow(szClassName,
      "Lid Locker",
      WS_OVERLAPPEDWINDOW,
      CW_USEDEFAULT,
      CW_USEDEFAULT,
      CW_USEDEFAULT,
      CW_USEDEFAULT,
      NULL,
      NULL,
      hInstance,
      NULL)) == NULL)
   {
      return -1;
   }

   ShowWindow(hwnd, SW_HIDE);
   UpdateWindow(hwnd);

   if((hpn = RegisterPowerSettingNotification(hwnd, &GLSC, DEVICE_NOTIFY_WINDOW_HANDLE)) == NULL)
   {
      return -1;
   }

   while(GetMessage(&msg, NULL, 0, 0))
   {
      TranslateMessage(&msg);
      DispatchMessage(&msg);
   }
   UnregisterPowerSettingNotification(hpn);

   return msg.wParam;
}

LRESULT CALLBACK LLWndProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
   const POWERBROADCAST_SETTING *ps;

   switch(msg)
   {
      case WM_DESTROY:
         PostQuitMessage(0);
         return 0;
      case WM_POWERBROADCAST:
         if(wParam == PBT_POWERSETTINGCHANGE)
         {
            ps = (POWERBROADCAST_SETTING *)lParam;
            if(IsEqualGUID(&(ps->PowerSetting), &GLSC))
            {
               LockWorkStation();
            }
         }
         break;
       default:
         break;
   }

   return DefWindowProc(hwnd, msg, wParam, lParam);
}
```
