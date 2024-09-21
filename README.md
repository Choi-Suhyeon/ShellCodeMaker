# shellcodeGen
  This shell script facilitates the generation of shellcode. It assists in crafting shellcode effortlessly from stdin or a file with the extension 's', which contains code written in either att or intel syntax.

## Version

6.2.0 (2024.09.21.)

## Programs Used Within The Program
  Many additional programs have been utilized; however, the following are directly pertinent to the process of generating shellcode.

- as
- ld
- hexdump
- objdump
- objcopy

## Useage Guide
```
shellcode-gen TARGET_FILE_NAME [-a -t -s -r -k]
shellcode-gen -h

<options>
TARGET_FILE_PATH         : (Optional) Specify the file with the extension 's' for generating shellcode.
-a --arch ARCHITECTURE   : (Optional) Specify architecture(x86_64(default) or i386).
-t --type LITERAL_TYPE   : (Optional) Specify the literal type(str(default), arr, or bytes) of bytes of shellcode to output.
-s --syntax SYNTAX       : (Optional) Specify syntax(att(default) or intel).
-b --blacklist FILE_PATH : (Optional) Specify the file containing the blacklist for bytes.
-r --run                 : (Optional) Run shell code by making it an executable file.
-k --keep                : (Optional) Keep the intermediate files.

-h --help                : Print this message and then exit.

<exit code>
0 : Success.
1 : Failure.
```

- If the '-a', '-t', or '-s' options are not specified, the defaults will be 'x86\_64', 'str', and 'att', respectively.
- Assembly code can be provided via stdin if a target file name is not specified.
- The location of the TARGET\_FILE\_PATH is adjustable, similar to other options.
- Only the final output is sent to stdout; all other messages are directed to stderr.
- Bytes listed in the blacklist are highlighted in red when displayed, making it easy to spot bytes that should not be used at a glance.
- Characters considered as whitespace in C are included in the blacklist by default.
- If there are additional bytes that require special attention, you must pass the path to the file containing them using the `--blacklist` option.

## Blacklist File Syntax

- Each element is separated by a whitespace character. The type and number of whitespace characters in between do not matter.
- Only one-line comments are supported, and both of the following formats are allowed: `# comment 1`, `// comment 2`.
- Case sensitivity is not an issue. All characters are converted to lowercase during processing.
- The string '0x' and commas (,) are ignored.

### Example of Blacklist File

In most cases, simply copying and pasting the provided condition or typing it out casually will generally fit the required format.

```
0x88, 0x89, 0x8A, 0x8B, 0x8C, 0x8E, // MOV
0xA0, 0xA1, 0xA2, 0xA3, // MOV
0xA4, 0xA5, // MOVS
0xB0, 0xB1, 0xB2, 0xB3, 0xB4, 0xB5, 0xB6, 0xB7, // MOV
0xB8, 0xB9, 0xBA, 0xBB, 0xBC, 0xBD, 0xBE, 0xBF, // MOV
0xC6, 0xC7 // MOV
```

## Example
### Use With I/O Redirection
```
$ cat execve.s
.section .text
.global _start
_start:
xorl %ecx, %ecx
pushl %ecx
pushl $0x68732F6E
pushl $0x69622F2F
pushl $0x16
popl %eax
sarl $1, %eax
movl %ecx, %edx
movl %esp, %ebx
int $0x80

$ cat execve.s | ./shellcode-gen -a i386 -t bytes > output
[-------------------- ASSEMBLY WITH MACHINE CODE --------------------]

temp_shellcode_gen6029.o:     file format elf32-i386


Disassembly of section .text:

00000000 <_start>:
   0:	31 c9                	xor    %ecx,%ecx
   2:	51                   	push   %ecx
   3:	68 6e 2f 73 68       	push   $0x68732f6e
   8:	68 2f 2f 62 69       	push   $0x69622f2f
   d:	6a 16                	push   $0x16
   f:	58                   	pop    %eax
  10:	d1 f8                	sar    %eax
  12:	89 ca                	mov    %ecx,%edx
  14:	89 e3                	mov    %esp,%ebx
  16:	cd 80                	int    $0x80

NOTE : Whitespace in C :
  0x09(Horizontal Tab), 0x0A(Line Feed),       0x0B(Vertical Tab),
  0x0C(Form Feed),      0x0D(Carriage Return), 0x20(Space)

[-------------------------------------------------------------------]


$ xxd output                                              
00000000: 31c9 5168 6e2f 7368 682f 2f62 696a 1658  1.Qhn/shh//bij.X
00000010: d1f8 89ca 89e3 cd80                      ........
```

### Run Your Shellcode
```
$ ./shellcode-gen -r -a i386 execve.s      
[-------------------- ASSEMBLY WITH MACHINE CODE --------------------]

execve.o:     file format elf32-i386


Disassembly of section .text:

00000000 <_start>:
   0:	31 c9                	xor    %ecx,%ecx
   2:	51                   	push   %ecx
   3:	68 6e 2f 73 68       	push   $0x68732f6e
   8:	68 2f 2f 62 69       	push   $0x69622f2f
   d:	6a 16                	push   $0x16
   f:	58                   	pop    %eax
  10:	d1 f8                	sar    %eax
  12:	89 ca                	mov    %ecx,%edx
  14:	89 e3                	mov    %esp,%ebx
  16:	cd 80                	int    $0x80

NOTE : Whitespace in C :
  0x09(Horizontal Tab), 0x0A(Line Feed),       0x0B(Vertical Tab),
  0x0C(Form Feed),      0x0D(Carriage Return), 0x20(Space)

[-------------------------------------------------------------------]

[-------------------------- run YOUR CODE --------------------------]
$ whoami
suhyeon
$ 
[-------------------------------------------------------------------]

\x31\xC9\x51\x68\x6E\x2F\x73\x68\x68\x2F\x2F\x62\x69\x6A\x16\x58\xD1\xF8\x89\xCA\x89\xE3\xCD\x80
```

## LICENSE

This project is licensed under the GNU General Public License v3.0
