# Pwn Template

My template for pwn CTFs.

```python
from pwn import *
import os

# Pwnlib settings
os.environ["PWNLIB_COLOR"] = "always"

# Set up pwntools for correct arch, library and 
exe = context.binary = ELF('')
libc = ELF('')
ld = ELF('')

# Connection details for challenge host & port
host = ''
port = 9000

# Define timeout interval for I/O operations
TIMEOUT = 1

# Connect to remote SSH server if there's one
shell = None
remote_path = ''
if args.SSH:
    shell = ssh(user='', host=host, port=22, password='')
    shell.set_working_directory(symlink=True)

def start_local(argv=[], *a, **kw):
    '''Execute the target binary locally'''
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe.path] + argv, *a, **kw)

def start_ssh(argv=[], *a, **kw):
    '''Execute the target binary on the remote host'''
    if args.GDB:
        return gdb.debug([remote_path] + argv, gdbscript=gdbscript, ssh=shell, *a, **kw)
    else:
        return shell.process([remote_path] + argv, *a, **kw)

def start_remote():
    '''Connect to the process on the remote host'''
    return remote(host, port)
    
def start(argv=[], *a, **kw):
    '''Start the exploit against the target.'''
    if args.LOCAL:
        return start_local(argv, *a, **kw)
    elif args.SSH:
        return start_ssh(argv, *a, **kw)
    else:
        return start_remote()

# Gdbscript for debugging
gdbscript = '''
tbreak *0x{exe.entry:x}
continue
'''.format(**locals())

# Choose whether to use aslr or not
context.aslr = True

# Terminal
# context.terminal = context.terminal = ["tmux","new-window"]
context.terminal = ["tmux", "splitw", "-h"]

#===========================================================
#                    EXPLOIT GOES HERE
#===========================================================

def exploit(io):
    ...
    
def run_once():
    ''' Simply, run the exploit once '''
    try:
        io = start()
        exploit(io)
        io.interactive()
    except Exception as e:
        log.failure(term.text.bold_red(str(e)))
    finally:
        io.close()

def _bf_run_once():
    ''' Run the exploit once, return True if successful, False otherwise '''
    try:
        exploited = True
        io = start()
        exploit(io)
        io.sendline(b'cat /home/*/flag*')
        io.sendline(b'echo AAAAAAA')
        flag = io.recvuntil(b'AAAAAAA', timeout=TIMEOUT)[:-8].decode()
        if not flag: raise Exception('Failed to retrieve flag')
        success(term.text.bold_italic_green(flag))
        io.interactive()
    except:
        exploited = False
    finally:
        io.close()
        return exploited

def brute_force():
    ''' Brute force the exploit until successful, logging every ITERS attempts '''
    ITERS, counter = 100, 0
    while True:
        for _ in range(ITERS):
            if _bf_run_once():
                break
        else:
            counter += 1
            info(term.text.bold_yellow('Attempt: {}'.format(counter * ITERS)))
            continue
        break
    
def main():
    ''' Main function '''
    if args.SPAM:
        brute_force()
    else:
        run_once()

if __name__ == "__main__":
    main()

```
