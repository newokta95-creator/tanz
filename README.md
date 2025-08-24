index.html
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>FRI PORTOFOLIO</title>
  <style>
    style.css * {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Segoe UI', sans-serif;
}

body, html {
  height: 100%;
  overflow-x: hidden;
  color: white;
}

#bg-video {
  position: fixed;
  top: 0;
  left: 0;
  min-width: 100vw;
  min-height: 100vh;
  object-fit: cover;
  z-index: -1;
}

.profile-container {
  position: fixed;
  top: 20px;
  left: 20px;
  z-index: 10;
}
.profile-pic {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: 2px solid #fff;
  cursor: pointer;
  transition: transform 0.3s;
}
.profile-pic:hover {
  transform: scale(1.1);
}
.profile-menu {
  margin-top: 10px;
  background: rgba(0, 0, 0, 0.6);
  padding: 12px;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.profile-menu a {
  color: white;
  text-decoration: none;
  padding: 8px;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.1);
}
.profile-menu a:hover {
  background: rgba(255, 255, 255, 0.3);
}
.hidden { display: none; }

.sidebar-toggle {
  position: fixed;
  top: 20px;
  right: 20px;
  font-size: 28px;
  color: white;
  cursor: pointer;
  z-index: 10;
}
.sidebar {
  display: none;
  position: fixed;
  top: 0;
  right: 0;
  height: 100%;
  width: 220px;
  background: rgba(0, 0, 0, 0.7);
  padding: 30px 20px;
  flex-direction: column;
  gap: 20px;
}
.sidebar.show {
  display: flex !important;
}
.sidebar ul {
  list-style: none;
}
.sidebar li {
  color: white;
  padding: 10px;
  border-bottom: 1px solid rgba(255,255,255,0.2);
}
.sidebar li:hover {
  background: rgba(255,255,255,0.1);
}
  </style>
</head>
<body>

  <!-------------AUTOPLAYVIDEO-------------->
  <video autoplay muted loop id="bg-video">
    <source src="media/bg.mp4" type="video/mp4" />
  </video>

    <!-------------AUTOPLAYMUSIC------------->
  <audio autoplay loop id="bg-audio">
    <source src="media/music.mp3" type="audio/mpeg" />
  </audio>

  <!-------------PROFIL-------------->
  <div class="profile-container">
    <img src="iklan1.jpeg" alt="Profil" class="profile-pic" onclick="toggleProfileMenu()" />
    <div id="profile-menu" class="hidden profile-menu">
      <a href="https://tiktok.com" target="_blank">TikTok</a>
      <a href="https://youtube.com" target="_blank">YouTube</a>
      <a href="https://instagram.com" target="_blank">Instagram</a>
    </div>
  </div>

  <!-------------SIDEBAR-------------->
  <div class="sidebar-toggle" onclick="toggleSidebar()">≡</div>
  <div id="sidebar" class="sidebar">
    <ul>
      <li>Home</li>
      <li>Tentang Saya</li>
      <li>Kontak</li>
    </ul>
  </div>

  <script>
    function toggleProfileMenu() {
      document.getElementById("profile-menu").classList.toggle("hidden");
    }
    function toggleSidebar() {
      document.getElementById("sidebar").classList.toggle("show");
    }
    window.addEventListener("click", function (e) {
      if (!e.target.closest(".profile-container")) {
        document.getElementById("profile-menu").classList.add("hidden");
      }
      if (!e.target.closest(".sidebar") && !e.target.closest(".sidebar-toggle")) {
        document.getElementById("sidebar").classList.remove("show");
      }
    });
  </script>

</body>
</html>
bekron html suprot vidio an audio aoutou peliay dan autou loop
oy ni gua udah fix sapa tau butuh yang bukan sharp ni ffmpeg const axios = require('axios');
const fs = require('fs');
const { spawn } = require('child_process');
const path = require('path');
const FormData = require('form-data');

module.exports = async (sock, msg, args, reply) => {
  const from = msg.key.remoteJid;
  const sender = msg.key.fromMe ? sock.user.id : msg.key.participant || from;
  const pushName = msg.pushName || 'Pengguna';
  const quoted = msg.quoted;
  const text = quoted?.text || args.join(" ");

  if (!text) return reply('Masukkan teks atau balas pesan dengan caption *.qc*');

    // ═══════════[ UPLOAD TO CATBOX]════════════ \\
  const uploadToCatbox = async (buffer, filename = 'avatar.jpg') => {
    try {
      const form = new FormData();
      form.append('reqtype', 'fileupload');
      form.append('fileToUpload', buffer, filename);
      const res = await axios.post('https://catbox.moe/user/api.php', form, {
        headers: form.getHeaders(),
      });
      if (typeof res.data === 'string' && res.data.startsWith('https://')) {
        return res.data.trim();
      }
      throw new Error('Upload gagal: ' + res.data);
    } catch (e) {
      console.error('[Catbox Error]', e.message);
      return null;
    }
  };

    // ═══════════[GET PROFILE ]═══════════ \\
  let profileUrl = 'https://files.catbox.moe/xmeefj.jpg';
  try {
    const pp = await sock.profilePictureUrl(sender, 'image');
    const res = await axios.get(pp, { responseType: 'arraybuffer' });
    const uploaded = await uploadToCatbox(res.data);
    if (uploaded) profileUrl = uploaded;
  } catch {
    console.warn('gagal ambil atau upload foto profil, pakai default.');
  }

  try {
    const apiUrl = `https://api.ownblox.my.id/api/qc?text=${encodeURIComponent(text)}&name=${encodeURIComponent(pushName)}&profile=${encodeURIComponent(profileUrl)}&sender=${encodeURIComponent(sender.split('@')[0])}`;
    const imageRes = await axios.get(apiUrl, { responseType: 'arraybuffer' });

    // ═══════════[ TEMP ]════════════ \\
    const temp = path.join(__dirname, '../temp');
    if (!fs.existsSync(temp)) fs.mkdirSync(temp);

    const inputPath = path.join(temp, `qc_${Date.now()}.png`);
    const outputPath = inputPath.replace('.png', '.webp');
    fs.writeFileSync(inputPath, imageRes.data);

    
    await new Promise((resolve, reject) => {
      const ffmpeg = spawn('ffmpeg', [
        '-i', inputPath,
        '-vf', 'scale=500:500:force_original_aspect_ratio=decrease,pad=512:512:(ow-iw)/2:(oh-ih)/2:color=0x00000000',
        '-vcodec', 'libwebp',
        '-lossless', '1',
        '-qscale', '75',
        '-preset', 'picture',
        '-an',
        '-y',
        outputPath
      ]);
      ffmpeg.on('close', code => (code === 0 ? resolve() : reject(`FFmpeg exited with code ${code}`)));
      ffmpeg.on('error', reject);
    });

    const buffer = fs.readFileSync(outputPath);
    await sock.sendMessage(from, { sticker: buffer }, { quoted: msg });

  
    fs.unlinkSync(inputPath);
    fs.unlinkSync(outputPath);

  } catch (e) {
    console.error('QC Error:', e);
    reply('Gagal membuat stiker QC.');
  }
};
playvid fitur make dependisi ytdl-core sama yt search

const fs = require('fs');
const { exec } = require('child_process');
const delay = (ms) => new Promise(res => setTimeout(res, ms));

const { sendThumbReply, reactHasil } = require('../helpers/sendSysMsg');

module.exports = async (sock, msg, args, reply) => {
  const from = msg.key.remoteJid;
  const text = args.join(" ");
  const thumbBot = 'https://files.catbox.moe/0z9zwv.jpg';

  if (!text) {
    return await sendThumbReply(sock, from, msg, {
      text: 'Masukkan judul video!\nContoh: .playvid lathi',
      title: 'Contoh Play Video',
      body: 'Mori Bot',
      thumb: thumbBot
    });
  }

  const output = `./tmp/playvid_${Date.now()}.mp4`;

  const clockEmojis = ["🕛", "🕘", "🕕"];
  for (const emoji of clockEmojis) {
    await sock.sendMessage(from, {
      react: { text: emoji, key: msg.key }
    });
    await delay(500);
  }

  const command = `yt-dlp -f mp4 -o "${output}" "ytsearch1:${text}"`;
  exec(command, async (err, stdout, stderr) => {
    if (err || !fs.existsSync(output)) {
      console.error(stderr || err);
      await reactHasil(sock, from, msg.key, false);
      return await sendThumbReply(sock, from, msg, {
        text: ' Gagal mengunduh video.',
        title: 'PlayVid Error',
        body: ' Bot',
        thumb: thumbBot
      });
    }

    await sock.sendMessage(from, {
      video: fs.readFileSync(output),
      mimetype: 'video/mp4',
      fileName: 'video.mp4',
      caption: `✅ Hasil pencarian: ${text}`,
      contextInfo: {
        externalAdReply: {
          title: 'Video berhasil diunduh',
          body: 'Bot',
          thumbnailUrl: thumbBot,
          mediaType: 1,
          renderLargerThumbnail: false
        }
      }
    }, { quoted: msg });

    await reactHasil(sock, from, msg.key, true);
    fs.unlinkSync(output);
  });
};
