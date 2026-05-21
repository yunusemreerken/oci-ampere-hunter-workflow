# 🦾 OCI Ampere Hunter — GitHub Actions Workflow
<details open>

<summary> 🇬🇧 English </summary>


A GitHub Actions workflow that automatically hunts for a **4 OCPU / 24 GB RAM Ampere A1** instance on Oracle Cloud Infrastructure by bypassing the infamous **“Out of Host Capacity”** error.

> **Base project:** [hitrov/oci-arm-host-capacity](https://github.com/hitrov/oci-arm-host-capacity)  
> This repository adapts the original PHP script to run **fully automated on GitHub Actions**. No VPS, cron job, or local server required.

---

## How It Works

OCI periodically adds new compute capacity. The script continuously checks for available resources by:

1. Calling the OCI `LaunchInstance` API
2. Checking the current instance count (`OCI_MAX_INSTANCES`)
3. Launching the instance immediately if capacity becomes available
4. Retrying automatically every **5–20 minutes** using GitHub Actions cron jobs

When successful, the workflow outputs the instance details as JSON in the Actions logs.

---

# Setup

## 1. Fork This Repository

Click the **Fork** button in the top-right corner of the repository page.

---

## 2. Create an OCI API Key

Go to:

[OCI Console](https://cloud.oracle.com/) → Profile → User Settings → API Keys → **Add API Key**

- Select **Generate API Key Pair**
- Download the private key as a `.pem` file
- Save the generated configuration values:
  - `region`
  - `user`
  - `tenancy`
  - `fingerprint`

---

## 3. Make the Private Key Accessible

You have two options:

### A) OCI Object Storage (Recommended)

- Upload the `.pem` file to a bucket
- Create a **Pre-Authenticated Request**
- Copy the generated URL

### B) Any Public URL

Host the `.pem` file on any publicly accessible server.

---

## 4. Configure GitHub Secrets

In your forked repository:

`Settings → Secrets and variables → Actions → New repository secret`

Add the following secrets one by one:

| Secret Name | Description |
|---|---|
| `OCI_REGION` | Example: `eu-frankfurt-1` |
| `OCI_USER_ID` | OCI User OCID |
| `OCI_TENANCY_ID` | OCI Tenancy OCID |
| `OCI_KEY_FINGERPRINT` | API key fingerprint |
| `OCI_PRIVATE_KEY_FILENAME` | Public URL to the `.pem` file |
| `OCI_SUBNET_ID` | Target subnet OCID |
| `OCI_IMAGE_ID` | OS image OCID |
| `OCI_SSH_PUBLIC_KEY` | Contents of `~/.ssh/id_rsa.pub` (single line) |
| `OCI_OCPUS` | OCPU count (default: `4`) |
| `OCI_MEMORY_IN_GBS` | RAM amount in GB (default: `24`) |
| `OCI_MAX_INSTANCES` | Maximum instance count (default: `1`) |

> **Tip:** To find `OCI_SUBNET_ID` and `OCI_IMAGE_ID`, open the OCI instance creation page and inspect the `/instances` request payload in your browser DevTools → Network tab.

---

## 5. Enable the Workflow

Go to the **Actions** tab in GitHub:

- Enable workflows
- Wait for the scheduled run
- Or manually trigger it using **Run workflow**

---

# How to Find Subnet ID and Image ID

1. Open OCI Console → Compute → Instances → **Create Instance**
2. Select shape: `VM.Standard.A1.Flex`
3. Under Networking, check:
   - **Do not assign a public IPv4 address**
4. Press `F12` → Open the **Network** tab
5. Click **Create**
6. Find the failed `/instances` request
7. Right-click → **Copy as cURL**
8. Extract:
   - `subnetId`
   - `imageId`

from the copied request payload.

---

# About `OCI_AVAILABILITY_DOMAIN`

- **ARM (A1.Flex):**  
  Can be launched in any availability domain within your home region.  
  You may leave `OCI_AVAILABILITY_DOMAIN` empty and let the script try all domains automatically.

- **AMD (E2.1.Micro - Always Free):**  
  You must select an availability domain marked as **Always Free Eligible**, for example:

```text
FeVO:EU-FRANKFURT-1-AD-2
```

---

# GitHub Actions ToS Warning ⚠️

This workflow runs continuously. According to the GitHub Actions Terms of Service:

> Actions must not be used for activities unrelated to the development, testing, deployment, or publication of software projects.

Official policy:  
https://docs.github.com/en/github/site-policy/github-terms-for-additional-products-and-features#actions

After your instance is successfully created, remove the workflow immediately:

```bash
git rm .github/workflows/ampere-hunter.yml
git commit -m "Instance created, remove workflow"
git push origin main
```

---

# After the Instance Is Created

## Assign a Public IP

OCI Console → Instance Details → Attached VNICs → IPv4 Addresses → **Edit** → Select **Ephemeral** → Update

---

## Connect via SSH

Using a public IP:

```bash
ssh -i ~/.ssh/id_rsa opc@<PUBLIC_IP>
```

Using a private IP:

```bash
ssh -i ~/.ssh/id_rsa opc@10.x.x.x
```

---

# Possible Outputs

## Capacity Full

```json
{
  "code": "InternalError",
  "message": "Out of host capacity."
}
```

---

## Resource Limit Exceeded

```json
{
  "code": "LimitExceeded",
  "message": "The following service limits were exceeded: standard-a1-memory-count, standard-a1-core-count."
}
```

---

## Success

The workflow logs will contain the newly created instance details in JSON format.

---

# Credits

- Original script and concept: [hitrov/oci-arm-host-capacity](https://github.com/hitrov/oci-arm-host-capacity)
- OCI API signing implementation: [hitrov/oci-api-php-request-sign](https://github.com/hitrov/oci-api-php-request-sign)

---

# License

MIT


</details>



# 🦾 OCI Ampere Hunter — GitHub Actions Workflow
<details open>

<summary>🇹🇷 Turkish</summary>


Oracle Cloud Infrastructure'da **"Out of Host Capacity"** hatasını atlayarak **4 OCPU / 24 GB RAM Ampere A1** instance'ı otomatik olarak avlayan GitHub Actions workflow'u.

> **Base proje:** [hitrov/oci-arm-host-capacity](https://github.com/hitrov/oci-arm-host-capacity)  
> Bu repo, orijinal PHP script'ini **GitHub Actions üzerinde tamamen otomatik** çalışacak şekilde yapılandırır. Sunucu kurulumu veya cron gerektirmez.

---

## Nasıl Çalışır?

OCI, zaman zaman kapasitesine yeni kaynaklar ekler. Script bu anı yakalamak için:

1. OCI `LaunchInstance` API'sini çağırır
2. Mevcut instance sayısını kontrol eder (`OCI_MAX_INSTANCES`)
3. Kapasite müsaitse instance'ı başlatır; değilse sessizce bekler
4. GitHub Actions cron job'u sayesinde her **5–20 dakikada bir** otomatik çalışır

Başarı durumunda instance oluşturulur ve workflow log'larına JSON çıktısı yazılır.

---

## Kurulum

### 1. Bu repoyu fork'la

Sağ üstteki **Fork** butonuna tıkla.

### 2. OCI API Key oluştur

[OCI Console](https://cloud.oracle.com/) → Profil → User Settings → API Keys → **Add API Key**

- "Generate API Key Pair" seçili olsun
- Private key'i `.pem` dosyası olarak indir
- Çıkan konfigürasyon metnini not al (`region`, `user`, `tenancy`, `fingerprint`)

### 3. Private key'i erişilebilir yap

İki seçenek:

**A) OCI Object Storage (önerilen):**  
Bucket → Nesneyi yükle → Pre-Authenticated Request oluştur → URL'yi kopyala

**B) Herhangi bir public URL:**  
`.pem` dosyasını erişilebilir bir web sunucusuna yükle

### 4. GitHub Secrets'ları ayarla

Forkladığın repoda: **Settings → Secrets and variables → Actions → New repository secret**

Aşağıdaki secret'ları tek tek ekle:

| Secret Adı | Açıklama |
|---|---|
| `OCI_REGION` | Örn: `eu-frankfurt-1` |
| `OCI_USER_ID` | OCI kullanıcı OCID |
| `OCI_TENANCY_ID` | OCI tenancy OCID |
| `OCI_KEY_FINGERPRINT` | API key parmak izi |
| `OCI_PRIVATE_KEY_FILENAME` | `.pem` dosyasının public URL'si |
| `OCI_SUBNET_ID` | Hedef subnet OCID |
| `OCI_IMAGE_ID` | Kullanılacak OS imajının OCID |
| `OCI_SSH_PUBLIC_KEY` | `~/.ssh/id_rsa.pub` içeriği (satır sonu olmadan) |
| `OCI_OCPUS` | OCPU sayısı (varsayılan: `4`) |
| `OCI_MEMORY_IN_GBS` | RAM miktarı GB (varsayılan: `24`) |
| `OCI_MAX_INSTANCES` | Maksimum instance sayısı (varsayılan: `1`) |

> **İpucu:** `OCI_SUBNET_ID` ve `OCI_IMAGE_ID` için OCI Console'da instance oluşturma ekranını aç, tarayıcı DevTools → Network sekmesinde `/instances` isteğinin `data-binary` parametresine bak.

### 5. Workflow'u etkinleştir

**Actions** sekmesine git → Workflow'u etkinleştir → İlk çalıştırmayı bekle veya **Run workflow** ile manuel tetikle.

---

## Subnet ve Image ID Nasıl Bulunur?

1. OCI Console → Compute → Instances → **Create Instance**
2. Shape olarak `VM.Standard.A1.Flex` seç
3. Networking → "Do not assign a public IPv4 address" işaretle
4. Tarayıcıda **F12** → Network sekmesi → **Create**'e tıkla
5. Kırmızıyla gelen `/instances` isteğine sağ tıkla → **Copy as cURL**
6. Kopyaladığın metinden `subnetId` ve `imageId` değerlerini bul

---

## OCI_AVAILABILITY_DOMAIN Hakkında

- **ARM (A1.Flex):** Home region içindeki herhangi bir availability domain'de oluşturulabilir. `OCI_AVAILABILITY_DOMAIN` boş bırakılabilir; script tüm domain'leri dener.
- **AMD (E2.1.Micro - Always Free):** `Always Free Eligible` etiketli domain seçilmeli, örn: `FeVO:EU-FRANKFURT-1-AD-2`

---

## GitHub Actions ToS Uyarısı ⚠️

Bu workflow **sürekli çalışır**. GitHub Actions [Kullanım Koşulları](https://docs.github.com/en/github/site-policy/github-terms-for-additional-products-and-features#actions) şunu belirtir:

> Actions, yazılım projenizin üretimi, test edilmesi, dağıtımı veya yayınlanmasıyla ilgisiz etkinlikler için kullanılmamalıdır.

**Instance'ın oluşturulduğunu gördükten hemen sonra** workflow dosyasını sil:

```bash
git rm .github/workflows/ampere-hunter.yml
git commit -m "Instance oluşturuldu, workflow kaldırıldı"
git push origin main
```

---

## Instance Oluştuktan Sonra

### Public IP atama

OCI Console → Instance Details → Attached VNICs → IPv4 Addresses → **Edit** → Ephemeral seç → Update

### SSH ile bağlanma

```bash
ssh -i ~/.ssh/id_rsa opc@<PUBLIC_IP>
```

Public IP yoksa aynı VNIC içinden private IP ile:

```bash
ssh -i ~/.ssh/id_rsa opc@10.x.x.x
```

---

## Olası Çıktılar

**Kapasite doluysa:**
```json
{
  "code": "InternalError",
  "message": "Out of host capacity."
}
```

**Limit aşıldıysa (instance zaten var):**
```json
{
  "code": "LimitExceeded",
  "message": "The following service limits were exceeded: standard-a1-memory-count, standard-a1-core-count."
}
```

**Başarılıysa:** Instance bilgilerini içeren JSON çıktısı log'a yazılır.

---

## Teşekkür

- Orijinal script ve fikir: [hitrov/oci-arm-host-capacity](https://github.com/hitrov/oci-arm-host-capacity)
- OCI API imzalama: [hitrov/oci-api-php-request-sign](https://github.com/hitrov/oci-api-php-request-sign)

---

## Lisans

MIT

</details>
