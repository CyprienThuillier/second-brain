
### Launch Metasploit :

```bash
msfconsole
```


### Modules :

Look at the [[Cybersecurity Basics]] for specific terms.

#### Auxiliary :

Any supporting module, such as scanners, crawlers and fuzzers, can be found here.

```markup
auxiliary/
├── admin
├── analyze
├── bnat
├── client
├── cloud
├── crawler
├── docx
├── dos
├── example.py
├── example.rb
├── fileformat
├── fuzzers
├── gather
├── parser
├── pdf
├── scanner
├── server
├── sniffer
├── spoof
├── sqli
├── voip
└── vsploit
```


#### Encoders :

Encoders will allow you to encode the exploit and payload in the hope that a signature-based antivirus solution may miss them.

Signature-based antivirus and security solutions have a database of known threats. They detect threats by comparing suspicious files to this database and raise an alert if there is a match. Thus encoders can have a limited success rate as antivirus solutions can perform additional checks.

```markup
encoders/
├── cmd
├── generic
├── mipsbe
├── mipsle
├── php
├── ppc
├── ruby
├── sparc
├── x64
└── x86
```


#### Evasion :

While encoders will encode the payload, they should not be considered a direct attempt to evade antivirus software. On the other hand, “evasion” modules will try that, with more or less success.

```markup
evasion/
└── windows
    ├── applocker_evasion_install_util.rb
    ├── applocker_evasion_msbuild.rb
    ├── applocker_evasion_presentationhost.rb
    ├── applocker_evasion_regasm_regsvcs.rb
    ├── applocker_evasion_workflow_compiler.rb
    ├── process_herpaderping.rb
    ├── syscall_inject.rb
    ├── windows_defender_exe.rb
    └── windows_defender_js_hta.rb
```


#### Exploits :

Exploits, neatly organized by target system.

```markup
exploits/
├── aix
├── android
├── apple_ios
├── bsd
├── bsdi
├── dialup
├── example_linux_priv_esc.rb
├── example.py
├── example.rb
├── example_webapp.rb
├── firefox
├── freebsd
├── hpux
├── irix
├── linux
├── mainframe
├── multi
├── netware
├── openbsd
├── osx
├── qnx
├── solaris
├── unix
└── windows
```


#### NOPs :

NOPs (No OPeration) do nothing, literally. They are represented in the Intel x86 CPU family with 0x90, following which the CPU will do nothing for one cycle. They are often used as a buffer to achieve consistent payload sizes.

```markup
nops/
├── aarch64
├── armle
├── cmd
├── mipsbe
├── php
├── ppc
├── sparc
├── tty
├── x64
└── x86
```


#### Payloads :

Payloads are codes that will run on the target system.

Exploits will leverage a vulnerability on the target system, but to achieve the desired result, we will need a payload. Examples could be; getting a shell, loading a malware or backdoor to the target system, running a command, or launching calc.exe as a proof of concept to add to the penetration test report. Starting the calculator on the target system remotely by launching the calc.exe application is a benign way to show that we can run commands on the target system.

Running command on the target system is already an important step but having an interactive connection that allows you to type commands that will be executed on the target system is better. Such an interactive command line is called a "shell". Metasploit offers the ability to send different payloads that can open shells on the target system.

```markup
payloads/
├── adapters
├── singles
├── stagers
└── stages
```

You will see four different directories under payloads: adapters, singles, stagers and stages.

- **Adapters:** An adapter wraps single payloads to convert them into different formats. For example, a normal single payload can be wrapped inside a PowerShell adapter, which will make a single PowerShell command that will execute the payload.  
- **Singles:** Self-contained payloads (add user, launch notepad.exe, etc.) that do not need to download an additional component to run.
- **Stagers:** Responsible for setting up a connection channel between Metasploit and the target system. Useful when working with staged payloads. “Staged payloads” will first upload a stager on the target system then download the rest of the payload (stage). This provides some advantages as the initial size of the payload will be relatively small compared to the full payload sent at once.
- **Stages:** Downloaded by the stager. This will allow you to use larger sized payloads.


#### Post :

Post modules will be useful on the final stage of the penetration testing process listed above, post-exploitation.

```terminal
post/
├── aix
├── android
├── apple_ios
├── bsd
├── firefox
├── hardware
├── linux
├── multi
├── networking
├── osx
├── solaris
└── windows
```


### Useful commands

##### ```use```

```bash
use <module>
```

##### ```show```

```baash
show <module>
```

This command lists available modules. For example, ```show payloads``` will lists all the available **payloads**.

##### ```back```

Syntax :

```bash
back
```

To leave the context.

##### ```info```

Syntax :

```bash
info
```

In a module context, use this command to get further information on. 

##### ```search```

Syntax :

```bash
search <query>
```

This command will search in the Metasploit database for modules relevant to the given search parameters. You can conduct searches using CVE numbers, exploit name ...

You can direct the search function using keywords such as type and platform :

```bash
search type:auxiliary telnet
```

##### Parameters :

Syntax :

```bash
set <NAME> <VALUE>
```

Once you have set a parameter, you can use the ```show options``` command to check the value was set correctly.

Useful parameters :

- **RHOSTS :** "Remote host", the IP address of the target system.
- **RPORT :** "Remote port", the port on the target system the vulnerable application is running on.
- **PAYLOAD :** The payload you will use with the exploit.
- **LHOST :** "Localhost", the attacking machine IP address.
- **LPORT :** "Local port", the port you will use for the reverse shell to connect back to.
- **SESSION :** Each connection established to the target system using Metasploit will have a session ID.

You can override any ```set``` parameter using the set command again with a different value. You can also clear any parameter value using the ```unset``` command or clear all set parameters with the ```unset all``` command.

##### ```setg```

Syntax :

```bash
setg <NAME> <VALUE>
```

Used to set value that will be used for all modules. You can also clear any value set with this command using ```unsetg```.

##### ```exploit```

Metasploit also supports the ```run``` command, which is an alias created for the ```exploit``` command as the word exploit did not make sens when using modules that were not exploits.

The ```exploit``` command can be used without any parameters or using the ```-z``` flag that will run the exploit and background the session as soon as it opens. 

To interact with any session, you can use the ```session -i``` command followed be the desired session number. 