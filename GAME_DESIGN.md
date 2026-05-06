# Tasawuf Battle Loop — Game Design & System Architecture

## 1) High Concept
**Tasawuf Battle Loop** adalah web-based, turn-based spiritual roguelike yang berfokus pada **pembacaan kondisi batin** lalu memilih **aksi (skill ability)** yang paling tepat. Pemain tidak menjawab soal teori; pemain menavigasi dinamika hati, ego, niat, dan refleksi dalam bentuk loop keputusan.

- **Genre:** Decision-based spiritual roguelike
- **Core fantasy:** “Menjernihkan batin dari gangguan nafs dalam putaran hidup yang berulang.”
- **Tone:** Mystical, calm, introspective, RPG-like, slightly abstract but playable

---

## 2) Core Loop (Turn-Based)
Setiap turn terdiri dari 4 fase:

1. **Narrative Event Trigger**
   - Sistem membuat 1 event naratif (1–3 kalimat)
   - Event menggambarkan kondisi manusiawi (emosi, godaan, keraguan, ibadah, ujub, refleksi)
   - Event **tidak menyebut jawaban**

2. **Skill Choice System**
   - Tampilkan 4 skill acak dari skill pool
   - Tepat 1 skill adalah solusi paling efektif
   - 3 skill lain terlihat plausible (misleading) namun tidak optimal

3. **Resolution**
   - Skill benar: **+20 Tasawuf Level**
   - Skill salah: **-10 Tasawuf Level**
   - Tampilkan feedback naratif singkat berdasarkan efek pilihan

4. **Loop Continues**
   - Generate event baru
   - Randomize 4 skill baru
   - Ulangi sampai win/lose

---

## 3) Core Stats & Win/Lose
- **Tasawuf Level:** 0–100
- **Start:** 50

### Win Condition
- **Tasawuf Level ≥ 100** → _Ma’rifat state achieved_

### Lose Condition
- **Tasawuf Level ≤ 0** → _Return to nafs state_

---

## 4) Skill System Architecture

## Skill Taxonomy (domain batin)
Agar balancing konsisten, tiap skill diberi domain:
- **Purification** (penyucian niat, reset ego)
- **Restraint** (menahan impuls, anti-reaktif)
- **Awareness** (mengenali pola batin, clarity)
- **Devotion** (koneksi ibadah & kehadiran hati)
- **Compassion** (melembutkan relasi sosial)

### Skill Data Model (JSON)
```json
{
  "id": "ego_disintegration_field",
  "name": "Ego Disintegration Field",
  "domain": "Purification",
  "tags": ["riya", "ujub", "self-image"],
  "power": 1.0,
  "risk": 0.2,
  "rarity": "common",
  "description": "Membubarkan lapisan citra diri yang mencari validasi.",
  "fx_text": "Kabut pembenaran diri menipis, niat menjadi lebih jernih."
}
```

### Event Data Model (JSON)
```json
{
  "id": "event_subtle_pride_after_charity",
  "text": "Setelah membantu seseorang, hatimu hangat, lalu muncul dorongan kecil agar orang lain tahu kebaikanmu.",
  "difficulty": 2,
  "tags": ["riya", "ujub", "social"],
  "correct_skill_id": "ego_disintegration_field",
  "misleading_skill_ids": [
    "aqua_wudhu_flow",
    "truth_awakening_pulse",
    "nafs_suppression_protocol"
  ]
}
```

### Matching Rule
- Event memiliki `tags`
- Skill memiliki `tags` + `domain`
- Skill yang benar dipilih dari score tertinggi terhadap event
- 3 decoy dipilih dari score menengah agar terlihat meyakinkan

Contoh scoring:
`match_score = (tag_overlap * 0.6) + (domain_affinity * 0.3) + (context_bonus * 0.1)`

---

## 5) Turn Engine (Pseudo Flow)
```text
init:
  tasawuf = 50

while 0 < tasawuf < 100:
  event = generate_event(player_state, difficulty_tier)
  options = generate_4_skill_options(event, skill_pool)
  choice = player_select(options)

  if choice == event.correct_skill_id:
      tasawuf += 20
      feedback = success_narrative(event, choice)
  else:
      tasawuf -= 10
      feedback = fail_narrative(event, choice)

  clamp tasawuf to [0, 100]
  update_pattern_memory(event, choice)

if tasawuf >= 100: WIN
if tasawuf <= 0: LOSE
```

---

## 6) Difficulty Scaling
Gunakan tier progresif berdasarkan jumlah turn atau Tasawuf band:

- **Tier 1 (Tasawuf 0–39):** event direct, decoy jelas beda domain
- **Tier 2 (40–69):** event lebih halus, decoy makin mirip
- **Tier 3 (70–89):** event multi-layer (emosi + niat), decoy hampir setara
- **Tier 4 (90–99):** event subtle (kesombongan halus, self-justification)

Parameter scaling:
- `decoy_similarity` naik per tier
- `event_ambiguity` naik per tier
- `feedback_specificity` turun sedikit (player harus membaca pola)

---

## 7) Boss Variant System
Setiap run bisa ditutup boss encounter batin:

### Boss: **The Mirror of Nafs**
- Muncul saat Tasawuf 80+
- 3 fase, masing-masing 1 event chain
- Salah di fase boss memberi penalti lebih besar: **-15**
- Benar memberi reward: **+25**

Boss traits:
- Meniru pola skill yang sering dipilih pemain
- Menghasilkan event yang mengeksploitasi bias pemain

Tujuan boss:
- Memaksa pemain lepas dari “hafalan skill”
- Menuntut pembacaan konteks secara murni

---

## 8) Skill Pool (Starter 24 Abilities)

1. Aqua Wudhu Flow
2. Ego Disintegration Field
3. Truth Awakening Pulse
4. Nafs Suppression Protocol
5. Sabr Fortress Stance
6. Shukr Resonance Wave
7. Ikhlas Refraction Seal
8. Tawakkul Gravity Well
9. Muraqabah Focus Lens
10. Muhasabah Echo Recall
11. Silence of the Tongue Ward
12. Lowered Gaze Barrier
13. Compassion Bloom Aura
14. Adab Alignment Matrix
15. Zuhud Detachment Drift
16. Qana’ah Stabilizer
17. Repentance Reboot Rite
18. Breath of Tuma’ninah
19. Intention Recalibration Node
20. Humility Grounding Chain
21. Forgiveness Release Beam
22. Presence in Prayer Channel
23. Restraint Lock Circuit
24. Heart Polishing Pulse

Balancing kasar:
- 10 common, 8 uncommon, 6 rare
- Rare bukan selalu “lebih benar”; hanya lebih niche atau berdampak narratif unik

---

## 9) 30 Narrative Events + Correct Mapping (Sample Pack)

| # | Event (ringkas) | Correct Skill |
|---|---|---|
| 1 | Setelah dipuji, kamu ingin mengulang amal agar dilihat lagi. | Ego Disintegration Field |
| 2 | Amarah naik saat pesanmu diabaikan di grup kerja. | Sabr Fortress Stance |
| 3 | Ibadah terasa mekanis, tubuh hadir hati tidak. | Presence in Prayer Channel |
| 4 | Kamu menunda minta maaf karena merasa “aku yang paling benar”. | Humility Grounding Chain |
| 5 | Rezeki teman naik cepat, hatimu menyempit diam-diam. | Qana’ah Stabilizer |
| 6 | Kamu tergoda membalas komentar tajam dengan sindiran halus. | Silence of the Tongue Ward |
| 7 | Setelah gagal, kamu yakin semua pintu tertutup. | Tawakkul Gravity Well |
| 8 | Fokus ibadah pecah oleh notifikasi tanpa henti. | Muraqabah Focus Lens |
| 9 | Kamu merasa tenang tapi sadar itu mungkin cuma pembenaran diri. | Muhasabah Echo Recall |
| 10 | Ada dorongan pamer kedisiplinan spiritual di media sosial. | Ikhlas Refraction Seal |
| 11 | Pandanganmu liar saat hati sedang kosong. | Lowered Gaze Barrier |
| 12 | Orang rumah mengulang kesalahan yang sama, kesal menumpuk. | Compassion Bloom Aura |
| 13 | Kamu gelisah karena hasil belum terlihat meski usaha konsisten. | Tawakkul Gravity Well |
| 14 | Kamu menolak bantuan karena takut terlihat lemah. | Humility Grounding Chain |
| 15 | Kesibukan membuat dzikir jadi sekadar checklist. | Heart Polishing Pulse |
| 16 | Kamu terlalu keras pada diri sendiri setelah satu kesalahan kecil. | Forgiveness Release Beam |
| 17 | Kamu ingin bicara jujur tapi takut kehilangan citra baik. | Truth Awakening Pulse |
| 18 | Perdebatan fiqih berubah jadi adu ego terselubung. | Adab Alignment Matrix |
| 19 | Kamu sulit berhenti membandingkan hidupmu dengan orang lain. | Zuhud Detachment Drift |
| 20 | Kebaikanmu tidak dihargai, lalu niatmu melemah. | Intention Recalibration Node |
| 21 | Kamu mulai merasa “sudah lebih suci” dari temanmu. | Ego Disintegration Field |
| 22 | Hati berdebar sebelum ibadah, pikiran melompat ke urusan dunia. | Breath of Tuma’ninah |
| 23 | Kamu ingin menegur, tapi caramu berpotensi mempermalukan. | Adab Alignment Matrix |
| 24 | Kamu menyesal kebiasaan lama muncul lagi setelah taubat. | Repentance Reboot Rite |
| 25 | Konflik kecil membuatmu ingin memutus silaturahmi. | Forgiveness Release Beam |
| 26 | Saat sendiri, keinginan impulsif terasa sangat kuat. | Restraint Lock Circuit |
| 27 | Kamu bingung: ini ilham atau sekadar keinginan terselubung? | Muraqabah Focus Lens |
| 28 | Ada rasa cukup, tapi disertai malas bertumbuh. | Intention Recalibration Node |
| 29 | Kamu menikmati pujian tentang kesabaranmu, lalu mencari panggung baru. | Ikhlas Refraction Seal |
| 30 | Dalam sujud, muncul ketakutan bahwa semua ini sia-sia. | Shukr Resonance Wave |

> Format ini siap diperluas ke 100+ event menggunakan template + tag combinator.

---

## 10) Content Generation Framework (100+ Event)
Gunakan template berbasis komponen:

`[Trigger sosial/privat] + [reaksi batin] + [dorongan nafs tersembunyi]`

Contoh generator:
- Trigger: dipuji / diremehkan / gagal / ditinggal / ditunggu hasil
- Reaksi: marah / takut / hampa / bangga / putus asa
- Dorongan: pamer / menghakimi / menunda / lari / menyalahkan

Dengan 12 trigger × 10 reaksi × 10 dorongan = **1200 kombinasi** mentah,
lalu dipilih & dikurasi untuk kualitas naratif.

---

## 11) Web Architecture (Practical)

### Frontend
- Framework: React / Vue (bebas)
- State: turn state machine (`idle -> event -> choice -> resolution -> next_turn`)
- UI Panels:
  - Narrative panel
  - 4 skill cards
  - Tasawuf meter (0–100)
  - Run log (last 5 turns)

### Backend
- Service modular:
  1. `EventService` (retrieve/generate event)
  2. `SkillService` (pool, rarity, tags)
  3. `MatchService` (correct + decoy selection)
  4. `ResolutionService` (+20 / -10)
  5. `RunService` (save state, history)

### Data Storage
- MVP: JSON files / SQLite
- Scale: Postgres (events, skills, runs, analytics)

---

## 12) Learning Pattern & Replayability
Untuk memastikan ada “sense progression”:
- Track `mistake_tags` per run
- Naikkan probabilitas event dengan tag yang sering salah
- Tampilkan end-run reflection:
  - “Kamu sering tersandung pada: validasi sosial, reaktivitas, dan putus asa halus.”

Ini menciptakan loop belajar personal, bukan hafalan jawaban.

---

## 13) Next Build Targets
1. Lengkapi database **100+ event** + mapping
2. Tambahkan **seeded randomness** untuk reproducible runs
3. Implement boss encounter 3 fase
4. Tambah mode harian (“Daily Inner Trial”)
5. Tambah telemetry balancing (win rate per skill/event)

