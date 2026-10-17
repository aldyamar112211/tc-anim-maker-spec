# Spesifikasi Proyek Anim Maker (versi 2)

Dokumen ini menjelaskan format JSON proyek animasi yang dibaca editor Anim Maker Terakhir Community (`/tools/anim-maker/app`). Format ini dipakai saat menempel JSON ke editor dan saat AI agent memakai `window.TCAnimMaker`.

Validasi mesin ada di [`schema.json`](schema.json). Contoh lengkap ada di folder [`contoh/`](contoh).

## 1. Struktur dasar

```json
{
  "versi": 2,
  "nama": "Angguk",
  "rig": "R15",
  "durasi": 1,
  "fps": 30,
  "atur": { "priority": 2, "loop": true, "fps": 30, "halus": 0, "follow": 0, "napas": 0, "ayun": 0 },
  "track": { },
  "event": [ ],
  "layer": [ ]
}
```

| Field | Tipe | Aturan |
|---|---|---|
| `versi` | angka | WAJIB `2`. Proyek tanpa `versi: 2` ditolak. |
| `nama` | teks | Maksimal 50 karakter. Dipakai sebagai nama aset saat diunggah. |
| `rig` | `"R15"` atau `"R6"` | Selain `"R6"` dianggap R15. |
| `durasi` | detik | 1/120 sampai 600. |
| `fps` | angka | Salah satu dari 24, 30, 60, 120. Selain itu menjadi 30. |
| `atur` | objek | Lihat bagian 5. |
| `track` | objek | Keyframe per sendi. Lihat bagian 3. |
| `event` | array | Marker. Lihat bagian 4. |
| `layer` | array | Opsional, layer transparansi. Boleh dikosongkan. |

## 2. Rig dan sendi

Nama sendi harus persis seperti di bawah. Sendi yang tidak ada di daftar rig yang dipilih diabaikan.

**R15 (15 sendi)**

| Sendi | Induk |
|---|---|
| LowerTorso | HumanoidRootPart |
| UpperTorso | LowerTorso |
| Head | UpperTorso |
| LeftUpperArm, RightUpperArm | UpperTorso |
| LeftLowerArm / RightLowerArm | UpperArm sisi yang sama |
| LeftHand / RightHand | LowerArm sisi yang sama |
| LeftUpperLeg, RightUpperLeg | LowerTorso |
| LeftLowerLeg / RightLowerLeg | UpperLeg sisi yang sama |
| LeftFoot / RightFoot | LowerLeg sisi yang sama |

**R6 (6 sendi)**

| Sendi | Induk |
|---|---|
| Torso | HumanoidRootPart |
| Head, Left Arm, Right Arm, Left Leg, Right Leg | Torso |

Nilai setiap sendi adalah **offset dari pose diam** (pose T/berdiri bawaan Roblox), relatif terhadap induknya. Nilai 0 di semua kanal berarti pose diam.

## 3. Track dan keyframe

```json
"track": {
  "Head": {
    "p": [[], [], []],
    "r": [
      [ { "t": 0, "v": 0 }, { "t": 0.5, "v": -15 }, { "t": 1, "v": 0 } ],
      [],
      []
    ]
  }
}
```

- `p` = posisi (satuan stud), `r` = rotasi (satuan **derajat**).
- Masing-masing berisi 3 kanal berurutan: **X, Y, Z**.
- Rotasi diterapkan dengan urutan Euler **XYZ**.
- Kanal kosong `[]` berarti sendi itu tidak bergerak pada sumbu tersebut.
- Posisi biasanya hanya dipakai untuk sendi akar (`LowerTorso` di R15, `Torso` di R6), misalnya menurunkan badan saat jongkok. Sendi lain cukup dirotasi.

**Isi keyframe**

| Field | Wajib | Arti |
|---|---|---|
| `t` | ya | Waktu (detik), 0 sampai `durasi`. Dibulatkan ke tick internal. |
| `v` | ya | Nilai (stud atau derajat). |
| `i` | tidak | Interpolasi ke key berikutnya: `0` konstan, `1` linear, `2` kubik halus otomatis (default), `3` easing. |
| `e` | jika `i: 3` | Kode easing `gaya.arah`, contoh `"back.out"`. |
| `m` | tidak | Mode tangen: `"a"` otomatis (default), `"d"`, `"m"` manual. |
| `l`, `r` | tidak | Tangen kiri dan kanan untuk mode manual. Abaikan kalau tidak perlu. |
| `id` | tidak | Dibuat otomatis kalau kosong atau ganda. |

**Gaya easing:** `sine`, `quad`, `cubic`, `quart`, `quint`, `sextic`, `expo`, `circ`, `back`, `bounce`, `elastic`.
**Arah easing:** `in`, `out`, `inout`, `outin`.

Saran: biarkan `i` kosong (kubik halus otomatis). Editor menghitung tangen sendiri sehingga gerakan halus tanpa melenceng. Pakai `i: 3` hanya untuk aksen tertentu, misalnya pantulan (`back.out`) saat mendarat.

## 4. Event (marker)

```json
"event": [
  { "t": 0.42, "nama": "Pukul", "nilai": "", "panjang": 0 }
]
```

- `nama` maksimal 64 karakter. Di Roblox dibaca lewat `AnimationTrack:GetMarkerReachedSignal("Pukul")`.
- `nilai` opsional (maksimal 64 karakter), dikirim sebagai parameter marker.
- `panjang` opsional (detik) untuk event berdurasi.
- **Jangan menaruh marker tepat di detik 0.** Marker di 0 tidak pernah terpanggil di Roblox. Pakai minimal 0.03 detik.

## 5. Pengaturan (`atur`)

| Field | Arti |
|---|---|
| `priority` | Prioritas animasi Roblox: 0 Idle, 1 Movement, 2 Action (default), 3 Action2, 4 Action3, 5 Action4, 1000 Core. |
| `loop` | `true` untuk animasi berulang. |
| `fps` | FPS ekspor: 30 atau 60. |
| `halus`, `follow`, `napas`, `ayun` | Efek tambahan otomatis 0 sampai 100. Biarkan 0 kecuali diminta pengguna. |
| `jenis` | Opsional, kata kecil tanpa spasi (misal `"jalan"`), membantu pemeriksa kaki. |

## 6. Konvensi rotasi R15 (terbukti di Studio)

| Sendi | Sumbu | Efek |
|---|---|---|
| Head | X positif | Mendongak |
| UpperTorso | X negatif | Condong ke depan |
| RightUpperArm | X positif | Lengan diangkat ke depan / atas |
| RightUpperArm | Z negatif | Lengan menyilang ke dalam badan |
| RightUpperArm | Z positif | Lengan membuka ke luar |
| Left/Right LowerArm | X positif | Siku menekuk normal |
| Left/Right LowerLeg | X negatif | Lutut menekuk normal |

Sisi kiri adalah cerminan sisi kanan: untuk sendi kiri, nilai rotasi Y dan Z dibalik tandanya, X tetap.

## 7. Aturan kualitas (WAJIB untuk AI)

Animasi yang kaku langsung terasa murahan. Setiap animasi harus:

1. **Antisipasi**: sebelum gerakan besar ada gerakan kecil berlawanan arah (ambil ancang-ancang).
2. **Follow-through dan overlap**: kepala, tangan, dan bagian ujung sedikit telat dari badan, lalu sedikit melewati posisi akhir sebelum diam.
3. **Tidak ada sendi yang diam total**: badan selalu sedikit bernapas atau bergoyang, terutama pada animasi loop.
4. **Interpolasi halus**: gunakan kubik otomatis (default). Hindari linear kecuali untuk gerakan mekanis.
5. **Loop menyambung**: untuk `loop: true`, nilai key di `t = 0` dan `t = durasi` harus sama di setiap kanal.
6. **Lantai**: badan tidak tenggelam atau melayang, kecuali memang sedang lompat atau terbang.
7. **Anatomi**: lutut dan siku tidak menekuk terbalik, tangan tidak menembus badan atau kepala.
8. **Kaki**: untuk animasi yang dipakai sambil berjalan, hanya gerakkan badan atas (UpperTorso, Head, lengan) supaya kaki tetap mengikuti animasi jalan.

## 8. Pemeriksaan otomatis di editor

Setelah JSON dimuat, editor memeriksa:

- Animasi kosong (belum ada key)
- Key yang lewat dari durasi
- Loop yang tidak menyambung
- Rotasi yang melompat (beda sudut terlalu jauh antar key)
- Kaki meluncur dan jalan di tempat
- Diam terlalu lama di akhir animasi
- Priority yang kurang cocok
- Key yang tidak berpengaruh

Kalau ada temuan, perbaiki JSON lalu muat ulang.

## 9. Batas

| Hal | Batas |
|---|---|
| Durasi | 600 detik |
| Total key | 300.000 |
| Event | 5.000 |
| Layer | 500 |

## 10. Unggah dan kepemilikan

- Unggah ke Roblox dilakukan pengguna dari editor, dengan API key Open Cloud milik pengguna sendiri. AI tidak boleh meminta atau memegang API key pengguna.
- Animasi hanya bisa diputar di place milik akun atau grup yang sama dengan pengunggahnya.
- Batas unggah mengikuti paket akun Terakhir Community (free atau TC Supporter). Jangan mencoba mengakali batas ini.
