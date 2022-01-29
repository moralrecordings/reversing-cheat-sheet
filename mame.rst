
Setting up MAME
###############

Stock MAME includes a full debugger for the emulated systems; this can be invoked with ``mame <target> -debug``. For basic reversing activities, this is usually enough. 

Eventually though, you will need to access the internals of MAME, which means making a build with debugging symbols enabled. The caveat is because the size of the codebase is massive, you will need to pick out the machine(s) you want to target. The machine is a single .cpp file in src/mame/drivers containing GAME() or CONS() entries for the system you want; it's visible on the splash screen MAME shows before starting the emulator.

::

   make SUBTARGET=mess OVERRIDE_CC=clang OVERRIDE_CXX=clang++ DEBUG=1 SYMBOLS=1 SYMLEVEL=2 NOWERROR=1 SANITIZE=address TOOLS=1 VERBOSE=1 PYTHON_EXECUTABLE=/usr/bin/python3 SOURCES=src/mame/drivers/cdi.cpp -j12

- SUBTARGET: Either ``arcade`` (arcade machines) or ``mess`` (everything else).
- OVERRIDE_CC/CXX: Use clang to build instead of GCC. Needed for the SANITIZE option.
- DEBUG: Build with debug settings, not release.
- SYMBOLS: Include symbols with the binary.
- SYMLEVEL: Include source code references with the binary.
- NOWERROR: Throw an error if there's a C++ warning. Good sanity check before sending a patch upstream.
- SANITIZE: Build with Clang's address sanitizer. Good sanity check before sending a patch upstream.
- TOOLS: Build all of the extra command line tools. 
- VERBOSE: Show the commands being run during the build.
- PYTHON_EXECUTABLE: Should be /usr/bin/python3 for recent versions.
- SOURCES: Comma separated list of all the machine .cpp files you want to include.

Be sure to run ``make clean`` before changing any of the above options.


Debug logging
#############

The MAME source code has a lot of helpful log annotation that is stripped at compile time. Unfortunately, this has to be enabled per file. Most .cpp files will have a little section at the start that looks similar to this:

::

   #define LOG_DECODES     (1 << 1)
   #define LOG_SAMPLES     (1 << 2)
   #define LOG_COMMANDS    (1 << 3)
   #define LOG_SECTORS     (1 << 4)
   #define LOG_IRQS        (1 << 5)
   #define LOG_READS       (1 << 6)
   #define LOG_WRITES      (1 << 7)
   #define LOG_UNKNOWNS    (1 << 8)
   #define LOG_RAM         (1 << 9)
   #define LOG_ALL         (LOG_DECODES | LOG_SAMPLES | LOG_COMMANDS | LOG_SECTORS | LOG_IRQS | LOG_READS | LOG_WRITES | LOG_UNKNOWNS | LOG_RAM)

   #define VERBOSE         (0)
   #include "logmacro.h"

In order to enable logging for the various types, you must change the ``#define VERBOSE (0)`` line to include a bitwise OR of the various flags, then rebuild. An example would be ``#define VERBOSE (LOG_IRQS | LOG_COMMANDS)`` to enable logging on IRQs and commands, or ``#define VERBOSE (LOG_ALL)`` to log everything.

In addition, these logs will only show up if logging is enabled from the command line. Add the ``-oslog`` flag to send the logs to stdout/stderr, or ``-log`` to write to an error.log file in the current directory.
