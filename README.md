# AutoClave

# TL;DR

No more "sandboxes" that don't work in practice.  Devs need root, Ops needs speed,
Security needs well defined interfaces and known good state.

Basically, seccomp-bpf on QEMU *after process start* but *before attacker
interaction* can deliver, because after setup, the userspace-focused QEMU
requires really not much from the kernel, in order to run not just Linux
(or its containers), but Windows and probably OSX as well.

All the compat, handful of syscalls, couple of memory maps and open files.

Also if you do some clever things with memory management you can boot an arbitrarily
complex environment *subsecond*, with memory deduplication *in a safe context* 
(no user data, only leaks what a dev puts in).  Maybe we don't need Unikernels,
to give known good state to every user coming in.

# List of exposed System Calls

<table cellspacing="0" border="0">
	<colgroup width="34"></colgroup>
	<colgroup width="104"></colgroup>
	<colgroup width="467"></colgroup>
	<colgroup width="129"></colgroup>
	<tr>
		<td height="17" align="left" bgcolor="#66CCFF"><b>No</b></td>
		<td align="left" bgcolor="#66CCFF"><b>Syscall name</b></td>
		<td align="left" bgcolor="#66CCFF"><b>Description</b></td>
		<td align="left" bgcolor="#66CCFF"><b>Source</b></td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="1" sdnum="1033;">1</td>
		<td align="left">nanosleep  </td>
		<td align="left">pause execution for a specified time (nano seconds)</td>
		<td align="left">kernel/sched.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="2" sdnum="1033;">2</td>
		<td align="left">read  </td>
		<td align="left">read from a file descriptor</td>
		<td align="left">fs/read_write.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="3" sdnum="1033;">3</td>
		<td align="left">readv  </td>
		<td align="left">read data into multiple buffers</td>
		<td align="left">fs/read_write.c</td>
	</tr>
	<tr>
		<td height="47" align="left" sdval="4" sdnum="1033;">4</td>
		<td align="left">preadv  </td>
		<td align="left">performs the same task as&nbsp;readv(), but adds a fourth argument,&nbsp;offset, which specifies the file offset at which the input operation is to be performed.</td>
		<td align="left">fs/read_write.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="5" sdnum="1033;">5</td>
		<td align="left">pread64  </td>
		<td align="left">differs from&nbsp;pread()&nbsp;in that the&nbsp;offset&nbsp;parameter is an&nbsp;off64_t&nbsp;instead of an&nbsp;off_t</td>
		<td align="left">fs/read_write.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="6" sdnum="1033;">6</td>
		<td align="left">write  </td>
		<td align="left">write to a file descriptor</td>
		<td align="left">fs/read_write.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="7" sdnum="1033;">7</td>
		<td align="left">writev  </td>
		<td align="left">write data into multiple buffers </td>
		<td align="left">fs/read_write.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="8" sdnum="1033;">8</td>
		<td align="left">pwritev  </td>
		<td align="left">performs the same task as&nbsp;writev(), but adds a fourth argument,&nbsp;offset, which specifies the file offset at which the output operation is to be performed.</td>
		<td align="left">fs/read_write.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="9" sdnum="1033;">9</td>
		<td align="left">pwrite64  </td>
		<td align="left">It differs from&nbsp;pwrite()&nbsp;in that the&nbsp;offset&nbsp;parameter is an&nbsp;off64_t&nbsp;instead of an&nbsp;off_t</td>
		<td align="left">fs/read_write.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="10" sdnum="1033;">10</td>
		<td align="left">clock_gettime  </td>
		<td align="left">set the time of the specified clock clk_id.</td>
		<td align="left"><br></td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="11" sdnum="1033;">11</td>
		<td align="left">gettimeofday  </td>
		<td align="left">get the date and time</td>
		<td align="left">kernel/time.c</td>
	</tr>
	<tr>
		<td height="47" align="left" sdval="12" sdnum="1033;">12</td>
		<td align="left">futex  </td>
		<td align="left">provides a method for a program to wait for a value at a given address to change, and a method to wake up anyone waiting on a particular address</td>
		<td align="left">kernel/futex.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="13" sdnum="1033;">13</td>
		<td align="left">poll  </td>
		<td align="left">wait for some event on a file descriptor</td>
		<td align="left">fs/select.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="14" sdnum="1033;">14</td>
		<td align="left">ppoll  </td>
		<td align="left">waits for one of a set of file descriptors to become ready to perform I/O.</td>
		<td align="left">fs/select.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="15" sdnum="1033;">15</td>
		<td align="left">exit  </td>
		<td align="left">terminate the current process </td>
		<td align="left">kernel/exit.c</td>
	</tr>
	<tr>
		<td height="47" align="left" sdval="16" sdnum="1033;">16</td>
		<td align="left">exit_group  </td>
		<td align="left">equivalent to&nbsp;exit,&nbsp;except that it terminates not only the calling thread, but all threads in the calling process's thread group.</td>
		<td align="left">kernel/exit.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="17" sdnum="1033;">17</td>
		<td align="left">rt_sigprocmask  </td>
		<td align="left">used to fetch and/or change the signal mask of the calling thread.</td>
		<td align="left">kernel/signal.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="18" sdnum="1033;">18</td>
		<td align="left">rt_sigtimedwait  </td>
		<td align="left">operates in exactly the same way as sigwaitinfo() except that it has an additional argument, timeout, which specifies a minimum interval for which the thread is suspended waiting for a signal.</td>
		<td align="left">kernel/signal.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="19" sdnum="1033;">19</td>
		<td align="left">rt_sigreturn  </td>
		<td align="left">return from signal handler and cleanup stack frame</td>
		<td align="left">arch/i386/kernel/signal.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="20" sdnum="1033;">20</td>
		<td align="left">rt_sigaction  </td>
		<td align="left">used to change the action taken by a process on receipt of a specific signal</td>
		<td align="left">kernel/signal.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="21" sdnum="1033;">21</td>
		<td align="left">rt_sigpending  </td>
		<td align="left">returns the set of signals that are pending for delivery to the calling thread</td>
		<td align="left">kernel/signal.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="22" sdnum="1033;">22</td>
		<td align="left">rt_sigqueueinfo  </td>
		<td align="left">&nbsp;low-level interfaces used to send a signal plus data to a process or thread</td>
		<td align="left">kernel/signal.c</td>
	</tr>
	<tr>
		<td height="62" align="left" sdval="23" sdnum="1033;">23</td>
		<td align="left">rt_sigsuspend  </td>
		<td align="left">temporarily replaces the signal mask of the calling process with the mask given by&nbsp;mask&nbsp;and then suspends the process until delivery of a signal whose action is to invoke a signal handler or to terminate a process.</td>
		<td align="left">arch/i386/kernel/signal.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="24" sdnum="1033;">24</td>
		<td align="left">mprotect  </td>
		<td align="left">set protection of memory mapping </td>
		<td align="left">mm/mprotect.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="25" sdnum="1033;">25</td>
		<td align="left">madvise  </td>
		<td align="left">give advice about use of memory</td>
		<td align="left"><br></td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="26" sdnum="1033;">26</td>
		<td align="left">clone  </td>
		<td align="left">create a child process</td>
		<td align="left">ch/i386/kernel/process.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="27" sdnum="1033;">27</td>
		<td align="left">mmap  </td>
		<td align="left">creates a new mapping in the virtual address space of the calling process.</td>
		<td align="left">mm/mmap.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="28" sdnum="1033;">28</td>
		<td align="left">munmap   </td>
		<td align="left">unmap pages of memory</td>
		<td align="left">mm/mmap.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="29" sdnum="1033;">29</td>
		<td align="left">mremap  </td>
		<td align="left">re-map a virtual memory address</td>
		<td align="left">mm/mremap.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="30" sdnum="1033;">30</td>
		<td align="left">brk  </td>
		<td align="left">change the amount of space allocated for the calling process's data segment</td>
		<td align="left">mm/mmap.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="31" sdnum="1033;">31</td>
		<td align="left">recvfrom  </td>
		<td align="left">used to receive messages from a socket</td>
		<td align="left">net/socket.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="32" sdnum="1033;">32</td>
		<td align="left">recvmsg  </td>
		<td align="left">used to receive messages from a socket</td>
		<td align="left">net/socket.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="33" sdnum="1033;">33</td>
		<td align="left">sendmsg  </td>
		<td align="left">used to transmit a message to another socket</td>
		<td align="left">net/socket.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="34" sdnum="1033;">34</td>
		<td align="left">sendto  </td>
		<td align="left">used to transmit a message to another socket.</td>
		<td align="left">net/socket.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="35" sdnum="1033;">35</td>
		<td align="left">fstat  </td>
		<td align="left">get file status </td>
		<td align="left">fs/stat.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="36" sdnum="1033;">36</td>
		<td align="left">select  </td>
		<td align="left">sync. I/O multiplexing</td>
		<td align="left">fs/select.c</td>
	</tr>
	<tr>
		<td height="32" align="left" sdval="37" sdnum="1033;">37</td>
		<td align="left">set_robust_list  </td>
		<td align="left">requests the kernel to record the head of the list of robust futexes owned by the calling thread</td>
		<td align="left">kernel/futex_compat.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="38" sdnum="1033;">38</td>
		<td align="left">restart_syscall  </td>
		<td align="left">returns the return value of the system call it restarts</td>
		<td align="left">net/core/net-sysfs.c</td>
	</tr>
	<tr>
		<td height="17" align="left" sdval="39" sdnum="1033;">39</td>
		<td align="left">close</td>
		<td align="left">close a file descriptor</td>
		<td align="left">fs/open.c</td>
	</tr>
</table>



# Quick Demo

https://autoclave.run has a pretty good demonstration of the ultimate vision here,
but here's some quick steps to get up and running yourself.

(This is very early, only supporting firewall-on-VNC connection in this patchset)

(No, I haven't tested this yet.  RELENG!)

    # apt-get install vde2 libvde-dev libvdeplug-dev libseccomp-dev git
    ### You need to add deb-src for deb line in /etc/apt/sources.list,
    ### because Reasons
    # apt-get update
    # apt-get build-dep qemu
    # git clone https://github.com/qemu/qemu.git
    # cd qemu
    # for i in ../patches/*; do patch -p1 < $i; done
    # ./configure --prefix=/usr --enable-vde --enable-seccomp --target-list=x86_64-softmmu
    # make
    # make install
    ### Easily the most dangerous thing going on, assuming you're running
    ### a multiuser system with users who aren't coming in through Autoclave.
    ### Assuming.
    # chmod 0777 /dev/kvm 

    ### Now, you don't need to be root for actual execution:
    $ vde_switch -d -s user.sock
    $ slirpvde -d -dhcp -s user.sock
    $ wget http://mirrors.xmission.com/linuxmint/iso//stable/18/linuxmint-18-mate-64bit.iso 
    $ qemu-system-x86_64 -cdrom linuxmint-18-mate-64bit.iso -m 4G -vnc :25,lossy -net nic -net \
      vde,sock=user.sock -global kvm-apic.vapic=false -smp 4,sockets=2,cores=2,threads=1 \
      -vga qxl -enable-kvm -monitor stdio
    QEMU 2.7.50 monitor - type 'help' for more information
    (qemu) migrate "exec: ls"
    (qemu) 
    ### Yes, arbitrary code execution in QEMU Hypervisor Shell works like this :)
    ### If you've got a VNC link, Control Alt 2 is often full of surprises.
    ### Drop the monitor element and you get this through VNC.
    ### But now, let's connect.  (TODO, insert nice pretty screenshot.)
    (qemu) migrate "exec: ls"
    Unable to open /dev/null: Permission denied


    

    
# TODO

1. Comprehensively document the exposed syscalls, memory maps, and file handles
2. Modify QEMU to reduce #1 to bare minimums, at least for specified performance levels
3. Manage exposure of /dev/kvm to untrusted users, modifying the module if necessary
4. Integrate dirtycow exploit into QMP, so we can see whether KVM_RUN semantics
   do or do not suppress thread races
5. Determine whether snapshot/loadvm conflict is resolved in QEMU Master, and if not,
   integrate patches.
6. Move apply_autoclave out of vnc.c (sigh) and into somewhere it can be called,
   somewhat generically, either right before VM start (in vl.c) or on network
   connection (vnc and spice)
7. Dive back into the monolithic beast that is libvirt, because we actually do
   want to work with the rest of the ecosystem
8. Figure out proper performance engineering, use of hugepages (explicit/transparent/
   blocked).
9. Finish extraction and demonstration to level of https://autoclave.run, i.e. actually
   show use of bypass_shared_memory
10. Make https://autoclave.run a thing that is scalable
11. Make Autoclave work well under nested virtualization, or at least understand
    what that would take (probably guarantees around linearized memory mapping)
12. Resolve precisely when xinetd should receive a VNC connection string, rather
    than just spitballing it.  Connections trigger the autoclave lockdown, so
    this does matter.
13. Integrate with Intel Clear Containers
14. Integrate with Docker
15. Map out how to host web servers like Guacamole, instead of VNC/Spice like
    interfaces.
16. Replace slirpvde/vde_switch with a safe daemon (one of the obvious ways to 
    attack the present implementation)
17. Expand this document!
