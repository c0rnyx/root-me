<h1>Web Application Exploitation and Privilege Escalation</h1>

<h2>Overview</h2>
<p>
This repository documents an authorized, lab-based penetration test against a Linux web server.
The assessment demonstrates a complete attack chain starting from external enumeration,
leading to remote code execution via a file upload vulnerability, and concluding with
local privilege escalation through a misconfigured SUID binary.
</p>

<hr>

<h2>Target Information</h2>
<ul>
  <li><strong>Target IP:</strong> 10.82.132.134 </li>
  <li><strong>Operating System:</strong> Ubuntu 20.04 LTS</li>
  <li><strong>Kernel:</strong> Linux 5.15.0-139-generic (x86_64)</li>
  <li><strong>Web Server:</strong> Apache HTTP Server 2.4.41</li>
  <li><strong>Open Ports:</strong>
    <ul>
      <li>22/tcp (SSH)</li>
      <li>80/tcp (HTTP)</li>
    </ul>
  </li>
  <li><strong>Assessment Type:</strong> Authorized lab environment</li>
</ul>

<hr>

<h2>Executive Summary</h2>
<p>
The target system was compromised through insufficient server-side validation in a file upload
functionality exposed by a web application. This flaw enabled remote code execution as the
<code>www-data</code> user. Subsequent local enumeration revealed a misconfigured SUID Python binary,
which was abused to escalate privileges and obtain full root access.
</p>

<hr>

<h2>Enumeration</h2>

<h3>Network Enumeration</h3>
<p>An initial Nmap scan was performed to identify exposed services.</p>

<pre><code>nmap -sS -sV -sC 10.82.132.134 </code></pre>
<img src="https://i.imgur.com/DUOI3XY.png" height="90%" width="90%" alt="Nmap Scan Results"/>

<p><strong>Results:</strong></p>
<ul>
  <li>Apache HTTP Server running on port 80</li>
  <li>OpenSSH running on port 22</li>
</ul>

<hr>

<h3>Web Enumeration</h3>
<p>Directory brute-forcing was conducted against the HTTP service.</p>

<pre><code>gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.82.132.134 </code></pre> 
<img src="https://i.imgur.com/exdajF9.png" height="80%" width="80%" alt="Gobuster Bruteforce"/>
<p><strong>Discovered directory:</strong></p>
<pre><code>/panel</code></pre>

<hr>

<h2>Vulnerability Analysis</h2>

<h3>File Upload Validation Bypass</h3>

<p>
The <code>/panel</code> directory exposed a file upload functionality that was assessed through
systematic manual testing. Multiple upload bypass techniques were evaluated, including
alternative file extensions, mixed-case variations, and non-standard PHP extensions.
</p>

<p>
Initial attempts using common executable extensions such as <code>.php</code> and <code>.phtml</code>
were correctly rejected by the server. Additional testing was performed using lesser-known
PHP-related extensions to evaluate the robustness of the server-side validation.
</p>

<p>
Through this iterative testing process, it was determined that files with the
<code>.php5</code> extension were accepted and executed by the server, indicating that validation
was based solely on a weak, extension-based filtering mechanism.
</p>
<img src="https://imgur.com/upepYos.png" height="50%" width="50%" alt="Successful upload"/>

<hr>

<h2>Initial Access</h2>

<p>
A PHP reverse shell was uploaded using the <code>.php5</code> extension.
Upon accessing the uploaded file, a reverse shell connection was received.
</p>

<pre><code>nc -lvnp 1234</code></pre>

<p>
The shell was obtained as the <code>www-data</code> user.
</p>
<img src="https://imgur.com/wm1W48n.png" height="80%" width="80%" alt="www-data user"/>
<hr>

<h2>Shell Stabilization</h2>

<p>
The initial shell lacked job control and was upgraded to a fully interactive Bash shell.
</p>

<pre><code>python -c 'import pty; pty.spawn("/bin/bash")'</code></pre>
<img src="https://imgur.com/mievAV3.png" height="50%" width="50%" alt="stabilizing the shell"/>
<hr>

<h2>Local Enumeration</h2>

<h3>User Flag Discovery</h3>

<pre><code>find / -name user.txt 2&gt;/dev/null</code></pre>

<p><strong>Result:</strong></p>

<pre><code>/var/www/user.txt</code></pre>
<img src="https://imgur.com/bXZgiap.png" height="70%" width="70%" alt="getting user flag"/>
<hr>

<h2>Privilege Escalation</h2>

<h3>SUID Enumeration</h3>

<pre><code>find / -perm -4000 -type f 2&gt;/dev/null</code></pre>
<p>
Among standard SUID binaries, the following misconfiguration was identified:
</p>

<pre><code>/usr/bin/python2.7</code></pre>
<img src="https://imgur.com/qwulNND.png" height="70%" width="70%" alt="python SUID found"/>
<hr>

<h3>SUID Python Exploitation</h3>

<p>
Python interpreters should never be configured with the SUID bit.
This misconfiguration allowed arbitrary command execution with root privileges.
</p>

<pre><code>python -c 'import os; os.execl("/bin/sh", "sh", "-p")'</code></pre>
<img src="https://imgur.com/Wbf9hXU.png" height="50%" width="50%" alt="abusing the misconfigured SUID"/>
<hr>

<h2>Result</h2>
<ul>
  <li>Root shell obtained</li>
  <li>Full system compromise achieved</li>
  <img src="https://imgur.com/UGuyNrV.png" height="70%" width="70%" alt="Root access"/>

</ul>

<hr>

<h2>Impact</h2>
<ul>
  <li>Remote Code Execution</li>
  <li>Privilege Escalation to Root</li>
  <li>Complete loss of system confidentiality, integrity, and availability</li>
</ul>

<hr>

<h2>Recommendations</h2>
<ul>
  <li>Enforce strict server-side file upload validation</li>
  <li>Remove the SUID bit from Python binaries</li>
  <li>Store uploaded files outside the web root</li>
  <li>Perform regular SUID audits</li>
  <li>Apply the principle of least privilege</li>
</ul>

<hr>

<h2>Attack Chain Summary</h2>

<pre><code>
Nmap
 → Gobuster
 → File Upload Bypass (.php5)
 → Reverse Shell (www-data)
 → SUID Enumeration
 → SUID Python Abuse
 → Root Access
</code></pre>

<hr>

<h2>Disclaimer</h2>
<p>
This project was completed in an <strong>authorized, educational lab environment</strong>.
All techniques demonstrated are intended solely for learning and defensive security improvement.
</p>
