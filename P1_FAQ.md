## **Commonly Asked Questions (FAQ)**

**How do I open the VM?**  
You cannot open an ISO by double-clicking. Use your VM software (VMware Fusion or VMware Workstation) to create a VM from the ISO. Follow the instructions in the PDF provided.

**What are the recommended external dependencies for this project?**  
No specific guides are provided besides the CMakeLists files. Check the GitHub pages of the libraries you use for installation and documentation. This approach prepares you for using external libraries in future projects.

**How should I run `mnbash` and `sudo mn`?**  
Run them in separate terminals after SSH’ing into the VM. Follow the spec for using `git` or `scp` to copy project files.

**What is Mininet?**  
Mininet interfaces with the Linux kernel to simulate networks. It cannot run natively on Mac or Windows, which is why the VM is needed.

**Why are we using a VM?**  
The VM provides a Linux environment where Mininet and other networking tools work correctly, isolating dependencies and ensuring consistency.

**Do I always need to connect remotely to the VM?**  
Ideally, set up a remote development environment (e.g., VSCode Remote Development). This allows you to work directly on your VM without manually SSH’ing every time.

**Where should I create my files and write code?**  
Canonically, this is done in the `src/` folder. As long as your project compiles with CMake from the `cpp/` directory, the exact location is flexible.

**What is the purpose of `CMakeLists.txt`?**  
It manages project compilation. You may add setup code, define executables, and specify required compiler features. Learn the basics of CMake to understand how your code will be compiled.

**For rate calculations, should I include 1-byte RTT messages?**  
No, only include the 80KB chunks of data. Ignore all other messages when calculating bandwidth.

**My backspace doesn’t work in Bash. What’s wrong?**  
This is a terminal setting mismatch. The client sends a backspace code different from what the host expects. Adjust terminal settings to fix it.

**What’s the difference between `add_executable` / `set_compile_features` and using `SET` in CMake?**  
`add_executable` creates a build target for your program. `set_compile_features` sets required language features. Using `SET` is just defining variables and doesn’t create a build target.

**My SSH session times out after 5 minutes. How do I fix it?**  
Enable SSH keepalives with:  
ssh \-o "ServerAliveInterval 60" \<SERVER\_ADDRESS\>

To apply globally, add `ServerAliveInterval 60` to `/etc/ssh/ssh_config` or `~/.ssh/config`.

**Should I use switches as endpoints for bandwidth measurements?**  
No, always measure bandwidth between hosts, not switches.

**Why am I seeing negative Mbps in iPerfer?**  
You are likely not using the `MSG_WAITALL` flag when receiving. This flag ensures the receiver gets the full amount of requested bytes.

**My program stops too early or too late when measuring bandwidth.**  
Stop timing after the last 80KB data is received, **before** the ACK is sent. The ACK is negligible and won’t affect your bandwidth calculation significantly.

**My bandwidth results differ between iPerf and iPerfer. Why?**  
iPerf saturates the link to measure throughput. iPerfer waits for application-level ACKs for 80KB chunks and uses RTT to estimate bandwidth. Differences are expected due to methodology.

**Why does my server keep running even after the client is done?**  
Possible causes:

1. Not using `MSG_WAITALL` on receive, so the buffer isn’t fully read.  
2. Not sending in a loop, so the full buffer isn’t transmitted.  
3. Address size not initialized correctly in `accept`.  
   These can create network backlogs and extend server runtime.

**My bandwidth calculation seems off.**  
Common errors:

1. Integer division instead of double division.  
2. Rounding RTT values before using them. Use the raw measured RTT values for accurate calculation.

---

## **Advice & Guidance**

### **VM & Environment**

* Use VMware Fusion or Workstation to create the VM from the ISO; don’t double-click it.  
* VM provides Linux for Mininet and networking tools; can’t run natively on Mac/Windows.  
* Use SSH or remote development tools (e.g., VSCode Remote) to work inside the VM.

### 

### **File & Code Setup**

* Place code in `src/` folder; compilation should work from `cpp/` with CMake.  
* `CMakeLists.txt` defines executables and compiler features (`add_executable`, `set_compile_features`).  
* Don’t confuse `SET` (just a variable) with creating a build target.


### **Running the Project**

* Run `mnbash` and `sudo mn` in separate terminals after SSH’ing into VM.  
* Measure bandwidth **between hosts only**, not switches.

**Network/Rate Calculations**

* Include **only 80KB data chunks** in bandwidth calculations; ignore RTT messages or ACKs.  
* Stop timing **after last data chunk received**, before sending final ACK.  
* If results differ from iPerf: iPerfer uses 80KB chunks \+ RTT, iPerf saturates the link—methodology differs.

**Common Network Bugs**

* Negative Mbps? Likely missing `MSG_WAITALL` on receive.  
* Server runs too long? Check:  
  * `MSG_WAITALL` not used,  
  * Not sending full buffer in loop,  
  * `accept` address size not initialized.

**SSH / Terminal Issues**

* Backspace not working → terminal mismatch; adjust client/server settings.

SSH timeout after 5 min → enable keepalives:  
`ssh -o "ServerAliveInterval 60" <SERVER_ADDRESS>`

* Or set globally in `~/.ssh/config`.

**Calculation Errors**

* Use **double division**, not integer division.  
* Don’t round RTT values before using; use raw measurements