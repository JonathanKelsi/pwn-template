# Pwn Template

My template for pwnable CTFs.

**main.py:**
```python
from typing_extensions import ParamSpec
from typing import List
from pwn import *
import os

from exploit import exploit

CWD: str = "../run/"
EXE: ELF = ELF(CWD + "challenge")
LIBC: ELF = ELF(CWD + "libc.so.6")

HOST: str = "pwn.ctf.cyber"
PORT: int = 1234

GDBSCRIPT: str = """
handle SIGALRM ignore
continue
""".format(**locals())

LOG_EVERY: int = 100

P = ParamSpec('P')

def start_local(argv: List[str] = [], *a: P.args, **kw: P.kwargs) -> tube:
    """Execute the target binary locally."""
    if args.GDB:
        return gdb.debug([EXE.path] + argv, *a, gdbscript=GDBSCRIPT, api=True, cwd=CWD, **kw)
    else:
        return process([EXE.path] + argv, *a, cwd=CWD, **kw)


def start_remote() -> tube:
    """Connect to the process on the remote host."""
    return remote(HOST, PORT)
    
    
def start(argv: List[str] = [], *a: P.args, **kw: P.kwargs) -> tube:
    """Start the exploit against the target."""
    if args.LOCAL:
        return start_local(argv, *a, **kw)
    else:
        return start_remote()


def brute_force() -> None:
    """Brute force the exploit until successful, logging every LOG_EVERY attempts."""
    counter = 0
    while True:
        for _ in range(LOG_EVERY):
            if exploit(start, EXE, LIBC):
                break
        else:
            counter += LOG_EVERY
            log.info(term.text.bold_yellow("Attempt: {}".format(counter)))
            continue
        break


def main() -> None:
    """Main function"""
    # Setup context
    context.binary = EXE
    context.terminal = ["tmux", "splitw", "-h"]
    os.environ["PWNLIB_COLOR"] = "always"
    
    # Execute exploit
    if args.SPAM:
        brute_force()
    else:
        exploit(start, EXE, LIBC)


if __name__ == "__main__":
    main()
```
**exploit.py:**
```python
from typing import Callable
from pwn import *


def _exploit(io: tube, exe: ELF, libc: ELF) -> bool:
    """Must not except."""
    pass


def exploit(start: Callable[..., tube], exe: ELF, libc: ELF) -> bool:
    """Run the exploit once, and return a status weather the exploit worked."""
    io = start() # <- change start arguments
    status = _exploit(io, exe, libc)
    io.close()
    return status
```