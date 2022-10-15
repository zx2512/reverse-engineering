
I ran the crackme and get greeted with a prompt that asks for a password. After inputting an arbitrary string, a message box pops up informing that the password is incorrect. I load the binary executable into IDA and find the main function. 

    .text:00402AEB                 push    esi             ; ArgList
    .text:00402AEC                 push    offset Format   ; "Enter password: "
    .text:00402AF1                 call    printf
    .text:00402AF6                 add     esp, 4
    .text:00402AF9                 mov     [ebp+var_38], 0
    .text:00402B00                 mov     [ebp+var_34], 0Fh
    .text:00402B07                 mov     byte ptr [ebp+var_48], 0
    .text:00402B0B ;   try {
    .text:00402B0B                 mov     [ebp+var_4], 0
    .text:00402B12                 mov     ecx, ds:?cin@std@@3V?$basic_istream@DU?$char_traits@D@std@@@1@A ; std::istream std::cin
    .text:00402B18                 push    offset unk_406410
    .text:00402B1D                 call    ds:??5?$basic_istream@DU?$char_traits@D@std@@@std@@QAEAAV01@AAH@Z ; std::istream::operator>>(int &)
    .text:00402B23                 call    sub_402D60
    .text:00402B28                 push    offset Command  ; "pause"

We can see that it prompts for a password, takes input from the command line and then stores it in memory at address 0x406410. We scroll down a little, and encounter a call to MessageBoxA in the same function.

    .text:00402C43 loc_402C43:                             ; CODE XREF: _main+143↑j
    .text:00402C43                 push    0               ; uType
    .text:00402C45                 push    offset Caption  ; lpCaption
    .text:00402C4A                 push    offset Text     ; lpText
    .text:00402C4F                 push    0               ; hWnd
    .text:00402C51                 call    ds:MessageBoxA

`Text` is the pointer to the string that contains the text that gets displayed in the message box. We load the executable in x64dbg and set a breakpoint at the address `Text` points to, then execute the debugee. We enter the password and then get notified with the MessageBox that tells us it was incorrect. The breakpoint didn't hit, which is strange. This could mean that the memory gets modified by another process. Using Process Explorer, we can see that the executable spawns a notepad process, which it could use to inject code into.

![enter image description here](https://i.imgur.com/JxgcGx8.png)

In order to inject code into the child notepad process that got spawned by the binary, it could use the `WriteProcessMemory` function. We set a breakpoint on the function call, which hits. From MSDN documentation, we see that the pointer to the memory to be overwritten is second:

    BOOL WriteProcessMemory( 
	    [in] HANDLE hProcess, 
	    [in] LPVOID lpBaseAddress, 
	    [in] LPCVOID lpBuffer, 
	    [in] SIZE_T nSize, 
	    [out] SIZE_T *lpNumberOfBytesWritten 
    );

We can see the address to be modified on the stack. We copy it, step out of the WriteProcessMemory function and then attach a debugger to the child notepad process. Then we go to the address and notice an interesting sequence of instructions:

07270293 | mov byte ptr ss:[ebp-54],4C             | 4C:'L'
07270297 | mov byte ptr ss:[ebp-53],6F             | 6F:'o'
0727029B | mov byte ptr ss:[ebp-52],73             | 73:'s'
0727029F | mov byte ptr ss:[ebp-51],65             | 65:'e'
072702A3 | mov byte ptr ss:[ebp-50],21             | 21:'!'
072702A7 | mov byte ptr ss:[ebp-4F],21             | 21:'!'
072702AB | mov byte ptr ss:[ebp-4E],0              |
072702AF | mov byte ptr ss:[ebp-94],57             | 57:'W'
072702B6 | mov byte ptr ss:[ebp-93],69             | 69:'i'
072702BD | mov byte ptr ss:[ebp-92],6E             | 6E:'n'
072702C4 | mov byte ptr ss:[ebp-91],21             | 21:'!'
072702CB | mov byte ptr ss:[ebp-90],21             | 21:'!'
072702D2 | mov byte ptr ss:[ebp-8F],21             | 21:'!'
072702D9 | mov byte ptr ss:[ebp-8E],0              |

It prepares the strings which are included in the message box text field. 

    mov dword ptr ss:[ebp-4],406410
    mov dword ptr ss:[ebp-C],406408

ebp-4 now contains the address of the password we input and ebp-C contains the address of the memory where the string gets output to.

    push 0
    push 4
    lea edx,dword ptr ss:[ebp-5C]
    push edx
    mov eax,dword ptr ss:[ebp-4]
    push eax
    mov ecx,dword ptr ss:[ebp-2C]
    push ecx
    call dword ptr ss:[ebp-58]

ebp-58 is a call to ReadProcessMemory and ebp-5C contains the output buffer. It reads data at memory ebp-4 which as we established, points to the address that contains the password we input initially.

    cmp dword ptr ss:[ebp-5C],23D4

And then it compares the buffer to 9172. If the buffer is equal to that number, it writes "Win!!!" to the parent process' memory, otherwise it is "Lose!!".

We test this and input 9172 as the password and the program informs us that we entered the correct password.