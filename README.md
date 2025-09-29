# 🛜Capture and Analyze Network Traffic Using Wireshark.
Capture live network packets and identify basic protocols and traffic types and credentials or suspicious activity.

🎯 The goal of this task was to capture and analyze live network traffic in order to identify credentials or suspicious activity. Tools used include **Wireshark** and **tcpdump**.

---

## 🛠 Tools & Setup
- **Wireshark** – For analyzing PCAP files.  
- **tcpdump** – For capturing raw packets.
- **Kali Linux** – OS used for the exercise.  
- **Target Website** – `testphp.vulnweb.com` (an intentionally vulnerable demo site).

---

## 📂 Captured Files
- `http_traffic.pcap` → Packets captured during browsing and login attempts.  
- `traffic_tcpdump.pcap` → Packets captured using tcpdump.


---



## 🔎 Steps & Analysis

### 1. Login Page
<img width="1920" height="1051" alt="login" src="https://github.com/user-attachments/assets/445538b4-cde7-47d0-a174-08d8c170d793" />

The login form accepts **username and password** over **HTTP (not HTTPS)**.  
Captured traffic revealed that credentials were transmitted in **cleartext**.  

**Security Issue:** No encryption → credentials can be sniffed.  


### 2. Registration Page

<img width="1920" height="1051" alt="website" src="https://github.com/user-attachments/assets/4c8d30ef-aa77-41a6-941d-83d36bb0d98a" />


The registration page allows users to submit sensitive details such as:  
- Username  
- Password  
- Name  
- Credit card number  
- Email address  
- Phone number  
- Address  

All of this information was submitted **unencrypted via HTTP**.  



### 3. Registration Details Stored

<img width="1920" height="1051" alt="register" src="https://github.com/user-attachments/assets/774f20fb-c826-474a-8cc3-640aa1883b43" />
<img width="1920" height="1051" alt="register_details" src="https://github.com/user-attachments/assets/a8c16ce5-9cd1-41d7-b81e-856d406572d0" />

After submitting the registration form, the sensitive details were **echoed back in plain text**.  
This exposes **critical information leakage**:  
- Username: `yk47`  
- Password: `yash1234`  
- Email: `yashtest@gmail.com`  
- Credit Card: `1234 5678 8765 4321`


### 4. Website Structure

<img width="1920" height="1051" alt="register_details" src="https://github.com/user-attachments/assets/57671670-270d-40b3-95bd-02da15ffafd8" />

The vulnerable site provides multiple entry points for **attacks** such as SQL Injection, XSS, and credential interception.  


### 5. Capturing Packets with tcpdump
 
<img width="1920" height="1051" alt="tcpdump_net" src="https://github.com/user-attachments/assets/2cfdf11b-e300-40df-b992-7482c1d3d4d2" />

Command used:
```bash
tcpdump -w traffic_tcpdump.pcap -i eth0 -v 'tcp and net 192.168.0.0/24'

#Capture only Http traffic (port 80)
tcpdump -i eth0 -v port 80 and src 192.168.0.175 -w http_traffic.pcap
```
**Explanation of Each Part:**
- `tcpdump` → The packet capture tool you’re running.
- `-i eth0` → Capture traffic on the eth0 network interface (your main network adapter).
- `-v` → Use verbose mode to display extra information about each packet (like TTL, IP ID, etc.) if shown in the terminal.
- `port 80` → Capture only traffic going through TCP port 80 (standard HTTP traffic).
- `and src 192.168.0.175` → Apply an additional filter: only capture packets where the source IP address is 192.168.0.175.
- `-w http_traffic.pcap` → Instead of printing captured packets to the screen, write them to a file named http_traffic.pcap.
- This file can later be opened in Wireshark or other analysis tools.

<img width="1012" height="672" alt="pcap_file" src="https://github.com/user-attachments/assets/53bbbe96-8945-46d3-8a3e-e699dc4be0cb" />

We got our **`traffic_tcpdump.pcap file`** and **`http_traffic.pcap`**

### 6. Opening Captured Files in Wireshark

Open the Wireshark
<img width="1920" height="1051" alt="wireshark" src="https://github.com/user-attachments/assets/60b7cf4b-5899-411b-8b97-cc5c575e594f" />

From the **File → Open** menu in Wireshark, the captured `.pcap` file (`http_traffic.pcap`) was loaded for analysis.  
<img width="1915" height="1079" alt="open_file" src="https://github.com/user-attachments/assets/a2144676-57d7-4142-966d-69aa4e1eeb9d" />
<img width="1920" height="1080" alt="open_http_traffic" src="https://github.com/user-attachments/assets/de0940d9-f568-495a-b656-3b8b1486d92c" />


This allows us to view all HTTP requests/responses and inspect credentials being transmitted.


### 7. Inspecting Login Request

<img width="1921" height="1079" alt="login_data" src="https://github.com/user-attachments/assets/ef68a9f8-c456-450f-af57-6830f8cf221a" />


Filtering with `tcp.stream eq 1` revealed the login request.  

The **HTTP GET/POST packets** show credentials (**username, password**) transmitted in cleartext.  

**Example:**  
- Username: `yk47`  
- Password: `yash1234`  

⚠️ **Issue:** No encryption — attackers sniffing the network can easily retrieve these details.


### 8. Extracting User Credentials
To inspect sensitive data, we right-clicked on the `login.php` packet and selected:  
**Follow → HTTP Stream** 
<img width="1905" height="1041" alt="login_http_strem" src="https://github.com/user-attachments/assets/a2ab2807-1b87-4f7e-bce9-4f92d21d3a86" />

By following the HTTP Stream, we confirmed that login data was captured directly. 
<img width="1918" height="1076" alt="user_pass" src="https://github.com/user-attachments/assets/44f644f1-5dbf-441a-ac79-c4dc9a344dae" />

Credentials were submitted in the **POST body**:  
This confirms successful **credential harvesting via packet sniffing**.


### 9. Registration Data Leakage
<img width="1919" height="1076" alt="register_data" src="https://github.com/user-attachments/assets/1d33c991-6407-4aa4-86f4-240548e0caa3" />


The registration form exposed much more sensitive information:  

- **Full Name:** Yash Karnik  
- **Email:** yashtest@gmail.com  
- **Phone:** 1234567890  
- **Credit Card:** 1234 5678 8765 4321  

⚠️ **Critical Issue:** The system transmits highly sensitive **PII and financial data** in plain HTTP.



### 10. Captured Files Overview


This `.pcap` file was generated during the exercise:  


- `http_traffic.pcap` – captured via tcpdump 
- `http_traffic.pcap` – analyzed via **Wireshark**  



This file serves as evidence of the captured network activity and was later used for protocol inspection and credential extraction.

---

✅ With these new screenshots, the report now clearly shows the **step-by-step attack chain**:

1. Capturing network traffic  
2. Opening PCAPs in Wireshark  
3. Following HTTP streams  
4. Extracting usernames, passwords, and PII



