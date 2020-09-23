<div align="center">

## Attach a console to a window


</div>

### Description

Learn how to quickly attach a console to a window for instant output and feedback. Console apps trivialize the ability to output text data in a fast efficient manner. Displaying diagnostic data in a windowed application is not so trivial or straight forward. Have you ever wanted instantaneous feedback on a variable or state in your windowed program at run-time? One solution is to "attach" a console to your window. Through this attached console, your window can conveniently output information. The WinProg source below does just that. WinProg allocates a console window in WM_CREATE. It is through this console that WinProg can communicate what messages are being sent to it. The example below handles various messages in the window handler and then outputs them to the console. By retrieving a HANDLE from my CreateOutputConsole() function, you can then display whatever data you wish with a simple printf-like function - ConPrintf(). Just pass the HANDLE to the console, the size of the output buffer (big enough to accommodate the format string and the optional variables), the format string i.e.("Data: %d") etc...and the optional arguments (if any). Very similar to printf().
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Mahalosoft](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/mahalosoft.md)
**Level**          |Intermediate
**User Rating**    |3.8 (15 globes from 4 users)
**Compatibility**  |Microsoft Visual C\+\+, Borland C\+\+
**Category**       |[Debugging and Error Handling](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/debugging-and-error-handling__3-6.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/mahalosoft-attach-a-console-to-a-window__3-4971/archive/master.zip)





### Source Code

```
#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include <tchar.h>
#include <stdio.h>
#define WIN_X 300
#define WIN_Y 200
TCHAR WIN_NAME[]=_T( "WinCon by Mike Leon" );
TCHAR WIN_CLASS[]=_T( "__WIN_SHELL__" );
LRESULT CALLBACK WindowProc( HWND,UINT,WPARAM,LPARAM );
HANDLE CreateOutputConsole( LPCTSTR );
int ConPrintf( HANDLE,DWORD,LPCTSTR,... );
int WINAPI WinMain( HINSTANCE hInstance,
          HINSTANCE hPrevinstance,
          LPTSTR lpCmdline,
          int nCmdshow )
  {
    HWND hWnd;
    MSG mSg;
    WNDCLASSEX winClass;
    winClass.cbSize    =sizeof( WNDCLASSEX );
    winClass.style    = 0;
    winClass.lpfnWndProc =WindowProc;
    winClass.cbClsExtra  =0;
    winClass.cbWndExtra  =0;
    winClass.hInstance  =hInstance;
    winClass.hIcon    =LoadIcon( NULL,IDI_APPLICATION );
    winClass.hCursor   =LoadCursor( NULL,IDC_ARROW );
    winClass.hbrBackground=( HBRUSH )GetStockObject( WHITE_BRUSH );
    winClass.lpszMenuName =NULL;
    winClass.lpszClassName=WIN_CLASS;
    winClass.hIconSm   =NULL;
    RegisterClassEx( &winClass );
    hWnd=CreateWindow( WIN_CLASS,
             WIN_NAME,
             WS_CAPTION | WS_MINIMIZEBOX |
             WS_MAXIMIZEBOX | WS_SIZEBOX |
             WS_SYSMENU,
             CW_USEDEFAULT,CW_USEDEFAULT,
             WIN_X,WIN_Y,
             NULL,
             NULL,
             hInstance,
             NULL );
    ShowWindow( hWnd,nCmdshow );
    UpdateWindow( hWnd );
    while( GetMessage( &mSg,NULL,0,0 ) )
     {
      TranslateMessage( &mSg );
      DispatchMessage ( &mSg );
     }
    return ( mSg.wParam );
  }
LRESULT CALLBACK WindowProc( HWND hWnd,
               UINT mSg,
               WPARAM wParam,
               LPARAM lParam )
   {
     PAINTSTRUCT ps;
     HDC hdc;
     TCHAR sBuffer[ 255 ];
     static HANDLE hCon;
     switch( mSg )
      {
        case WM_CREATE:
          hCon=CreateOutputConsole( _T( "Debug Shell" ) );
          if( INVALID_HANDLE_VALUE==hCon )
            MessageBox( hWnd,_T( "ERROR:Failed" ),
                  _T( "FAILED" ),MB_OK );
          else
            ConPrintf( hCon,255,"WM_CREATE: value of %d\n",
                  WM_CREATE );
        break;
        case WM_PAINT:
		      hdc=BeginPaint( hWnd,&ps);
		      EndPaint( hWnd,&ps );
           ConPrintf( hCon,255,"WM_PAINT: value %d\n",
                 WM_PAINT );
        break;
        case WM_CHAR:
          ConPrintf( hCon,255,_T( "WM_CHAR: value of %c\n" ),
                ( TCHAR )wParam );
        break;
        case WM_LBUTTONDOWN:
          ConPrintf( hCon,255,_T( "WM_LBUTTONDOWN: value of %d\n" ),
                      WM_LBUTTONDOWN );
        break;
        case WM_LBUTTONUP:
          ConPrintf( hCon,255,_T( "WM_LBUTTONUP: value of %d\n" ),
                WM_LBUTTONUP );
        break;
        case WM_SYSKEYDOWN:
          if( lParam & 0x20000000 &&
            wParam==VK_RETURN )
          ConPrintf( hCon,255,_T( "ALT+Enter key is was pressed\n" ) );
        break;
        case WM_MOVING:
          ConPrintf( hCon,255,_T( "WM_MOVING: value of %d\n" ),
                WM_MOVING );
        break;
        case WM_SIZE:
          lstrcpy( sBuffer,_T( " " ) );
          if( wParam==SIZE_MINIMIZED )
            lstrcpy( sBuffer,_T( "MINIMIZED WINDOW" ) );
          else if( wParam==SIZE_MAXIMIZED )
            lstrcpy( sBuffer,_T( "MAXIMIZED WINDOW" ) );
          else if( wParam==SIZE_RESTORED )
            lstrcpy( sBuffer,_T( "RESTORED WINDOW" ) );
          if( lstrcmp( sBuffer,_T( " " ) )!=0 )
            ConPrintf( hCon,255,"%s\n",sBuffer );
        break;
        case WM_RBUTTONUP:
          FreeConsole();
        break;
        case WM_DESTROY:
          FreeConsole();
	         PostQuitMessage( 0 );
        break;
	       default: return( DefWindowProc( hWnd,mSg,
                        wParam,lParam ) );
      }
    return 0;
  }
HANDLE CreateOutputConsole( LPCTSTR lpConTitle )
 {
   HANDLE hCon;
   if( !AllocConsole() ) return NULL;
   hCon=CreateFile( _T( "CONOUT$" ),GENERIC_WRITE,0,0,
            CREATE_NEW,FILE_ATTRIBUTE_NORMAL,0 );
   if( hCon!=INVALID_HANDLE_VALUE && lpConTitle )
     SetConsoleTitle( lpConTitle);
   return hCon;
 }
int ConPrintf( HANDLE hCon,DWORD dwSize,LPCTSTR lpText,... )
 {
   TCHAR *sMsg;
   DWORD dwBytes;
   va_list ptr;
   int nReturn;
   sMsg=NULL;
   nReturn=0;
   if( dwSize>0 )
     sMsg=( TCHAR* )HeapAlloc( GetProcessHeap(),0,
                  sizeof( TCHAR )*dwSize );
   if( !sMsg )
     return -1;
   va_start( ptr,lpText );
   if( vsprintf( sMsg,lpText,ptr )<0 )
     nReturn=-2;
   if( nReturn!=-2 )
     nReturn=( !WriteConsole( hCon,sMsg,lstrlen( sMsg ),
                  &dwBytes,NULL ) ) ?
       -3 : dwBytes;
   HeapFree( GetProcessHeap(),0,sMsg );
   return nReturn;
 }
```

