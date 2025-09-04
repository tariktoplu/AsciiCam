# Amaç

Web kamerandaki canlı görüntüyü terminalde ASCII karakterleriyle akış halinde göstermek. Bu rehber, **kendin yazarak** öğrenmen için küçük adımlara bölünmüş bir yol haritası, açıklamalar, ipuçları ve mini görevler içerir. En altta "başvurmalık" tam örnek kod da var (takılırsan bak).

---

## Yol Haritası

1. Ortamı kur ve doğrula
2. Proje iskeleti (g++ ve/veya CMake)
3. Kameradan kare yakala (Hello OpenCV)
4. Gri ton ve ön‑işleme
5. Terminal piksel oranı + yeniden boyutlandırma
6. Parlaklığı karaktere eşleme (ASCII paleti)
7. Terminale akıcı yazdırma (ANSI, çift tampon)
8. Klavye ile çıkış ve FPS kontrolü
9. (İsteğe bağlı) Renkli ASCII
10. (İsteğe bağlı) Dithering (hata yayılımı)
11. Performans ve sorun giderme
12. Alıştırmalar
13. Referans çözüm (spoiler)

---

## 0) Ön Koşullar ve Kurulum

**OpenCV** gereklidir.

### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install libopencv-dev build-essential
```

Doğrula:

```bash
pkg-config --modversion opencv4
```

### macOS (Homebrew)

```bash
brew install opencv
```

### Windows (vcpkg önerilir)

1. vcpkg kur: [https://github.com/microsoft/vcpkg](https://github.com/microsoft/vcpkg)
2. OpenCV ekle:

```powershell
vcpkg install opencv[contrib]:x64-windows
```

3. CMake projende vcpkg toolchain’i kullan.

> **Kamera test**: Laptop/USB kamera takılı ve başka uygulama tarafından kilitlenmemiş olmalı.

---

## 1) Proje İskeleti

```
ascii-cam/
 ├─ CMakeLists.txt     (CMake kullanacaksan)
 └─ src/
     └─ main.cpp
```

### CMakeLists.txt (minimal)

```cmake
cmake_minimum_required(VERSION 3.15)
project(ascii_cam CXX)
set(CMAKE_CXX_STANDARD 17)
find_package(OpenCV REQUIRED)
add_executable(ascii_cam src/main.cpp)
target_link_libraries(ascii_cam PRIVATE ${OpenCV_LIBS})
```

### g++ ile tek satır derleme (Linux/macOS)

```bash
g++ src/main.cpp -o ascii_cam `pkg-config --cflags --libs opencv4` -O2
```

---

## 2) Kameradan Kare Yakala (Hello OpenCV)

**Mini görev:** Aşağıdaki iskeleti yaz. "Kamera açıldı" yazısını gör ve bir kareyi pencere yerine terminale bilgi olarak bas.

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>
int main(){
    cv::VideoCapture cap(0); // 0: varsayılan kamera
    if(!cap.isOpened()) {
        std::cerr << "Kamera acilamadi!" << std::endl;
        return 1;
    }
    std::cout << "Kamera acildi" << std::endl;

    cv::Mat frame;
    cap >> frame; // tek kare
    std::cout << "Kare boyutu: " << frame.cols << "x" << frame.rows << "\n";
}
```

**Kontrol listesi:**

- Derleniyor mu?
- Kare boyutu mantıklı mı (ör. 640x480, 1280x720)?

---

## 3) Gri Ton ve Ön‑İşleme

**Neden?** ASCII eşlemesini tek kanal (parlaklık) üzerinden yapmak daha kolay ve hızlı.

```cpp
cv::Mat gray;
cv::cvtColor(frame, gray, cv::COLOR_BGR2GRAY);
```

> İleri seviye: Gamma düzeltme eklemek istersen `gray.convertTo` + normalize + `pow` ile yapabilirsin; insan gözü parlaklığı lineer algılamaz.

---

## 4) Terminal Karakter Oranı ve Yeniden Boyutlandırma

Terminalde karakter hücresinin _genişliği\:yüksekliği_ kare değildir. Yaklaşık **1:2** (genişlik daha dar) kabul ederek yükseklik tarafını ölçekle.

**Formül:**

```
newWidth  = hedef_sütun
newHeight = (orijinal_yükseklik * newWidth) / (char_aspect * orijinal_genişlik)
```

Burada `char_aspect ≈ 2.0`. Birçok fontta 1.8–2.2 arası deneyerek netliği artır.

```cpp
int colsOut = 120;            // terminal sütun sayın
double charAspect = 2.0;      // karakter yüksekliği/genişliği
int rowsOut = static_cast<int>(frame.rows * colsOut / (charAspect * frame.cols));
cv::resize(gray, gray, cv::Size(colsOut, rowsOut), 0, 0, cv::INTER_AREA);
```

---

## 5) Parlaklığı Karaktere Eşleme (ASCII Paleti)

Koyu→açık sırada bir palet seç:

```
"@%#*+=-:. "    // klasik
"$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/|()1{}[]?-_+~<>i!lI;:,\"^`'. "
```

**Eşleme:** `index = luma * (palette_len-1) / 255`

```cpp
static const std::string P = "@%#*+=-:. ";
inline char toAscii(unsigned char v){
    int i = (v * (int(P.size()) - 1)) / 255;
    return P[i];
}
```

**Mini görev:** Gri görüntüyü dolaş, her pikseli karaktere çevirip satır satır string’e yaz.

---

## 6) Terminale Akıcı Yazdırma (ANSI, Çift Tampon)

`system("clear")` yavaş ve titreşim yapar. Bunun yerine **ANSI kaçış kodları** kullan:

- Ekranı temizle: `"\x1b[2J"`
- İmleci (1,1)’e götür: `"\x1b[H"`
- Hepsini tek seferde yaz: **çift tampon** (std::string buffer)

```cpp
std::string buf; buf.reserve(gray.total()+gray.rows);
for(int r=0;r<gray.rows;++r){
    const uchar* row = gray.ptr<uchar>(r);
    for(int c=0;c<gray.cols;++c) buf.push_back(toAscii(row[c]));
    buf.push_back('\n');
}
std::cout << "\x1b[H" << buf; // sadece başa dön, tüm ekranı silmek istiyorsan once \x1b[2J
buf.clear();
```

**İpucu:** `std::ios::sync_with_stdio(false); std::cout.tie(nullptr);` çıkışı hızlandırır.

---

## 7) Klavye ile Çıkış ve FPS Kontrolü

OpenCV’nin `waitKey` pencereler için; terminalde Linux’ta **termios** ile non-blocking okuma yapabilirsin. Kolay yol: her döngüye küçük bir uyku koy (`std::this_thread::sleep_for`) ve **q** tuşu okunmuşsa çık.

**Basit FPS sınırlama:**

```cpp
using clock = std::chrono::steady_clock;
auto t0 = clock::now();
// ... kare işle
auto t1 = clock::now();
auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(t1 - t0);
int target = 33; // ~30 FPS
if(ms.count() < target) std::this_thread::sleep_for(std::chrono::milliseconds(target - ms.count()));
```

---

## 8) (İsteğe Bağlı) Renkli ASCII

ASCII karakterini basmadan önce 24‑bit ANSI ön plan rengi kullan:

```
"\x1b[38;2;R;G;Bm"  // R,G,B 0..255
```

Sonra karakteri yaz ve **sadece satır sonunda** `\x1b[0m` ile sıfırla (karakter başına sıfırlama pahalıdır). Renkli mod için orijinal çerçeveyi `cv::resize` ile küçült, her pikselin BGR’sini kullan.

**Not:** Renkli çıktı konsola çok yük bindirir; küçük çözünürlük seç.

---

## 9) (İsteğe Bağlı) Dithering (Floyd–Steinberg)

Siyah‑beyaz veya kısa palet kullanırken keskinliği artırır. Gri matristen çalış; hata terimini sağ ve alt komşulara dağıt:

```
* x+1,y   += 7/16 * err
* x-1,y+1 += 3/16 * err
* x,y+1   += 5/16 * err
* x+1,y+1 += 1/16 * err
```

---

## 10) Performans İpuçları

- Küçük çözünürlük (örn. 120×H) seç.
- `buf.reserve(...)` ile tek seferde string büyüt.
- Satır sonlarına kadar ANSI renk sıfırlama yapma.
- Karakter paletini **kısa tut** (10–20 karakter hızlıdır).
- Windows Konsolunda ANSI için _Virtual Terminal Processing_ aç (örnek kod aşağıda).

---

## 11) Sorun Giderme

- **Kamera açılamadı**: Cihaz `v4l2-ctl --list-devices` (Linux), başka uygulama kapat.
- **OpenCV bulunamadı**: `pkg-config --cflags --libs opencv4` çıktısını derlemeye ekle veya CMake `find_package(OpenCV REQUIRED)`.
- **Windows ANSI çalışmıyor**: Aşağıdaki kodu program başında çalıştır:

```cpp
#ifdef _WIN32
#include <windows.h>
void enableANSI(){
    HANDLE h = GetStdHandle(STD_OUTPUT_HANDLE);
    DWORD mode; GetConsoleMode(h, &mode);
    mode |= ENABLE_VIRTUAL_TERMINAL_PROCESSING;
    SetConsoleMode(h, mode);
}
#endif
```

---

## 12) Alıştırmalar (Kendin Yazarak)

1. **Siyah‑beyaz ASCII**: 0–255 eşiğine göre iki karakter ("#" ve boşluk) kullan.
2. **Palet kalibrasyonu**: Farklı paletler dene, hangisi daha net?
3. **Karakter oranı**: `charAspect` değerini 1.8–2.2 arası tarayıp en net sonucu bul.
4. **FPS sayacı**: Sol üst köşeye (ilk satıra) “FPS: xx.x” yazdır.
5. **Renkli mod**: 24‑bit ANSI ile renklendir, satır sonlarında `\x1b[0m` kullan.
6. **Dithering**: Floyd–Steinberg uygula ve farkı kıyasla.
7. **Kaydetme**: Her N karede bir, ASCII buffer’ı `.txt` dosyasına yaz.

---

## 13) Referans Çözüm (Spoiler)

> Takıldıysan bak. Parça parça inşa ettiğin kodla karşılaştır.

```cpp
// src/main.cpp
#include <opencv2/opencv.hpp>
#include <iostream>
#include <string>
#include <thread>
#include <chrono>

#ifdef _WIN32
#include <windows.h>
void enableANSI(){
    HANDLE h = GetStdHandle(STD_OUTPUT_HANDLE);
    DWORD mode; GetConsoleMode(h, &mode);
    mode |= ENABLE_VIRTUAL_TERMINAL_PROCESSING;
    SetConsoleMode(h, mode);
}
#endif

static const std::string PALETTE = "@%#*+=-:. ";
inline char toAscii(unsigned char v){
    int i = (v * (int(PALETTE.size()) - 1)) / 255;
    return PALETTE[i];
}

int main(){
#ifdef _WIN32
    enableANSI();
#endif
    std::ios::sync_with_stdio(false);
    std::cout.tie(nullptr);

    cv::VideoCapture cap(0);
    if(!cap.isOpened()){ std::cerr << "Kamera acilamadi!\n"; return 1; }

    const int colsOut = 120;
    const double charAspect = 2.0; // karakterin yukseklik/genislik orani

    using clock = std::chrono::steady_clock;
    while(true){
        auto t0 = clock::now();
        cv::Mat frame, gray;
        cap >> frame; if(frame.empty()) break;

        cv::cvtColor(frame, gray, cv::COLOR_BGR2GRAY);
        int rowsOut = int(frame.rows * colsOut / (charAspect * frame.cols));
        cv::resize(gray, gray, cv::Size(colsOut, rowsOut), 0,0, cv::INTER_AREA);

        std::string buf; buf.reserve(gray.total()+gray.rows+8);
        buf.append("\x1b[H"); // imleci basa al
        for(int r=0;r<gray.rows;++r){
            const uchar* row = gray.ptr<uchar>(r);
            for(int c=0;c<gray.cols;++c) buf.push_back(toAscii(row[c]));
            buf.push_back('\n');
        }
        std::cout << buf; // tek seferde bas

        // FPS sinirlama ~30
        auto dt = std::chrono::duration_cast<std::chrono::milliseconds>(clock::now()-t0).count();
        int target = 33; if(dt < target) std::this_thread::sleep_for(std::chrono::milliseconds(target-dt));
        // Cikmak icin Ctrl+C
    }
}
```

---

## Sonraki Adımlar

- Renkli ASCII s
