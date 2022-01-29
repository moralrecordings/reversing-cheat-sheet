
IBM PC + DOS
############




Setting up DOSBox
=================

You need to make a build of DOSBox with the heavy debugging stack switched on. In addition you want to disable the fast JIT/pipelined CPU cores, as the debugger will jump around like crazy if they're switched on.


.. code-block:: bash

   ./configure --enable-debug=heavy --disable-dynamic-core && make -j4
   ./src/dosbox

DOSBox will start with a debugger view taking over the terminal window. You can switch to the debugger by pressing Left Alt + Pause.


.. code-block:: text

   Debugger commands (enter all values in hex or as register):
   Commands------------------------------------------------
   BP     [segment]:[offset] - Set breakpoint.
   BPINT  [intNr] *          - Set interrupt breakpoint.
   BPINT  [intNr] [ah] *     - Set interrupt breakpoint with ah.
   BPINT  [intNr] [ah] [al]  - Set interrupt breakpoint with ah and al.
   BPM    [segment]:[offset] - Set memory breakpoint (memory change).
   BPPM   [selector]:[offset]- Set pmode-memory breakpoint (memory change).
   BPLM   [linear address]   - Set linear memory breakpoint (memory change).
   BPLIST                    - List breakpoints.
   BPDEL  [bpNr] / *         - Delete breakpoint nr / all.
   C / D  [segment]:[offset] - Set code / data view address.
   DOS MCBS                  - Show Memory Control Block chain.
   INT [nr] / INTT [nr]      - Execute / Trace into interrupt.
   LOG [num]                 - Write cpu log file.
   LOGS/LOGL/LOGC [num]      - Write short/long/cs:ip-only cpu log file.
   HEAVYLOG                  - Enable/Disable automatic cpu log when dosbox exits.
   ZEROPROTECT               - Enable/Disable zero code execution detection.
   SR [reg] [value]          - Set register value.
   SM [seg]:[off] [val] [.]..- Set memory with following values.
   IV [seg]:[off] [name]     - Create var name for memory address.
   SV [filename]             - Save var list in file.
   LV [filename]             - Load var list from file.
   ADDLOG [message]          - Add message to the log file.
   MEMDUMP [seg]:[off] [len] - Write memory to file memdump.txt.
   MEMDUMPBIN [s]:[o] [len]  - Write memory to file memdump.bin.
   SELINFO [segName]         - Show selector info.
   INTVEC [filename]         - Writes interrupt vector table to file.
   INTHAND [intNum]          - Set code view to interrupt handler.
   CPU                       - Display CPU status information.
   GDT                       - Lists descriptors of the GDT.
   LDT                       - Lists descriptors of the LDT.
   IDT                       - Lists descriptors of the IDT.
   PAGING [page]             - Display content of page table.
   EXTEND                    - Toggle additional info.
   TIMERIRQ                  - Run the system timer.
   HELP                      - Help
   Keys------------------------------------------------
   F3/F6                     - Previous command in history.
   F4/F7                     - Next command in history.
   F5                        - Run.
   F9                        - Set/Remove breakpoint.
   F10/F11                   - Step over / trace into instruction.
   ALT + D/E/S/X/B           - Set data view to DS:SI/ES:DI/SS:SP/DS:DX/ES:BX.
   Escape                    - Clear input line.
   Up/Down                   - Move code view cursor.
   Page Up/Down              - Scroll data view.
   Home/End                  - Scroll log messages.
   



Interrupts
----------

The DOS API uses software interrupts to .

These are not to be confused with Interrupt Requests, or IRQ. An IRQ is a hardware 

A large index of standard DOS interrupts can be found `here <http://stanislavs.org/helppc/idx_interrupt.html>`_.


Ports and DMA
-------------

DOS has an extremely minimal hardware abstraction layer. A DOS program must bundle its own drivers for graphics, sound and peripheral support.






System
------

=========== ==================== ====================================== ========================================================================
Interrupt   Function code        Description                            Modern pseudocode
=========== ==================== ====================================== ========================================================================
int 21h     ah = 25h             Set an interrupt handler               _dos_setvect(al function_code, ds:dx \*function);
int 21h     ah = 35h             Get an interrupt handler               es:bx = _dos_getvect(al function_code);
int 21h     ah = 4ch             Exit with return code                  exit(al status);
=========== ==================== ====================================== ========================================================================



Real-Time Clock
---------------

=========== ==================== ====================================== ========================================================================
Interrupt   Function code        Description                            Modern pseudocode
=========== ==================== ====================================== ========================================================================
int 21h     ah = 2ah             Get current date                       tm = \*localtime(); cx = tm.tm_year+1900; dh = tm.mon+1; dl = tm.mday;
int 21h     ah = 2ch             Get current time                       tm = \*localtime(); ch = tm.tm_hour; cl = tm.tm_min; dh = tm.tm_sec;
=========== ==================== ====================================== ========================================================================


Memory
------

=========== ==================== ====================================== ========================================================================
Interrupt   Function code        Description                            Modern pseudocode
=========== ==================== ====================================== ========================================================================
int 21h     ah = 48h             Allocate memory                        ax:0000 = malloc(bx*16 size);
int 21h     ah = 49h             Free allocated memory                  free(es);
=========== ==================== ====================================== ========================================================================


Console I/O
-----------

=========== ==================== ====================================== ========================================================================
Interrupt   Function code        Description                            Modern pseudocode
=========== ==================== ====================================== ========================================================================
int 21h     ah = 01h             Read char from stdin with echo         al = getc(stdin);
int 21h     ah = 02h             Write char to stdout with echo         al = putc(dl, stdout);
=========== ==================== ====================================== ========================================================================



Filesystem I/O
--------------

=========== ==================== ====================================== ========================================================================
Interrupt   Function code        Description                            Modern pseudocode
=========== ==================== ====================================== ========================================================================
int 21h     ah = 0dh             Flush buffers to disk                  sync();
int 21h     ah = 0eh             Set current drive                      _chdrive(dl drive);
int 21h     ah = 19h             Get current drive                      al = _getdrive();
int 21h     ah = 39h             Create a directory                     mkdir(ds:dx \*pathname);
int 21h     ah = 3ah             Remove a directory                     rmdir(ds:dx \*pathname);
int 21h     ah = 3bh             Set current directory                  chdir(ds:dx \*pathname);
int 21h     ah = 3ch             Create an empty file                   creat(ds:dx \*pathname);
int 21h     ah = 3dh, al = 00h   Open a file read only                  ax = fopen(ds:dx \*pathname, "r");
int 21h     ah = 3dh, al = 01h   Open a file write only (truncated)     ax = fopen(ds:dx \*pathname, "w");
int 21h     ah = 3dh, al = 02h   Open a file read-write                 ax = fopen(ds:dx \*pathname, "r+");
int 21h     ah = 3eh             Close a file                           fclose(bx \*fp);
int 21h     ah = 3fh             Read from a file                       fread(ds:dx \*ptr, cx size, 1, bx \*fp);
int 21h     ah = 40h             Write to a file                        ax = fwrite(ds:dx \*ptr, cx size, 1, bx \*fp);
int 21h     ah = 41h             Remove a file                          unlink(ds:dx \*pathname);
int 21h     ah = 42h, al = 00h   Seek from start of a file              dx:ax = lseek(bx \*fp, cx:dx offset, SEEK_SET);
int 21h     ah = 42h, al = 01h   Seek from current position in a file   dx:ax = lseek(bx \*fp, cx:dx offset, SEEK_CUR);
int 21h     ah = 42h, al = 02h   Seek from end of a file                dx:ax = lseek(bx \*fp, cx:dx offset, SEEK_END);
int 21h     ah = 47h             Get current working directory          _getdcwd(dl drive, ds:si \*buffer, 64);
int 21h     ah = 56h             Rename a file                          rename(ds:dx \*oldpath, es:di \*newpath);
=========== ==================== ====================================== ========================================================================


Mouse
-----

=========== ==================== ====================================== ===========================================================================
Interrupt   Function code        Description                            Modern pseudocode
=========== ==================== ====================================== ===========================================================================
int 33h     ah = 03h             Get mouse position and buttons         cx = mouse.x; dx = mouse.y; bx = mouse.button_right*2 + mouse.button_left;
int 33h     ah = 07h             Set min/max horizontal mouse position  mouse.x_min = cx; mouse.x_max = dx;
int 33h     ah = 08h             Set min/max vertical mouse position    mouse.y_min = cx; mouse.y_max = dx;


=========== ==================== ====================================== ===========================================================================


Graphics
--------

http://www.ctyme.com/intr/rb-0069.htm

Set Video Mode

=========== ============= ============ ============== ============= ======== ========= ====== ==========
Interrupt   Function code Display mode Type           Graphical res Text res Colours   Planes FB segment
=========== ============= ============ ============== ============= ======== ========= ====== ==========
int 10h     ah = 00h      al = 00h     Text           360x400       40x25    16 gray   1      B800
int 10h     ah = 00h      al = 01h     Text           360x400       40x25    16        1      B800
int 10h     ah = 00h      al = 02h     Text           720x400       80x25    16 gray   1      B800
int 10h     ah = 00h      al = 03h     Text (default) 720x400       80x25    16        1      B800
int 10h     ah = 00h      al = 04h     CGA            320x200       40x25    4         2      B800
int 10h     ah = 00h      al = 05h     CGA            320x200       40x25    4 gray    2      B800
int 10h     ah = 00h      al = 06h     CGA            640x200       80x25    2         1      B800
int 10h     ah = 00h      al = 07h     Text           720x400       80x25    2         1      B800
int 10h     ah = 00h      al = 0dh     EGA            320x200       40x25    16        4      A000
int 10h     ah = 00h      al = 0eh     EGA            640x200       80x25    16        4      A000
int 10h     ah = 00h      al = 0fh     EGA            640x350       80x25    2         1      A000
int 10h     ah = 00h      al = 10h     EGA            640x350       80x25    16        4      A000
int 10h     ah = 00h      al = 11h     VGA            640x480       80x30    2         1      A000
int 10h     ah = 00h      al = 12h     VGA            640x480       80x30    16        4      A000
int 10h     ah = 00h      al = 13h     VGA            320x200       40x25    256       1      A000
int 10h     ax = 4f02h    bx = 0100h   VESA           640x400       80x25    256       1      A000
int 10h     ax = 4f02h    bx = 0101h   VESA           640x480       80x30    256       1      A000
int 10h     ax = 4f02h    bx = 0102h   VESA           800x600       100x37   16        1      A000
int 10h     ax = 4f02h    bx = 0103h   VESA           800x600       100x37   256       1      A000
=========== ============= ============ ============== ============= ======== ========= ====== ==========

Useful video controls

=========== ==================== ====================================== ===========================================================================
Interrupt   Function code        Description                            Modern pseudocode
=========== ==================== ====================================== ===========================================================================
int 10h     ah = 0fh             Get current video mode                 ah = video.text_res.w; al = video.display_mode; bh = video.active_page;
int 10h     ax = 4f01h           Get VESA video mode info               get_vesa_mode_info(cx mode, es:di \*dst);
=========== ==================== ====================================== ===========================================================================


=========== ==================== ============================  ===== ===============================
Interrupt   Function code        Description                   bl    Detail
=========== ==================== ============================  ===== ===============================
int 10h     ax = 1a00h           Get display combination code  00h   No display
                                                               01h   MDA/HERC w/mono display
                                                               02h   CGA w/colour display
                                                               04h   EGA w/colour display
                                                               05h   EGA w/mono display
                                                               07h   VGA w/mono analog display
                                                               08h   VGA w/colour analog display
                                                               0ah   MCGA w/colour digital display
                                                               0bh   MCGA w/mono analog display
                                                               0ch   MCGA w/colour analog display
                                                               ffh   Unknown
=========== ==================== ============================  ===== ===============================

http://www.phatcode.net/res/224/files/html/index.html
http://www.osdever.net/FreeVGA/vga/portidx.htm

Palette operations

=========== ========= =========================== ================= =============================================================
Port        Mode      Description                 Bit               Purpose
=========== ========= =========================== ================= =============================================================
3c7h        In        Palette state               1 0               0: prepared to accept reads
                                                                    3: prepared to accept writes
3c7h        Out       Palette address read mode   7 6 5 4 3 2 1 0   Index to start reading palette entries into the data register
3c8h        In/Out    Palette address write mode  7 6 5 4 3 2 1 0   Index to start writing palette entries into the data register
3c9h        In/Out    Palette data register       5 4 3 2 1 0       Palette data. 1/colour for EGA, 3/colour for VGA
=========== ========= =========================== ================= =============================================================



========== =========== ==============================
Port       Bit         Purpose
========== =========== ==============================
3ceh       3           Read mode:
           2           Test condition
           1 0         Write mode (0-2)
========== =========== ==============================



========== =========== ==============================
Port       Bit         Purpose
========== =========== ==============================
3dah       3           Vertical retrace is happening
           2           Light pen switch is open
           1           Light pen switch is triggered
           0           Display is enabled
========== =========== ==============================


