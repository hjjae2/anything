### File System Basic

> Persistence

> 메모리(휘발성) 상의 문제는 컴퓨터를 reboot 하면 해결된다.
> 
> 디스크(비휘발성) 상의 문제는 컴퓨터를 reboot 해도 해결되지 않는다.

<br>

**Computer system's 4 key abstractions**
1. Process
2. Virtual Memory
3. Lock
4. File

<br>

**What is a file?**

(영구히 저장되는) Bytes 들의 선형 집합(배열)이다.

각각의 file 은 **절대경로, 상대경로**를 갖는다.

Inode (OS, Low-level 단의 이름) 을 갖는다.

Device, Pipe, Socket, some processes 는 file 로서 취급된다.<br>
(OS 가 관리하는 모든 장치는 file 로서 접근 가능하다.)

DRAM 에 write 하는 것은, Disk 에 write 하는 것 보다 대략 10000배 빠르다.

**\* Delayed write : Write later all dirty data into disk(memory)**

<br>

**Contents in a FS**

1. Userdata : data written by users
2. Metadata : data written by fs for managing files (in inode)