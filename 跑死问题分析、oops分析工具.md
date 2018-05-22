

示例源代码

```c
int main(){
	fstream fs;
	char*  buf = new char[100];
	fs.open("readme");
	if(!fs.is_open())
		cout<< "open file failed"<<endl;
    fs.getline(buf, 100);
       cout<< buf<<endl;
    fs.close();

}
```

### objdump
如果是情况一，直接用地址dump出来。咱们回头看一下Backtrace信息：bdi_register+0xec/0x150，这里的0xec是偏移，而0x150是该函数的大小。用objdump默认可以获取整个vmlinux的代码，但是咱们其实只获取一部分，这个可以通过--start-address和--stop-address来指定。另外-d可以汇编代码，-S则可以并入源代码。

```c
 显示来自目标 <文件> 的信息。
 至少必须给出以下选项之一：
  -a, --archive-headers    Display archive header information
  -f, --file-headers       Display the contents of the overall file header
  -p, --private-headers    Display object format specific file header contents
  -P, --private=OPT,OPT... Display object format specific contents
  -h, --[section-]headers  Display the contents of the section headers
  -x, --all-headers        Display the contents of all headers
  -d, --disassemble        Display assembler contents of executable sections
  -D, --disassemble-all    Display assembler contents of all sections
  -S, --source             Intermix source code with disassembly            源代码和汇编代码混合显示
  -s, --full-contents      Display the full contents of all sections requested
  -g, --debugging          Display debug information in object file
  -e, --debugging-tags     Display debug information using ctags style
  -G, --stabs              Display (in raw form) any STABS info in the file
  -W[lLiaprmfFsoRt] or
  --dwarf[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,
          =frames-interp,=str,=loc,=Ranges,=pubtypes,
          =gdb_index,=trace_info,=trace_abbrev,=trace_aranges,
          =addr,=cu_index]
                           Display DWARF info in the file
  -t, --syms               Display the contents of the symbol table(s)
  -T, --dynamic-syms       Display the contents of the dynamic symbol table
  -r, --reloc              Display the relocation entries in the file
  -R, --dynamic-reloc      Display the dynamic relocation entries in the file
  @<file>                  Read options from <file>
  -v, --version            Display this program's version number
  -i, --info               List object formats and architectures supported
  -H, --help               Display this informatio
```

-C ：c++ 符号名逆向，可以显示出可读的C++符号名（最好的方式）

```c
ll@ll-VirtualBox:~/mycode/c++/lession3$ objdump -SC a.out 
0000000000400ca6 <main>:
  400ca6:	55                   	push   %rbp
  400ca7:	48 89 e5             	mov    %rsp,%rbp
  400caa:	53                   	push   %rbx
  400cab:	48 81 ec 38 02 00 00 	sub    $0x238,%rsp
  400cb2:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  400cb9:	00 00 
  400cbb:	48 89 45 e8          	mov    %rax,-0x18(%rbp)
  400cbf:	31 c0                	xor    %eax,%eax
  400cc1:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400cc8:	48 89 c7             	mov    %rax,%rdi
  400ccb:	e8 50 fe ff ff       	callq  400b20 <std::basic_fstream<char, std::char_traits<char> >::basic_fstream()@plt>
  400cd0:	bf 64 00 00 00       	mov    $0x64,%edi
  400cd5:	e8 b6 fd ff ff       	callq  400a90 <operator new[](unsigned long)@plt>
  400cda:	48 89 85 c8 fd ff ff 	mov    %rax,-0x238(%rbp)
  400ce1:	be 10 00 00 00       	mov    $0x10,%esi
  400ce6:	bf 08 00 00 00       	mov    $0x8,%edi
  400ceb:	e8 3a 01 00 00       	callq  400e2a <std::operator|(std::_Ios_Openmode, std::_Ios_Openmode)>
  400cf0:	89 c2                	mov    %eax,%edx
  400cf2:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400cf9:	be c4 0e 40 00       	mov    $0x400ec4,%esi
  400cfe:	48 89 c7             	mov    %rax,%rdi
  400d01:	e8 8a fe ff ff       	callq  400b90 <std::basic_fstream<char, std::char_traits<char> >::open(char const*, std::_Ios_Openmode)@plt>
  400d06:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400d0d:	48 89 c7             	mov    %rax,%rdi
  400d10:	e8 3b fe ff ff       	callq  400b50 <std::basic_fstream<char, std::char_traits<char> >::is_open()@plt>
  400d15:	83 f0 01             	xor    $0x1,%eax
  400d18:	84 c0                	test   %al,%al
  400d1a:	74 1c                	je     400d38 <main+0x92>
  400d1c:	be cb 0e 40 00       	mov    $0x400ecb,%esi
  400d21:	bf c0 20 60 00       	mov    $0x6020c0,%edi
  400d26:	e8 b5 fd ff ff       	callq  400ae0 <std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)@plt>
  400d2b:	be 60 0b 40 00       	mov    $0x400b60,%esi
  400d30:	48 89 c7             	mov    %rax,%rdi
  400d33:	e8 08 fe ff ff       	callq  400b40 <std::ostream::operator<<(std::ostream& (*)(std::ostream&))@plt>
  400d38:	48 8b 8d c8 fd ff ff 	mov    -0x238(%rbp),%rcx
  400d3f:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400d46:	ba 64 00 00 00       	mov    $0x64,%edx
  400d4b:	48 89 ce             	mov    %rcx,%rsi
  400d4e:	48 89 c7             	mov    %rax,%rdi
  400d51:	e8 aa fd ff ff       	callq  400b00 <std::istream::getline(char*, long)@plt>
  400d56:	48 8b 85 c8 fd ff ff 	mov    -0x238(%rbp),%rax
  400d5d:	48 89 c6             	mov    %rax,%rsi
  400d60:	bf c0 20 60 00       	mov    $0x6020c0,%edi
  400d65:	e8 76 fd ff ff       	callq  400ae0 <std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)@plt>
  400d6a:	be 60 0b 40 00       	mov    $0x400b60,%esi
  400d6f:	48 89 c7             	mov    %rax,%rdi
  400d72:	e8 c9 fd ff ff       	callq  400b40 <std::ostream::operator<<(std::ostream& (*)(std::ostream&))@plt>
  400d77:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400d7e:	48 89 c7             	mov    %rax,%rdi
  400d81:	e8 6a fd ff ff       	callq  400af0 <std::basic_fstream<char, std::char_traits<char> >::close()@plt>
  400d86:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400d8d:	48 89 c7             	mov    %rax,%rdi
  400d90:	e8 7b fd ff ff       	callq  400b10 <std::basic_fstream<char, std::char_traits<char> >::~basic_fstream()@plt>
  400d95:	b8 00 00 00 00       	mov    $0x0,%eax
  400d9a:	48 8b 4d e8          	mov    -0x18(%rbp),%rcx
  400d9e:	64 48 33 0c 25 28 00 	xor    %fs:0x28,%rcx
  400da5:	00 00 
  400da7:	74 24                	je     400dcd <main+0x127>
  400da9:	eb 1d                	jmp    400dc8 <main+0x122>
  400dab:	48 89 c3             	mov    %rax,%rbx
  400dae:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400db5:	48 89 c7             	mov    %rax,%rdi
  400db8:	e8 53 fd ff ff       	callq  400b10 <std::basic_fstream<char, std::char_traits<char> >::~basic_fstream()@plt>
  400dbd:	48 89 d8             	mov    %rbx,%rax
  400dc0:	48 89 c7             	mov    %rax,%rdi
  400dc3:	e8 b8 fd ff ff       	callq  400b80 <_Unwind_Resume@plt>
  400dc8:	e8 63 fd ff ff       	callq  400b30 <__stack_chk_fail@plt>
  400dcd:	48 81 c4 38 02 00 00 	add    $0x238,%rsp
  400dd4:	5b                   	pop    %rbx
  400dd5:	5d                   	pop    %rbp
  400dd6:	c3                   	retq
```
objdump -S a.out  显示出汇编和C++的混合符号代码，不宜读，不推荐
```c
0000000000400ca6 <main>:
  400ca6:	55                   	push   %rbp
  400ca7:	48 89 e5             	mov    %rsp,%rbp
  400caa:	53                   	push   %rbx
  400cab:	48 81 ec 38 02 00 00 	sub    $0x238,%rsp
  400cb2:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  400cb9:	00 00 
  400cbb:	48 89 45 e8          	mov    %rax,-0x18(%rbp)
  400cbf:	31 c0                	xor    %eax,%eax
  400cc1:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400cc8:	48 89 c7             	mov    %rax,%rdi
  400ccb:	e8 50 fe ff ff       	callq  400b20 <_ZNSt13basic_fstreamIcSt11char_traitsIcEEC1Ev@plt>
  400cd0:	bf 64 00 00 00       	mov    $0x64,%edi
  400cd5:	e8 b6 fd ff ff       	callq  400a90 <_Znam@plt>
  400cda:	48 89 85 c8 fd ff ff 	mov    %rax,-0x238(%rbp)
  400ce1:	be 10 00 00 00       	mov    $0x10,%esi
  400ce6:	bf 08 00 00 00       	mov    $0x8,%edi
  400ceb:	e8 3a 01 00 00       	callq  400e2a <_ZStorSt13_Ios_OpenmodeS_>
  400cf0:	89 c2                	mov    %eax,%edx
  400cf2:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400cf9:	be c4 0e 40 00       	mov    $0x400ec4,%esi
  400cfe:	48 89 c7             	mov    %rax,%rdi
  400d01:	e8 8a fe ff ff       	callq  400b90 <_ZNSt13basic_fstreamIcSt11char_traitsIcEE4openEPKcSt13_Ios_Openmode@plt>
  400d06:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400d0d:	48 89 c7             	mov    %rax,%rdi
  400d10:	e8 3b fe ff ff       	callq  400b50 <_ZNSt13basic_fstreamIcSt11char_traitsIcEE7is_openEv@plt>
  400d15:	83 f0 01             	xor    $0x1,%eax
  400d18:	84 c0                	test   %al,%al
  400d1a:	74 1c                	je     400d38 <main+0x92>
  400d1c:	be cb 0e 40 00       	mov    $0x400ecb,%esi
  400d21:	bf c0 20 60 00       	mov    $0x6020c0,%edi
  400d26:	e8 b5 fd ff ff       	callq  400ae0 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
  400d2b:	be 60 0b 40 00       	mov    $0x400b60,%esi
  400d30:	48 89 c7             	mov    %rax,%rdi
  400d33:	e8 08 fe ff ff       	callq  400b40 <_ZNSolsEPFRSoS_E@plt>
  400d38:	48 8b 8d c8 fd ff ff 	mov    -0x238(%rbp),%rcx
  400d3f:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400d46:	ba 64 00 00 00       	mov    $0x64,%edx
  400d4b:	48 89 ce             	mov    %rcx,%rsi
  400d4e:	48 89 c7             	mov    %rax,%rdi
  400d51:	e8 aa fd ff ff       	callq  400b00 <_ZNSi7getlineEPcl@plt>
  400d56:	48 8b 85 c8 fd ff ff 	mov    -0x238(%rbp),%rax
  400d5d:	48 89 c6             	mov    %rax,%rsi
  400d60:	bf c0 20 60 00       	mov    $0x6020c0,%edi
  400d65:	e8 76 fd ff ff       	callq  400ae0 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
  400d6a:	be 60 0b 40 00       	mov    $0x400b60,%esi
  400d6f:	48 89 c7             	mov    %rax,%rdi
  400d72:	e8 c9 fd ff ff       	callq  400b40 <_ZNSolsEPFRSoS_E@plt>
  400d77:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400d7e:	48 89 c7             	mov    %rax,%rdi
  400d81:	e8 6a fd ff ff       	callq  400af0 <_ZNSt13basic_fstreamIcSt11char_traitsIcEE5closeEv@plt>
  400d86:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400d8d:	48 89 c7             	mov    %rax,%rdi
  400d90:	e8 7b fd ff ff       	callq  400b10 <_ZNSt13basic_fstreamIcSt11char_traitsIcEED1Ev@plt>
  400d95:	b8 00 00 00 00       	mov    $0x0,%eax
  400d9a:	48 8b 4d e8          	mov    -0x18(%rbp),%rcx
  400d9e:	64 48 33 0c 25 28 00 	xor    %fs:0x28,%rcx
  400da5:	00 00 
  400da7:	74 24                	je     400dcd <main+0x127>
  400da9:	eb 1d                	jmp    400dc8 <main+0x122>
  400dab:	48 89 c3             	mov    %rax,%rbx
  400dae:	48 8d 85 d0 fd ff ff 	lea    -0x230(%rbp),%rax
  400db5:	48 89 c7             	mov    %rax,%rdi
  400db8:	e8 53 fd ff ff       	callq  400b10 <_ZNSt13basic_fstreamIcSt11char_traitsIcEED1Ev@plt>
  400dbd:	48 89 d8             	mov    %rbx,%rax
  400dc0:	48 89 c7             	mov    %rax,%rdi
  400dc3:	e8 b8 fd ff ff       	callq  400b80 <_Unwind_Resume@plt>
  400dc8:	e8 63 fd ff ff       	callq  400b30 <__stack_chk_fail@plt>
  400dcd:	48 81 c4 38 02 00 00 	add    $0x238,%rsp
  400dd4:	5b                   	pop    %rbx
  400dd5:	5d                   	pop    %rbp
  400dd6:	c3                   	retq   
```

### nm 命令 --显示可执行文件中的符号表
显示关于指定 File 中符号的信息，文件可以是对象文件、可执行文件或对象文件库。如果文件没有包含符号信息，nm 命令报告该情况，但不把它解释为出错条件。 nm 命令缺省情况下报告十进制符号表示法下的数字值。
### readelf -a a.out 显示关于 ELF 格式文件内容的信息，可以读出架构等信息

### addr2line

### gdb 
因为可以直接用 符号+偏移 的方式，因此，即使其他地方有改动，这个相对的位置是不变的。编译时加上-g，即在编译时添加上调试信息。无该选项，则不能用。
```c
ll@ll-VirtualBox:~/mycode/c++/lession3$ g++ -o file-test -c -g file-test.c 
ll@ll-VirtualBox:~/mycode/c++/lession3$ gdb 
(gdb) list *(main)
0x0 is in main() (file-test.c:11).
6	using std::cin;
7	using std::cout;
8	using std::endl;
9	using std::fstream;
10	
11	int main(){
12		fstream fs;
13		char*  buf = new char[100];
14		fs.open("readme");
15		if(!fs.is_open())

```
