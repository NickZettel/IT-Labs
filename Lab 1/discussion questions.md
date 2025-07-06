
### Why does Windows Server Core use a command-line interface instead of a GUI?

Windows Server Core uses a command-line interface to reduce resource usage and improve performance, which may be critical on busy servers. Removing these features also improves security by reducing the number of attack surfaces for malicious actors to exploit.

### How does PowerShell differ from the traditional command prompt (cmd.exe)?

There are many differences between the two. PowerShell was made with the open source .Net Core Framework, allowing it to run on Windows, Linux, and MacOS, while CMD runs only on Windows.  CMD outputs plain text, whereas Powershell outputs .Net objects. Powershell scripting(.ps1) is more advanced, supporting functions, loops, and conditionals, whereas CMD(.bat) supports basic commands. Still, many legacy systems still rely on CMD, and it remains ideal for basic tasks.

### What challenges did you face when navigating Server Core for the first time?

There was very little navigating of Server Core involved since the lab immediately directed to PowerShell. However, due to the GUI-less nature, my first question was to be sure this CLI menu was in fact what I should be seeing and not something else.
