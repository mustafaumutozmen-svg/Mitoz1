<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mitoz Görev Tamamlama - Basit Öğretici Oyun</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #e8f5e9;
            color: #2e7d32;
            text-align: center;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .container {
            width: 90%;
            max-width: 800px;
            background-color: #ffffff;
            border-radius: 15px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
            padding: 30px;
            margin-top: 20px;
        }

        h1 {
            color: #1b5e20;
            margin-bottom: 20px;
            font-size: 1.8em;
        }

        #puan-gosterge {
            font-size: 1.5em;
            font-weight: bold;
            color: #ff9800;
            margin-bottom: 20px;
        }

        #gorev-kutusu {
            background-color: #a5d6a7;
            padding: 25px;
            border-radius: 10px;
            margin-bottom: 30px;
            min-height: 100px;
            display: flex;
            flex-direction: column;
            justify-content: center;
        }

        #gorev-metni {
            font-size: 1.4em;
            font-weight: bold;
            color: #004d40;
        }

        #secenekler-kutusu {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .secenek-btn {
            background-color: #4caf50;
            color: white;
            padding: 15px;
            border: none;
            border-radius: 8px;
            font-size: 1.1em;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.1s;
        }

        .secenek-btn:hover {
            background-color: #388e3c;
            transform: translateY(-2px);
        }

        .secenek-btn.dogru {
            background-color: #00c853; /* Yeşil */
        }

        .secenek-btn.yanlis {
            background-color: #d32f2f; /* Kırmızı */
            pointer-events: none;
        }

        #mesaj-kutusu {
            margin-top: 20px;
            font-size: 1.3em;
            font-weight: bold;
            min-height: 30px;
        }

        #devam-btn {
            background-color: #ff9800;
            color: white;
            padding: 12px 25px;
            border: none;
            border-radius: 8px;
            font-size: 1.1em;
            cursor: pointer;
            margin-top: 20px;
            display: none;
        }

        #oyun-sonu {
            font-size: 1.8em;
            color: #1b5e20;
            display: none;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Mitoz Bölünme: Hücrenin Görev Defteri</h1>
    <div id="puan-gosterge">Puan: 0 / 6</div>

    <div id="gorev-kutusu">
        <p id="gorev-metni">Yükleniyor...</p>
    </div>

    <div id="secenekler-kutusu">
        </div>

    <div id="mesaj-kutusu"></div>
    
    <button id="devam-btn">Sıradaki Görev</button>
    
    <div id="oyun-sonu"></div>
</div>

<script>
    // Yeni müfredata uygun, açıklayıcı evre isimleri ve görevleri
    const gorevler = [
        { id: 'i', ad: 'Hazırlık ve Çoğalma Evresi', metin: 'DNA’yı ve organelleri kopyalayıp hücre hacmini büyütme görevini tamamlıyorum.' },
        { id: 'p', ad: 'Yoğunlaşma ve Kaybolma Evresi', metin: 'Kromatin iplikleri kısa ve kalın kromozomlara dönüştürüyor, çekirdek zarını yok ediyorum.' },
        { id: 'm', ad: 'Dizilme Evresi (Orta Hat)', metin: 'Tüm kromozomları hücrenin tam ortasında tek sıra halinde hizalama görevini yapıyorum.' },
        { id: 'a', ad: 'Ayrılma ve Çekilme Evresi', metin: 'Kardeş kromatitleri birbirinden ayırıp zıt kutuplara çekme görevimi yerine getiriyorum.' },
        { id: 't', ad: 'Yeniden Yapılanma Evresi', metin: 'Kutuplarda yeni çekirdek zarlarını ve çekirdekçiği oluşturuyor, kromozomları tekrar iplik haline getiriyorum.' },
        { id: 's', ad: 'Sitoplazma Bölünmesi', metin: 'Hücreyi ortadan ikiye bölerek genetik olarak aynı iki yavru hücreyi oluşturma görevimi bitiriyorum.' }
    ];

    let mevcutGorevIndex = 0;
    let puan = 0;
    const secenekSayisi = 3;
    let karisikGorevler = [...gorevler]; // Görevleri karıştırmak için kopya al

    const gorevMetni = document.getElementById('gorev-metni');
    const seceneklerKutusu = document.getElementById('secenekler-kutusu');
    const mesajKutusu = document.getElementById('mesaj-kutusu');
    const devamBtn = document.getElementById('devam-btn');
    const puanGosterge = document.getElementById('puan-gosterge');
    const oyunSonuDiv = document.getElementById('oyun-sonu');
    const container = document.querySelector('.container');

    // Görevleri rastgele sırala
    function gorevleriKaristir() {
        karisikGorevler.sort(() => Math.random() - 0.5);
    }

    // Seçenekleri oluşturmak için rastgele yanlış cevaplar seç
    function secenekleriHazirla(dogruCevap) {
        const tumSecenekler = gorevler.filter(g => g.ad !== dogruCevap.ad).map(g => g.ad);
        const karisikYanlislar = tumSecenekler.sort(() => Math.random() - 0.5);

        // Doğru cevap ve 2 rastgele yanlış cevap
        let secenekler = [dogruCevap.ad, ...karisikYanlislar.slice(0, secenekSayisi - 1)];
        // Seçenekleri kendi aralarında da karıştır
        return secenekler.sort(() => Math.random() - 0.5);
    }

    function goreviYukle() {
        if (mevcutGorevIndex >= karisikGorevler.length) {
            oyunuBitir();
            return;
        }

        const guncelGorev = karisikGorevler[mevcutGorevIndex];
        gorevMetni.textContent = guncelGorev.metin;
        mesajKutusu.textContent = '';
        devamBtn.style.display = 'none';
        seceneklerKutusu.innerHTML = '';

        const secenekler = secenekleriHazirla(guncelGorev);

        secenekler.forEach(secenekAd => {
            const btn = document.createElement('button');
            btn.className = 'secenek-btn';
            btn.textContent = secenekAd;
            btn.addEventListener('click', () => cevabiKontrolEt(btn, secenekAd, guncelGorev.ad));
            seceneklerKutusu.appendChild(btn);
        });
    }

    function cevabiKontrolEt(secilenBtn, secilenAd, dogruAd) {
        // Tüm butonları devre dışı bırak
        const tumButonlar = document.querySelectorAll('.secenek-btn');
        tumButonlar.forEach(btn => {
            btn.style.pointerEvents = 'none';
            if (btn.textContent === dogruAd) {
                btn.classList.add('dogru');
            }
        });

        if (secilenAd === dogruAd) {
            puan++;
            secilenBtn.classList.add('dogru');
            mesajKutusu.innerHTML = `<span style="color: #00c853;">✅ Görev Tamamlandı!</span>`;
        } else {
            secilenBtn.classList.add('yanlis');
            mesajKutusu.innerHTML = `<span style="color: #d32f2f;">❌ Hata! Doğru görev: ${dogruAd}</span>`;
        }

        puanGosterge.textContent = `Puan: ${puan} / ${gorevler.length}`;
        devamBtn.style.display = 'block';
    }

    function oyunuBitir() {
        container.style.display = 'none';
        oyunSonuDiv.style.display = 'block';
        oyunSonuDiv.innerHTML = `
            🎉 TEBRİKLER! TÜM GÖREVLER BİTTİ! 🎉<br>
            Nihai Puanın: ${puan} / ${gorevler.length}
        `;
    }

    // Devam butonuna tıklama olayını ekle
    devamBtn.addEventListener('click', () => {
        mevcutGorevIndex++;
        goreviYukle();
    });

    // Oyunu Başlat
    gorevleriKaristir();
    goreviYukle();
</script>

</body>
</html>

