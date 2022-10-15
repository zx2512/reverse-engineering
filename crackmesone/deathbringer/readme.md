The sub_7FF791436640 function is used to copy strings from one place to another. Here it initializes the strings which are outputted to the console

```
.text:00007FF791434AE7                 mov     [rsp+120h+enterSerialStr], rsi
.text:00007FF791434AEC                 mov     [rsp+120h+var_A8], rsi
.text:00007FF791434AF1                 mov     [rbp+20h+var_A0], 0Fh
.text:00007FF791434AF9                 mov     byte ptr [rsp+120h+enterSerialStr], sil
.text:00007FF791434AFE                 lea     r8d, [rsi+12h]  ; Size
.text:00007FF791434B02                 lea     rdx, aEnterSerial ; "[+] Enter Serial: "
.text:00007FF791434B09                 lea     rcx, [rsp+120h+enterSerialStr] ; void *
.text:00007FF791434B0E                 call    sub_7FF791436640
.text:00007FF791434B13                 nop
.text:00007FF791434B14                 mov     [rsp+120h+invalidSerialStr], rsi
.text:00007FF791434B19                 mov     [rsp+120h+var_C8], rsi
.text:00007FF791434B1E                 mov     [rsp+120h+var_C0], 0Fh
.text:00007FF791434B27                 mov     byte ptr [rsp+120h+invalidSerialStr], sil
.text:00007FF791434B2C                 lea     r8d, [rsi+13h]  ; Size
.text:00007FF791434B30                 lea     rdx, aInvalidSerial ; "[!] Invalid Serial\n"
.text:00007FF791434B37                 lea     rcx, [rsp+120h+invalidSerialStr] ; void *
.text:00007FF791434B3C                 call    sub_7FF791436640
.text:00007FF791434B41                 nop
.text:00007FF791434B42                 mov     [rsp+120h+correctSerialStr], rsi
.text:00007FF791434B47                 mov     [rsp+120h+var_E8], rsi
.text:00007FF791434B4C                 mov     [rsp+120h+var_E0], 0Fh
.text:00007FF791434B55                 mov     byte ptr [rsp+120h+correctSerialStr], sil
.text:00007FF791434B5A                 lea     r8d, [rsi+13h]  ; Size
.text:00007FF791434B5E                 lea     rdx, aCorrectSerial ; "[!] Correct Serial\n"
.text:00007FF791434B65                 lea     rcx, [rsp+120h+correctSerialStr] ; void *
.text:00007FF791434B6A                 call    sub_7FF791436640
```

