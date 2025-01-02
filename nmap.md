# Nmap Kullanımı ve Port Taramaları

## 1. **Temel Tarama**
```bash
nmap 10.10.137.90
```
Bu komut, hedefte açık olan temel TCP portlarını kontrol eder.

---

## 2. **Servis ve Versiyon Bilgisi**
```bash
nmap -sV 10.10.137.90
```
Bu komut, açık portlarda hangi servislerin çalıştığını ve servislerin versiyon bilgilerini gösterir.

---

## 3. **Detaylı Tarama (Aggressive Scan)**
```bash
nmap -A 10.10.137.90
```
Bu tarama, servis versiyon bilgilerini, işletim sistemi tespiti ve traceroute gibi ek bilgileri verir.

---

## 4. **Belirli Portları Hedefleme**
Sadece belirli portları taramak istersen:
```bash
nmap -p 21,22,80,443 10.10.137.90
```
Bu komut, belirttiğin portları (ör. FTP, SSH, HTTP, HTTPS) tarar.

---

## 5. **UDP Port Tarama**
UDP portlarını kontrol etmek için:
```bash
nmap -sU 10.10.137.90
```
Bu komut UDP portları üzerinde hangi servislerin çalıştığını kontrol eder.

---

## 6. **Tüm Portları Tarama**
Tüm portları taramak için:
```bash
nmap -p- 10.10.137.90
```
Bu komut, 1-65535 arasındaki tüm portları tarar.

---

## 7. **Script Tarama (NSE Kullanarak)**
Nmap'in script motoruyla daha spesifik testler yapabilirsin. Örneğin, HTTP servisinde zafiyet taraması yapmak için:
```bash
nmap --script http-vuln* 10.10.137.90
```

---

Elde edilen sonuçlara göre bulgular üzerinden özelleştirilmiş komutlarla devam edebilirsin.
