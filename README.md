# Stock-prediction-webapp
<!DOCTYPE html>
<html>
<head>
  <title>📲 Multi-Task JS Dashboard</title>
</head>
<body>
  <h1>📷 Multi-Task JS App</h1>

  <!-- 1. Capture Photo -->
  <h2>1. Capture Photo</h2>
  <video id="video" width="320" height="240" autoplay></video>
  <button onclick="takePhoto()">📸 Take Photo</button>
  <canvas id="canvas" width="320" height="240" style="display:none;"></canvas>
  <br><img id="photo" src="">

  <!-- 2. Send Email -->
  <h2>2. Send Email (via backend)</h2>
  <button onclick="sendEmail()">📧 Send Email</button>

  <!-- 3. Send Captured Photo via Email -->
  <h2>3. Send Captured Photo via Email</h2>
  <button onclick="sendPhotoByEmail()">📤 Send Photo Email</button>

  <!-- 4. Record Video -->
  <h2>4. Record Video</h2>
  <button onclick="startRecording()">🎥 Start Recording</button>
  <button onclick="stopRecording()">🛑 Stop & Send</button>
  <video id="recorded" controls></video>

  <!-- 5. Send WhatsApp -->
  <h2>5. Send WhatsApp</h2>
  <input id="whatsappNumber" placeholder="Phone number with country code"/>
  <input id="whatsappMessage" placeholder="Your message"/>
  <button onclick="sendWhatsApp()">📨 Send WhatsApp</button>

  <!-- 6. Send SMS (needs backend API like Twilio) -->
  <h2>6. Send SMS</h2>
  <button onclick="alert('Use Twilio or Fast2SMS API to send SMS via backend')">📲 Send SMS</button>

  <!-- 7. Get Geolocation -->
  <h2>7. Geolocation</h2>
  <button onclick="getLocation()">📍 Get Location</button>
  <div id="location"></div>

  <!-- 8. Open Google Map -->
  <h2>8. Open Google Maps with Live Location</h2>
  <button onclick="openMap()">🌍 Open Google Maps</button>

  <!-- 9. Show Last Email Info (placeholder) -->
  <h2>9. Show Last Email Info</h2>
  <button onclick="getLastEmail()">📩 Get Last Email</button>
  <p id="emailInfo">No email fetched yet.</p>

  <!-- 10. Post on Instagram -->
  <h2>10. Post on Instagram</h2>
  <button onclick="alert('Instagram posting requires backend automation via Instabot or API')">📤 Post to Instagram</button>

  <script>
    navigator.mediaDevices.getUserMedia({ video: true })
      .then(stream => { document.getElementById('video').srcObject = stream; });

    function takePhoto() {
      let canvas = document.getElementById('canvas');
      let video = document.getElementById('video');
      canvas.getContext('2d').drawImage(video, 0, 0, canvas.width, canvas.height);
      let dataUrl = canvas.toDataURL('image/png');
      document.getElementById('photo').src = dataUrl;
      localStorage.setItem('capturedPhoto', dataUrl);
    }

    function sendEmail() {
      fetch('/send-email', { method: 'POST' })
        .then(res => alert('Email sent (mock)!'))
        .catch(err => alert('Error sending email.'));
    }

    function sendPhotoByEmail() {
      let data = localStorage.getItem('capturedPhoto');
      fetch('/send-photo-email', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ image: data })
      }).then(res => alert('Photo sent via email!'));
    }

    let mediaRecorder, recordedBlobs = [];
    async function startRecording() {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      recordedBlobs = [];
      mediaRecorder = new MediaRecorder(stream);
      mediaRecorder.ondataavailable = e => recordedBlobs.push(e.data);
      mediaRecorder.onstop = () => {
        const blob = new Blob(recordedBlobs, { type: 'video/webm' });
        const url = URL.createObjectURL(blob);
        document.getElementById('recorded').src = url;

        let reader = new FileReader();
        reader.onloadend = () => {
          fetch('/send-video-email', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ video: reader.result })
          }).then(() => alert('Video sent!'));
        };
        reader.readAsDataURL(blob);
      };
      mediaRecorder.start();
      alert('Recording...');
    }

    function stopRecording() {
      mediaRecorder.stop();
    }

    function sendWhatsApp() {
      let phone = document.getElementById('whatsappNumber').value;
      let msg = document.getElementById('whatsappMessage').value;
      let url = `https://wa.me/${phone}?text=${encodeURIComponent(msg)}`;
      window.open(url, '_blank');
    }

    function getLocation() {
      navigator.geolocation.getCurrentPosition(position => {
        const lat = position.coords.latitude;
        const lon = position.coords.longitude;
        document.getElementById('location').innerText = `Latitude: ${lat}, Longitude: ${lon}`;
      });
    }

    function openMap() {
      navigator.geolocation.getCurrentPosition(position => {
        const lat = position.coords.latitude;
        const lon = position.coords.longitude;
        window.open(`https://www.google.com/maps?q=${lat},${lon}`, '_blank');
      });
    }

    function getLastEmail() {
      document.getElementById('emailInfo').innerText = 'Last email: Subject - Hello, Sent at 10:05 AM';
    }
  </script>
</body>
</html>

