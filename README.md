<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ders Değişimi</title>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
  * { box-sizing: border-box; }
  body {
    font-family: Arial, sans-serif;
    background: #f2f4f7;
    margin: 0;
    padding: 20px;
  }
  .login-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: calc(100vh - 40px);
  }
  .login-box {
    background: white;
    padding: 32px;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    width: 100%;
    max-width: 360px;
  }
  .login-box h1 {
    font-size: 20px;
    margin-bottom: 20px;
    text-align: center;
  }
  .login-box input {
    width: 100%;
    padding: 10px;
    margin-bottom: 12px;
    border: 1px solid #ccc;
    border-radius: 6px;
    font-size: 15px;
  }
  .login-box button {
    width: 100%;
    padding: 10px;
    background: #2563eb;
    color: white;
    border: none;
    border-radius: 6px;
    font-size: 15px;
    cursor: pointer;
  }
  .login-box button:hover { background: #1d4ed8; }
  #hata-mesaji {
    color: #dc2626;
    font-size: 13px;
    margin-top: 8px;
    text-align: center;
  }
  #app-alani {
    display: none;
    max-width: 480px;
    margin: 0 auto;
  }
  .topbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
  }
  #hosgeldin-mesaji { font-weight: bold; }
  .topbar button {
    padding: 8px 14px;
    background: #dc2626;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
  }
  .card {
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    padding: 16px 20px;
    margin-bottom: 16px;
  }
  .card label {
    display: block;
    font-weight: bold;
    margin-bottom: 8px;
    font-size: 14px;
    color: #444;
  }
  .card input, .card select {
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 6px;
    font-size: 15px;
  }
  .program-tablosu {
    width: 100%;
    border-collapse: collapse;
  }
  .program-tablosu th, .program-tablosu td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
    font-size: 14px;
  }
  .program-tablosu th { background: #f2f4f7; }
</style>
</head>
<body>

<div class="login-wrapper" id="giris-wrapper">
  <div id="giris-alani" class="login-box">
    <h1>Ders Değişimi - Giriş</h1>
    <input type="text" id="kullanici-adi" placeholder="Kullanıcı adı veya email">
    <input type="password" id="sifre" placeholder="Şifre">
    <button onclick="girisYap()">Giriş Yap</button>
    <div id="hata-mesaji"></div>
  </div>
</div>

<div id="app-alani">
  <div class="topbar">
    <span id="hosgeldin-mesaji"></span>
    <button onclick="cikisYap()">Çıkış Yap</button>
  </div>

  <div class="card">
    <label>Tarih</label>
    <input type="date" id="tarih-secici">
  </div>

  <div class="card">
    <label>Öğretmen</label>
    <select id="ogretmen-secici">
      <option value="">-- Öğretmen seçin --</option>
    </select>
  </div>

  <div class="card">
    <label>Günün Programı</label>
    <div id="program-listesi">Önce tarih ve öğretmen seçin.</div>
  </div>
</div>

<script>
  const SUPABASE_URL = "https://pnkpovgpmraaduuniurz.supabase.co";
  const SUPABASE_ANON_KEY = "sb_publishable_N3lbUDiriU7EhHspUOY8pw_2Kw1OR_G";
  const supabaseClient = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

  const GUN_ISIMLERI = { 1: 'Pazartesi', 2: 'Salı', 3: 'Çarşamba', 4: 'Perşembe', 5: 'Cuma' };

  async function girisYap() {
    const girilenDeger = document.getElementById('kullanici-adi').value.trim();
    const sifre = document.getElementById('sifre').value;
    const hataAlani = document.getElementById('hata-mesaji');
    hataAlani.textContent = "";

    const email = girilenDeger.includes('@') ? girilenDeger : girilenDeger + '@ders.local';

    const { data, error } = await supabaseClient.auth.signInWithPassword({
      email: email,
      password: sifre
    });

    if (error) {
      hataAlani.textContent = "Giriş başarısız: " + error.message;
      return;
    }

    ekraniGuncelle(data.user);
  }

  async function cikisYap() {
    await supabaseClient.auth.signOut();
    ekraniGuncelle(null);
  }

  function ekraniGuncelle(kullanici) {
    if (kullanici) {
      document.getElementById('giris-wrapper').style.display = 'none';
      document.getElementById('app-alani').style.display = 'block';
      document.getElementById('hosgeldin-mesaji').textContent = 'Hoş geldin: ' + kullanici.email;
      oturumBaslat();
    } else {
      document.getElementById('giris-wrapper').style.display = 'flex';
      document.getElementById('app-alani').style.display = 'none';
    }
  }

  async function oturumBaslat() {
    const tarihInput = document.getElementById('tarih-secici');
    const bugun = new Date();
    tarihInput.value = bugun.toISOString().split('T')[0];

    await ogretmenleriYukle();

    tarihInput.addEventListener('change', programGetir);
    document.getElementById('ogretmen-secici').addEventListener('change', programGetir);
  }

  async function ogretmenleriYukle() {
    const secici = document.getElementById('ogretmen-secici');
    secici.innerHTML = '<option value="">-- Öğretmen seçin --</option>';

    const { data, error } = await supabaseClient
      .from('teachers')
      .select('id, full_name, branch')
      .order('full_name');

    if (error) {
      console.error(error);
      return;
    }

    data.forEach(ogretmen => {
      const secenek = document.createElement('option');
      secenek.value = ogretmen.id;
      secenek.textContent = ogretmen.full_name + ' (' + ogretmen.branch + ')';
      secici.appendChild(secenek);
    });
  }

  function gunIndexHesapla(tarihStr) {
    const tarih = new Date(tarihStr + 'T00:00:00');
    const jsGun = tarih.getDay();
    if (jsGun === 0 || jsGun === 6) return null;
    return jsGun;
  }

  async function programGetir() {
    const tarihStr = document.getElementById('tarih-secici').value;
    const ogretmenId = document.getElementById('ogretmen-secici').value;
    const listeAlani = document.getElementById('program-listesi');

    if (!tarihStr || !ogretmenId) {
      listeAlani.textContent = 'Önce tarih ve öğretmen seçin.';
      return;
    }

    const gunIndex = gunIndexHesapla(tarihStr);
    if (gunIndex === null) {
      listeAlani.textContent = 'Seçilen gün hafta sonu, ders yok.';
      return;
    }

    const { data, error } = await supabaseClient
      .from('schedule_slots')
      .select('period, classes(name)')
      .eq('teacher_id', ogretmenId)
      .eq('day_of_week', gunIndex)
      .order('period');

    if (error) {
      listeAlani.textContent = 'Hata: ' + error.message;
      return;
    }

    if (data.length === 0) {
      listeAlani.textContent = GUN_ISIMLERI[gunIndex] + ' günü bu öğretmenin dersi yok.';
      return;
    }

    let html = '<table class="program-tablosu"><tr><th>Saat</th><th>Sınıf</th></tr>';
    data.forEach(satir => {
      html += '<tr><td>' + satir.period + '</td><td>' + satir.classes.name + '</td></tr>';
    });
    html += '</table>';
    listeAlani.innerHTML = html;
  }

  supabaseClient.auth.getSession().then(({ data: { session } }) => {
    ekraniGuncelle(session ? session.user : null);
  });
</script>

</body>
</html># ders-degisimi
