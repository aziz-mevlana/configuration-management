# Pomodoro App & Cloud Infrastructure Automation

Bu proje, modern bir **React + Vite** frontend uygulamasının (Pomodoro Timer) ve bu uygulamanın bulut sunucuya (AWS/DigitalOcean) dağıtımını tamamen otomatize eden **Ansible Infrastructure as Code (IaC)** altyapısının birleşimidir.

Uygulama, bulut sunucusu üzerinde **Node.js v22 (LTS)** ortamında dinamik olarak derlenmekte ve **Nginx** web sunucusu arkasında yüksek performansla servis edilmektedir.

---

## Proje Mimarisi & Özellikler

- **Frontend:** React, Vite, Tailwind CSS (Seans yönetimi, dinamik arka plan renkleri ve sesli uyarılar).
- **Altyapı Otomasyonu (IaC):** Ansible kullanılarak sunucu konfigürasyonu tamamen kodla (Idempotent) yönetilir.
- **Modern Node.js Ortamı:** Vite gereksinimlerini karşılamak adına sunucuya resmi NodeSource deposundan **Node.js v22 (LTS)** kurulur.
- **Web Sunucu:** Nginx entegrasyonu, temiz web root yönetimi (/var/www/html).
- **Güvenlik & Sıkılaştırma:** SSH anahtar yönetimi ve `fail2ban` entegrasyonu (Opsiyonel/Base rolü).

---

## Klasör Yapısı

Ansible rolleri modern en iyi pratiklere (Best Practices) uygun olarak modüler bir yapıda tasarlanmıştır:

```text
.
├── inventory.ini               # Sunucu IP ve SSH bilgilerinin yer aldığı envanter
├── setup.yml                   # Ana Playbook (Rolleri tetikleyen orkestratör)
└── roles/
    ├── base/                   # Sistem güncellemeleri ve temel araçlar (curl, git, fail2ban)
    ├── nginx/                  # Nginx kurulumu ve servis yönetimi
    ├── app/                    # Node.js v22 kurulumu, Git clone, NPM build ve deployment
    └── ssh/                    # Güvenli SSH anahtarlarının yapılandırılması
```
## Kurulum ve Canlıya Alma (Deployment)

### 1. Ön Gereksinimler
Yerel makinenizde (Mac/Linux) Ansible'ın kurulu olduğundan ve hedef bulut sunucusuna SSH erişiminizin bulunduğundan emin olun.

### 2. Envanter Dosyasının Düzenlenmesi
inventory.ini dosyasını kendi sunucu bilgilerinize göre güncelleyin:

[sunucular]
sunucum_bulut ansible_host=YOUR_SERVER_IP ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa

### 3. Otomasyonu Tetikleme
Tüm altyapıyı tek bir komutla ayağa kaldırmak ve uygulamayı canlıya almak için:

ansible-playbook setup.yml -i inventory.ini

Eğer sistem güncellemelerini atlayıp doğrudan sadece web sunucusu ve uygulama katmanını (Vite build & Nginx) çalıştırmak isterseniz **Tag** mimarisini kullanabilirsiniz:

ansible-playbook setup.yml -i inventory.ini --tags "nginx,app"

---

## Derinlemesine Bakış: Uygulama Dağıtım Adımları (app Rolü)

Ansible app rolü arka planda şu kritik DevOps süreçlerini yönetir:
1. **NodeSource Entegrasyonu:** Ubuntu depolarındaki eski Node sürümleri yerine resmi NodeSource script'i ile Node.js v22 kurulur.
2. **Güvenli Derleme:** Kaynak kodlar /tmp/pomodoro-src dizinine çekilir. Sunucu kirliliğinin önüne geçilir.
3. **Bağımlılık & Derleme Yönetimi:** npm install ve npm run build komutları otomatik koşturularak Vite'ın statik dist çıktıları üretilir.
4. **Sıfır Kesinti (Zero-Downtime) Dağıtımı:** Eski Nginx dosyaları temizlenir ve yeni statik dosyalar /var/www/html/ dizinine güvenli bir şekilde kopyalanır.

---

## Geliştirici

- **Aziz Mevlana Alim** - [GitHub](https://github.com/aziz-mevlana)
