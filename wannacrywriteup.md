## 1. WannaCry
SHA256 hash for sample: 24d004a104d4d54034dbcffc2a4b19a11f39008a575aa614ea04703480b1022c (obtained from MalwareBazaar)
### Entry Point

The entry point of the program can be found under **Functions** in Ghidra under the **Symbol Tree**. The entry point is defined as the `entry` function.



### Primary Functions

* **`entry`**: Begins the program.

* **`FUN_00408140` (`WinMain`)**: Windows entry point of the program.

* **`InternetOpenUrlA`**: Function that reaches out to the URL/domain of the C2 server to determine whether infection should begin.

* **`FUN_00408090` (`wannacry_real_entry`)**: Function that begins the real entry execution of the WannaCry malware.

* **`FUN_00407c40` (`no_argument_handler`)**: If no arguments are provided, creates a WannaCry service disguised as *Microsoft Security Service 2.0*.

* **`FUN_00407ce0` (`write_1831_to_taskche.exe`)**: Writes the `1831` payload to `taskche.exe` (the program that executes the ransomware).



### Program Flow

1. Starts with `entry`.

2. Program executes `FUN_00408140` to reach out to a URL string via `InternetOpenUrlA` to decide whether to infect the machine.

   * If the URL returns anything other than `0` (kill switch triggered), the malware closes connections and terminates execution.

   * Otherwise, it calls `FUN_00408090` to begin the malware infection process.

3. Within `FUN_00408090`:

   * **No Arguments:** Executes `FUN_00407f20`, which calls `FUN_00407c40` to create and start the WannaCry service disguised as *Microsoft Security Center 2.0 Service*, and executes `FUN_00407ce0` to write the `1831` payload to `taskche.exe`.

   * **With Arguments:** Executes `OpenServiceA` to run `s_mssecsvc2.0_004312fc` as a process. When started with `-m security` arguments, it begins exploiting **EternalBlue** and **DoublePulsar** vulnerabilities on connected computers via its `ServiceMain` function to propagate across networks.

4. Within `taskche.exe`:

   * Retrieves local computer name and obfuscates it.

   * If started with the `/i` argument, copies itself to `C:\ProgramData\<obfuscatedPCname>\taskche.exe`. It then attempts to start `taskche.exe` from that directory location as a service (or as a process if service creation fails).

   * Checks for the global mutex `MsWinZonesCacheCounterMutex`. If present, `taskche.exe` exits, indicating another instance of WannaCry is already active.

   * Extracts an embedded `.zip` file using password `WNcry@ol7` into its residing directory.

   * Loads Bitcoin wallet addresses `[INSERT HERE]`.

   * Reads `c.wnry`, which contains TOR zip info, configuration data, and `.onion` addresses:

     * `gx7ekbenv2riucmf.onion`

     * `57g7spgrzlojinas.onion`

     * `xxlvbrloxvriy2c5.onion`

     * `76jdd2ir2embyv47.onion`

     * `cwwnhwhlz52maqm7.onion`

   * Alters attributes of extracted files/folders by hiding directory contents and granting full access permissions (`icacls ./grant Everyone:F /T /C /Q` and `attrib +h`).

   * Retrieves the cryptographic provider: *Microsoft Enhanced RSA and AES Cryptographic Provider*.

   * Reads and verifies `t.winry`, then reads an encrypted AES key decrypted via a hardcoded RSA public key in `taskche.exe`. The decrypted AES key decrypts a DLL in memory, which is copied to the heap.

5. The `TaskStart` function exported by the DLL executes the following actions:

   * Retrieves the public key used for file encryption.

   * Creates global mutex `MsWinZonesCacheCounterMutex`.

   * Sets Registry key: `HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Run\<obfuscatedPCName> = <tasksche.exeFilePath>`

   * Encrypts files on the system and renames extensions to `.WNCRY` (with code branches also accounting for `.WNCRYT` and `.WNCYR`).

   * Starts secondary processes:

     * `taskse.exe`: Enumerates active RDP sessions and executes `taskche.exe` in each session (worm capabilities).

     * `@WanaDecryptor@.exe` (or `u.wnry`): Displays timer and payment GUI (the ransomware screen).

     * `taskdl.exe`: Deletes temporary files created during setup.

     * `r.wnry`: Contains ransom message.

     * `s.winr`: Contains TOR executable.

     * `msg/`: Directory containing multi-language ransomware notes.



### Important Strings

* **C2 URL:** `https://www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com`

* **Service Name:** `mssecsvc.exe` (Microsoft Security Service 2.0)

* **Executable:** `taskche.exe`

* **Mutex:** `Global\MsWinZonesCacheCounterMutex`

* **Attribute Modification Command:** `icacls ./grant Everyone:F /T /C /Q attrib +h`



### Behavior

TO DO: Add diagram here



### Exploited Vulnerabilities

* EternalBlue

* DoublePulsar



### Spread Method

Spreads using unpatched Server Message Block (**SMBv1**) ports over network connections, allowing automated propagation without user interaction.



### Exploitation Method

The **EternalBlue** exploit sends malicious packets to target machines via Port 445 (SMB). Target machines misinterpret these packets, causing a buffer overflow memory corruption error that grants attackers arbitrary code execution capabilities with SYSTEM/administrative privileges. In infected machines, background scanners sweep local networks and internet addresses to infect accessible target endpoints.



### Encryption Method

Hybrid encryption using **RSA-2048** and **AES-128**.
