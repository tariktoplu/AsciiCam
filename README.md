# 🎥 AsciiCam

> **Kameranızdan canlı görüntüyü terminalde ASCII karakterleriyle görün**

[![C++](https://img.shields.io/badge/C%2B%2B-17-blue)](https://en.cppreference.com/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green)](https://opencv.org/)
[![CMake](https://img.shields.io/badge/CMake-3.15+-brightgreen)](https://cmake.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📋 İçindekiler

- [Nedir?](#nedir)
- [Özellikler](#özellikler)
- [Hızlı Başlangıç](#hızlı-başlangıç)
- [Kurulum](#kurulum)
- [Kullanım](#kullanım)
- [Proje Yapısı](#proje-yapısı)
- [Katkıda Bulunma](#katkıda-bulunma)

---

## 🎯 Nedir?

**AsciiCam**, bilgisayarınızın web kamerası veya bağlı kameradan canlı görüntüyü yakaladıktan sonra bunu ASCII karakterlerine dönüştürerek terminalde akış halinde gösterir.

### Nasıl Çalışır?

```
Kamera Girdisi
    ↓
Gri Tona Dönüştürme
    ↓
Parlaklığı Karaktere Eşleme
    ↓
Terminal Çıktısı (Akışlı)
```

---

## ✨ Özellikler

- 🎬 **Canlı Kamera Akışı** - OpenCV ile gerçek zamanlı görüntü yakalama
- 📝 **ASCII Dönüştürme** - Piksel parlaklığını karaktere eşleme
- ⚡ **Performans Odaklı** - Düşük CPU kullanımı, yüksek FPS
- 🖥️ **ANSI Renk Desteği** - Renkli ASCII çıkışı (isteğe bağlı)
- ⌨️ **Klavye Kontrolü** - Q tuşu ile çıkış, FPS kontrollü oynatma
- 🔧 **CMake Build** - Platform bağımsız derleme

---

## 🚀 Hızlı Başlangıç

### Gereksinimler

- **C++17** uyumlu derleyici (g++, clang, MSVC)
- **OpenCV 4.x**
- **CMake 3.15+**
- **Web Kamerası**

### Linux/macOS

```bash
# Repoyu klonla
git clone https://github.com/tariktoplu/AsciiCam.git
cd AsciiCam

# Derleme klasörü oluştur
mkdir build && cd build

# Derle
cmake ..
make

# Çalıştır
./Ascii
```

### Windows (MSVC + vcpkg)

```bash
# vcpkg ile OpenCV kur
vcpkg install opencv:x64-windows

# CMake konfigure et (vcpkg toolchain ile)
cmake .. -DCMAKE_TOOLCHAIN_FILE=[vcpkg-root]/scripts/buildsystems/vcpkg.cmake

# Visual Studio'da derle
cmake --build . --config Release
```

---

## 💻 Kurulum

### Step 1: OpenCV Yükleme

#### Ubuntu/Debian:

```bash
sudo apt update
sudo apt install libopencv-dev build-essential cmake
```

#### macOS (Homebrew):

```bash
brew install opencv cmake
```

Kurulumun başarılı olduğunu kontrol et:

```bash
pkg-config --modversion opencv4
```

### Step 2: Projeyi Derleme

```bash
cd /path/to/AsciiCam
mkdir -p build
cd build
cmake ..
make -j$(nproc)
```

---

## 🎮 Kullanım

### Basit Başlangıç

```bash
./Ascii
```

Kameraız başında veya yazması dini tuşu açılır. Kamera ön kamerası otomatik olarak seçilir.

### Klavye Kontrolleri

| Tuş       | İşlem                 |
| --------- | --------------------- |
| **Q**     | Programı kapat        |
| **Space** | Oynat / Duraklat      |
| **R**     | Ekran oranını sıfırla |

### Seçenekler (İleride)

```bash
# Özel genişlik ayarla
./Ascii --width 150

# Renkli çıktı
./Ascii --color

# FPS sınırla
./Ascii --fps 30
```

---

## 📁 Proje Yapısı

```
AsciiCam/
├── README.md                    # Bu dosya
├── CMakeLists.txt               # CMake konfigürasyonu
├── CMakePresets.json            # CMake preset'leri
├── gerekenler.md                # Detaylı yol haritası
├── main.cpp                     # Ana uygulama kodu
└── build/                       # Derleme çıktıları
    └── Ascii                    # Çalıştırılabilir dosya
```

---

## 🏗️ Mimarı

### Kod Akışı

```cpp
int main() {
    // 1. Kamera aç
    cv::VideoCapture camera(0);

    // 2. Kare oku
    cv::Mat frame;
    camera >> frame;

    // 3. Gri tona dönüştür
    cv::cvtColor(frame, gray, cv::COLOR_BGR2GRAY);

    // 4. ASCII'ye dönüştür
    for (her piksel) {
        piksel_parlaklığını_karaktere_eşle();
    }

    // 5. Terminale yazdır
    std::cout << ascii_buffer << std::endl;
}
```

### Performans İpuçları

- **Çözünürlüğü azalt** - Terminal genişliği kadar boyutlandır (~80-150 piksel)
- **İki buffering kullan** - Titreme azaltmak için
- **ANSI escape kodları** - Hızlı temizleme için `\033[2J` kullan
- **Sadeleştirilmiş ASCII paleti** - Parlaklık seviyelerine uygun karakterler seç

---

## 🧪 Derleme Sorunları

### OpenCV Bulunamıyor

```bash
# CMakeLists.txt'de değiştir:
find_package(OpenCV REQUIRED)

# Eğer hala çalışmazsa, manuel olarak ver:
cmake .. -DOpenCV_DIR=/usr/local/lib/cmake/opencv4
```

### Kamera Açılamıyor

```bash
# Linux'ta izinleri kontrol et
ls -la /dev/video*

# Gerekirse izin ver
sudo usermod -a -G video $USER
```

---

## 🤝 Katkıda Bulunma

Katkılar daima hoştur!

### Nasıl?

1. Fork et
2. Feature branch oluştur (`git checkout -b feature/NewFeature`)
3. Değişiklikleri commit et (`git commit -m 'Add NewFeature'`)
4. Push et (`git push origin feature/NewFeature`)
5. Pull Request aç

### İdea'lar

- [ ] Renk desteği (24-bit ANSI)
- [ ] Dithering algoritması (Floyd-Steinberg)
- [ ] CLI argümanları
- [ ] Çoklu kamera desteği
- [ ] Video dosyası desteği
- [ ] Snapshot alma
- [ ] GIF kaydı

---

## 💬 İletişim

- 📧 Email: tarikttoplu@gmail.com
- 🐙 GitHub: [@tariktoplu](https://github.com/tariktoplu)

---

```
Made with ❤️ for terminal enthusiasts
```
