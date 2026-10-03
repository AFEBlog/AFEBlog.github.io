---
layout: post
title: "Zyxel VMG Seri Modemlerde Donanım ve Yazılım Analizi: UART, Config İnceleme ve SSH Erişimi"
date: 2026-10-03 10:46:00 +0300
tags:
  - zyxel
  - reverse-engineering
  - uart
  - embedded-security
  - networking
math: true
toc: true
---

Gömülü cihazlar (embedded systems) ve ev ağı donanımları, ağ güvenliği araştırmalarında her zaman ilgi çekici bir yere sahiptir. Bu yazıda, Zyxel'in **VMG33XX** ve **VMG1312** serisi modemleri üzerinde gerçekleştirdiğim donanımsal ve yazılımsal analiz sürecini, konfigürasyon dosyası manipülasyonlarını ve eski Dropbear SSH sunucularına modern sistemlerden nasıl güvenli bir şekilde erişilebileceğini adım adım inceleyeceğiz.

---

## 1. Zyxel VMG33XX Serisi: Konfigürasyon Ayıklama

İlk aşamada Zyxel VMG33XX serisi bir cihazın yedeklenmiş konfigürasyon dosyasını inceledim. Bu işlem için açık kaynaklı [zyxel-modem-backup-extract](https://github.com/iscilyas/zyxel-modem-backup-extract) aracından yararlandım. 

Aracı konfigürasyon dosyası üzerinde çalıştırdığımızda, cihaz üzerindeki kullanıcı hesapları, PPP servis sağlayıcı bilgileri ve Wi-Fi kimlik doğrulama anahtarları açık metin olarak elde edilebiliyor:

```bash
$ python ./zyxel-extract.py configuration-backupsettings.conf
Users configured on router:
Username: root		Password: [REDACTED_PASS] 	[DISABLED]
Username: admin		Password: [REDACTED_PASS]

PPP configuration:
Username: user@isp_example		Password: [REDACTED_PPP_PASS]

Wifi info:
SSID: 'My Wifi'    Authentication: psk psk2     Password: '[REDACTED_WIFI_PASS]'
```

Bu yöntem eski seri cihazlarda hızlı bir bilgi toplama imkanı sağlasa da, daha yeni veya farklı mimariye sahip modemlerde konfigürasyon yapısı değişiklik gösterebilir.

---

## 2. Zyxel VMG1312 Serisi: UART İle Donanımsal Shell Erişimi

VMG1312 serisi cihazda ise yazılımsal sınırları aşmak adına modemi sökerek **UART (Universal Asynchronous Receiver-Transmitter)** seri iletişim hatlarına erişim sağladım.

### Donanımsal Aşama ve Pin Tespiti
1. Modem kasası açılarak devre kartı (PCB) üzerindeki seri haberleşme bacakları incelendi.
2. Multimetre yardımıyla voltaj seviyeleri ve GND (Toprak) hattı tespit edildi.
3. Deneme-yanılma ve GND referansı ile **TX**, **RX** ve **VCC** pin dizilimi doğrulandı.
4. USB-TTL dönüştürücü aracılığıyla seri konsol bağlantısı kuruldu.

UART bağlantısı sağlandıktan sonra seri arayüz üzerinden `admin` kullanıcısı ile sisteme giriş yapıldı:
- **Kullanıcı:** `admin` (`$` root yetkisiz shell)
- **Kullanıcı:** `root` (`#` tam yetkili root shell)

Daha sonra dosya sistemini inceledim ve cihazın ham konfigürasyon dosyası indirdim.

![UART](../Medya/003)

> [!Note]
> UART baud hızı`115200`.

![UART ve SSH ekran alıntısı](../Medya/001)

---
## 3. RAW JSON Config Yapısı ve Şifrelenmiş Key'ler

VMG1312 serisinden elde edilen yedek dosyası önceki serinin aksine **JSON** formatındaydı. Dosya içeriğinde dinamik DNS ve sistem kullanıcı yapılandırmalarını barındıran bölümler tespit edildi:

```json
  "X_ZYXEL_EXT": {
    "DynamicDNS": {
      "Enable": false,
      "ServiceProvider": "",
      "DDNSType": "",
      "HostName": "",
      "UserName": "",
      "Password": "",
      "IPAddressPolicy": 0,
      "UserIPAddress": "0.0.0.0",
      "Wildcard": false,
      "Offline": false
    }
  }
```

Aynı JSON dosyası içinde kullanıcı hesaplarının parola dizilimleri (`Password` ve `DefaultPassword`) şifrelenmiş biçimde (`_encrypt_...`) yer alıyordu:

```json
  "X_ZYXEL_LoginCfg": {
    "LoginGroupNumberOfEntries": 3,
    "LoginGroupConfigurable": true,
    "LogGp": [
      {
        "Account": [
          {
            "AutoShowQuickStart": false,
            "Enabled": true,
            "EnableQuickStart": true,
            "Page": "",
            "Username": "root",
            "PasswordHash": "",
            "Privilege": "login",
            "Password": "_encrypt_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
            "DefaultPassword": "_encrypt_YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY",
            "AccountCreateTime": 0,
            "AccountRetryTime": 0,
            "AccountIdleTime": 600,
            "AccountLockTime": 900,
            "RemoHostAddress": "",
            "DotChangeDefPwd": false,
            "AutoGenPwdBySn": false
          }, 
          {
            "AutoShowQuickStart": false,
            "Enabled": true,
            "EnableQuickStart": true,
            "Page": "",
            "Username": "supervisor",
            "PasswordHash": "",
            "Privilege": "login,httpd",
            "Password": "_encrypt_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
            "DefaultPassword": "_encrypt_YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY",
            "AccountCreateTime": 0,
            "AccountRetryTime": 0,
            "AccountIdleTime": 600,
            "AccountLockTime": 900,
            "RemoHostAddress": "",
            "DotChangeDefPwd": false,
            "AutoGenPwdBySn": false
          }
        ]
      },
      {
        "Account": [
          {
            "AutoShowQuickStart": true,
            "Enabled": true,
            "EnableQuickStart": true,
            "Page": "Broadband,Wireless,Home_Networking,QoS,NAT,Routing,DNS,IGMP_MLD,Vlan_Group,Interface_Grouping,USB_Service,Firewall,MAC_Filter,Parental_Control,Scheduler_Rule,Certificates,Log,Traffic_Status,Routing_Table,McastSt,ARP_Table,ARPTable_handle,SNMP,System,User_Account,Remote_MGMT,Time,Log_Setting,Backup/Restore,Backup_Restore,Reboot,Diagnostic,Status,Upnp_Portmap,Diagnostic_Result,xDSL_Statistics,xDSLStatistics_handle,NATSession_handle,RoutingTable_handle,McastSt,wps_status_handle,PortMirror,ParseDirectory,ParseUSBInfo,Email_Notify,Firmware_Upgrade,Diagnostic_id,ROMD",
            "Username": "admin",
            "PasswordHash": "",
            "Privilege": "login,httpd",
            "AccountIdleTime": 300,
            "Password": "_encrypt_ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ",
            "DefaultPassword": "_encrypt_ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ",
            "AccountCreateTime": 0,
            "AccountRetryTime": 0,
            "AccountLockTime": 900,
            "RemoHostAddress": "",
            "DotChangeDefPwd": true,
            "AutoGenPwdBySn": false
          }
        ]
      }
    ]
  }
```

---

## 4. Config Manipülasyonu ile Şifre Çözme (DDNS Injection)

Elde edilen şifreli anahtarları çözebilmek adına web arayüzünün şifre çözme (decryption) mekanizmasından yararlanan bir enjeksiyon yöntemi uyguladım.

Elde ettiğimiz şifreli metni JSON dosyasındaki `X_ZYXEL_EXT` -> `DynamicDNS` bölümüne enjekte ediyoruz:

```json
  "X_ZYXEL_EXT": {
    "DynamicDNS": {
      "Enable": false,
      "ServiceProvider": "userdefined",
      "DDNSType": "",
      "HostName": "foobar",
      "UserName": "foobar",
      "Password": "_encrypt_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      "IPAddressPolicy": 0,
      "UserIPAddress": "0.0.0.0",
      "Wildcard": false,
      "Offline": false
    }
  }
```

> [!Important]
> `"Enable": false` olarak bırakıldığından emin olunmalıdır. Aksi takdirde modem açılışta DDNS servisini başlatmaya çalışırken kilitlenebilir ve `192.168.1.1` web arayüzüne erişim tamamen kesilebilir.

Ben en başta `"Enable": true` yaptım ve web arayüzüne erişemedim, ve `Reset` tuşunu kullanarak sıfırladım, bende bu yeni şifreyle tekrar yama uyguladım.
### Adım Adım Uygulama ve DOM Trick:
1. Düzenlenen JSON dosyası farklı bir isimle kaydettim (Orijinal yedeğinizi az önceki gibi durumlar amacıyla saklamayı/yedeklemeyi unutmayın!).
2. İlk denemede `Enable: true` yapıtığım için web arayüzü kilitlendi. Cihaz arkasındaki Reset butonundan sıfırladım (Reset sonrası `root` gibi dinamik parolalar yenilense de `admin` statik parolaları sabit kaldı).
3. Web arayüzüne girerek **Bakım > Yedekle & Geri Yükle** kısmından modifiye edilmiş yedeği yükledim.
4. Reboot tamamlandıktan sonra **Ağ Ayarı - DNS - Dinamik DNS** sekmesine gittim.
5. Şifre alanındaki Gizle/Göster (Göz) butonu bulunmuyordu. Geliştirici Araçları (F12) ile basit bir düzenleme yaparak şifreyi elde ettim:
   
   *Mevcut durum:*
   ```html
   <input type="password" name="ddnsPassword" maxlength="32" id="yui_3_8_0_2_1791013062268_656">
   ```
   
   *Değiştirilen durum:*
   ```html
   <input type="text" name="ddnsPassword" maxlength="32" id="yui_3_8_0_2_1791013062268_656">
   ```

Eleman türü `text` yapıldığında, cihazın arka planda çözmüş olduğu açık metin (decrypted) parola ekranda görünür hale geldi ve not edildi.

![Ekran Görüntüsü](../Medya/002)

*(Dipnot: Bu pratik web enjeksiyon mantığı [DonanımHaber Forumu'ndaki ilgili paylaşımdan](https://forum.donanimhaber.com/mesaj/yonlen/149377529) esinlenilerek uygulanmıştır.)*

---

## 5. Ağ Servisleri ve Port İncelemesi

UART root shell erişimi aktifken cihaz üzerinde arka planda çalışan servisler ve dinlenen portları analiz ettim.

### Binary Tespiti
```bash
# which dropbear
/usr/sbin/dropbear

# which sshd
# which telnetd
/usr/sbin/telnetd

# which busybox
/bin/busybox
```
Aramada standart OpenSSH (`sshd`) bulunamadı. SSH sunucusunun hafif sıklet **Dropbear** üzerinden çalıştığı doğrulandı.

### Çalışan Süreçler (Process List)
```bash
# ps | grep -E 'dropbear|sshd|telnet'
 2129 root      1968 S    /usr/sbin/telnetd -p 23
 2159 root      1252 S    dropbear -p 22 -P /var/run/dropbear.pid
 2597 root      1968 S    grep -E 'dropbear|sshd|telnet'
```

### Dinlenen Portlar (Netstat & Kernel Sockets)
`netstat -lntp` çıktısı ve `/proc/net/tcp` (hex: `:0016` -> 22, `:0017` -> 23) doğrulaması sonucunda cihazın tüm ağ arayüzlerinde dinlediği servisler haritalandı:

```text
PC (192.168.1.9)
   │
   │ Wi-Fi / Ethernet
   ▼
Zyxel Modem (192.168.1.1)
   ├── TCP 21  → pure-ftpd
   ├── TCP 22  → Dropbear SSH
   ├── TCP 23  → Telnet
   ├── TCP 53  → dnsmasq
   ├── TCP 80  → zhttpd (Web GUI)
   ├── TCP 139 → Samba (smbd)
   ├── TCP 443 → zhttpd (HTTPS)
   └── TCP 445 → Samba (smbd)
```

Windows PowerShellüzerinden bağlantı testi doğrulandı:
```powershell
Test-NetConnection 192.168.1.1 -Port 22
# TcpTestSucceeded : True
```

---

## 6. Modern SSH İstemcileri İle Uyumsuzluk Sorunları ve Çözümü

Modern işletim sistemlerinde (Windows OpenSSH / OpenSSH 8.8+) eski ve zayıf kabul edilen kriptografik algoritmalar varsayılan olarak devre dışı bırakılmıştır. Bu durum, eski Dropbear sürümlerine bağlanırken aşamalı sıkılaşma hatalarına (handshake failure) yol açar.

> [!Important]
> geri kalan kısmı okumak istemeyenler `ssh -o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o MACs=+hmac-sha1 root@192.168.1.1` komutunu kullanabilir.
### Karşılaşılan Aşamalar ve Hatalar

#### Aşama 1: KEX (Key Exchange) Hatası
```bash
$ ssh root@192.168.1.1
Unable to negotiate with 192.168.1.1 port 22: no matching key exchange method found.
Their offer: diffie-hellman-group1-sha1,diffie-hellman-group14-sha1
```
*Çözüm parametresi:* `-o KexAlgorithms=+diffie-hellman-group14-sha1`

#### Aşama 2: Host Key Hatası
```bash
Unable to negotiate with 192.168.1.1 port 22: no matching host key type found.
Their offer: ssh-rsa
```
*Çözüm parametresi:* `-o HostKeyAlgorithms=+ssh-rsa`

#### Aşama 3: MAC (Message Authentication Code) Hatası
```bash
Unable to negotiate with 192.168.1.1 port 22: no matching MAC found.
Their offer: hmac-sha1-96,hmac-sha1,hmac-md5
```
*Çözüm parametresi:* `-o MACs=+hmac-sha1`

### Başarılı Bağlantı Komutu

Tüm protokol müzakerelerini kapsayan nihayi SSH bağlantı komutu:

```bash
ssh -o KexAlgorithms=+diffie-hellman-group14-sha1 \
    -o HostKeyAlgorithms=+ssh-rsa \
    -o MACs=+hmac-sha1 \
    root@192.168.1.1
```

Bu parametreler sağlandığında SSH el sıkışması tamamlanır ve parola sorma aşamasına (`root@192.168.1.1's password:`) sorunsuz bir şekilde geçilir.

---

## Sonuç

Bu araştırmada gömülü cihazların donanımsal arayüzlerinden (UART) başlayıp web katmanındaki şifreleme mantıklarına ve ağ servislerinin yapılandırmasına kadar geniş bir yelpazeyi inceledik. Modern sistemlerin güvenlik standartları yükseldikçe, eski ağ cihazlarına erişimde kriptografik uyumluluk parametrelerinin ayarlanması kritik bir rol oynamaktadır.

- [x] UART pin tespiti ve shell erişimi
- [x] Raw JSON konfigürasyon analizi
- [x] Web arayüzü enjeksiyonu ile parola deşifresi
- [x] Servis ve port analizi
- [x] Legacy SSH bağlantı optimizasyonu