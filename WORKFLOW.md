# Magic Podcast — Kompletter Workflow

> Von der Aufnahme bis Millionen Views auf allen Plattformen.

## Host & Kanal

| | |
|---|---|
| **Host** | Jonas Schiffmann |
| **YouTube** | @millionenchemiker |
| **Zielgruppe** | DACH-Raum (Schweiz, Deutschland, Österreich) |
| **Sprache** | Deutsch |
| **Strategie** | Growth Hacking + Multi-Plattform Dominanz |

---

## Übersicht: Die Pipeline

```
Riverside Aufnahme
      │
      ├─ 1. Export (WAV + MP4 → Mac)
      │
      ├─ 2. Audio-Mastering (ffmpeg + sox)
      │      Noise Reduction → EQ → Kompression → Limiter → Loudness
      │
      ├─ 3. Transkription + Kapitel (Whisper + Claude API)
      │      Transkript → Kapitelmarken → Show Notes → Social Posts
      │
      ├─ 4. Filler-Wörter entfernen ("Ähm", "Also", Pausen)
      │
      ├─ 5. Video-Produktion
      │      ├─ YouTube Full Episode (1080p/4K, Thumbnail via Canva)
      │      └─ 10-15 Clips (Shorts, Reels, TikTok, LinkedIn)
      │
      ├─ 6. Distribution
      │      ├─ Podcast: Transistor.fm → Spotify, Apple, Amazon, 15+ Plattformen
      │      ├─ YouTube: Full Episode + Shorts
      │      └─ Social: TikTok, Instagram, LinkedIn, X
      │
      └─ 7. Growth Engine
             Newsletter → Community → Monetarisierung
```

---

## Phase 1: Aufnahme (Riverside)

### Setup
- **Qualität:** 48kHz, WAV (unkomprimiert)
- **Video:** Bis 4K pro Teilnehmer
- **Separate Tracks:** Jeder Sprecher bekommt eigene Audio- + Video-Spur
- **Auto-Export:** Riverside → Google Drive oder Dropbox konfigurieren

### Nach der Aufnahme
1. In Riverside auf "Export" klicken
2. Format: **WAV** (Audio) + **MP4** (Video)
3. "Separate Tracks" aktivieren (wichtig für Multitrack-Editing)
4. Dateien auf den Mac in `/Users/jonasschiffmann/magic-podcast/raw/` ablegen

### Ordnerstruktur
```
magic-podcast/
├── raw/                    # Rohdateien von Riverside
│   └── EP001/
│       ├── host.wav
│       ├── guest.wav
│       ├── combined.wav
│       └── video.mp4
├── mastered/               # Fertig gemasterte Audio-Dateien
├── transcripts/            # Transkripte + SRT-Dateien
├── clips/                  # Geschnittene Clips für Social Media
├── thumbnails/             # YouTube Thumbnails
├── show-notes/             # Show Notes + Beschreibungen
├── exports/                # Finale Export-Dateien
│   ├── podcast/            # MP3 für Podcast-Plattformen
│   ├── youtube/            # MP4 für YouTube
│   └── social/             # Clips für TikTok, Reels, Shorts
└── scripts/                # Automations-Scripts
```

---

## Phase 2: Audio-Mastering

### Professionelle Mastering-Chain (Reihenfolge wichtig!)

```
Roh-Audio
  │
  1. Noise Profile erstellen (aus stiller Stelle)
  2. Noise Reduction (Hintergrundrauschen entfernen)
  3. Highpass Filter 80Hz (Rumpeln/Handling entfernen)
  4. Lowpass Filter 16kHz (Zischen entfernen)
  5. EQ (Stimme formen)
     - 200Hz: -3dB (Mulm reduzieren)
     - 3kHz: +2.5dB (Präsenz/Klarheit)
     - 8kHz: +1.5dB (Luft/Brillanz)
  6. Kompressor (Lautstärke ausgleichen)
  7. Limiter (Clipping verhindern)
  8. Loudness-Normalisierung
     - Podcast (RSS): -19 LUFS
     - YouTube/Spotify: -16 LUFS
```

### Befehle

**Schritt 1 — Noise Profile + Reduction (sox):**
```bash
# Noise Profile aus den ersten 0.5 Sekunden (Stille)
sox raw/EP001/combined.wav -n trim 0 0.5 noiseprof /tmp/noise.prof

# Rauschen entfernen
sox raw/EP001/combined.wav /tmp/denoised.wav noisered /tmp/noise.prof 0.21
```

**Schritt 2 — Komplette Mastering-Chain (ffmpeg):**
```bash
ffmpeg -i /tmp/denoised.wav -af \
"highpass=f=80,"\
"lowpass=f=16000,"\
"equalizer=f=200:t=q:w=1.5:g=-3,"\
"equalizer=f=3000:t=q:w=1.0:g=2.5,"\
"equalizer=f=8000:t=q:w=1.5:g=1.5,"\
"acompressor=threshold=-18dB:ratio=3:attack=5:release=200:makeup=2,"\
"alimiter=limit=0.95:level=false,"\
"loudnorm=I=-19:TP=-1.0:LRA=7" \
-ar 44100 -c:a pcm_s24le mastered/EP001_mastered.wav
```

**Schritt 3 — Export-Formate:**
```bash
# Podcast (MP3, 128kbps, -19 LUFS, Mono)
ffmpeg -i mastered/EP001_mastered.wav -ac 1 -b:a 128k exports/podcast/EP001.mp3

# YouTube/Spotify (AAC, 192kbps, -16 LUFS, Stereo)
ffmpeg -i mastered/EP001_mastered.wav -af "loudnorm=I=-16:TP=-1.5:LRA=11" \
  -c:a aac -b:a 192k exports/youtube/EP001.m4a

# Archiv (FLAC, verlustfrei)
ffmpeg -i mastered/EP001_mastered.wav -c:a flac exports/archive/EP001.flac
```

### Quick One-Liner (alles in einem Befehl):
```bash
sox raw/EP001/combined.wav -n trim 0 0.5 noiseprof /tmp/n.prof && \
sox raw/EP001/combined.wav /tmp/clean.wav noisered /tmp/n.prof 0.21 && \
ffmpeg -i /tmp/clean.wav -af \
"highpass=f=80,lowpass=f=16000,equalizer=f=200:t=q:w=1.5:g=-3,equalizer=f=3000:t=q:w=1.0:g=2.5,acompressor=threshold=-18dB:ratio=3:attack=5:release=200:makeup=2,alimiter=limit=0.95,loudnorm=I=-19:TP=-1.0:LRA=7" \
-ar 44100 -b:a 128k exports/podcast/EP001.mp3
```

---

## Phase 3: Transkription + Content-Generierung

### Transkription mit Whisper (lokal auf Mac)

```bash
# Audio für Whisper vorbereiten (16kHz, Mono)
ffmpeg -i mastered/EP001_mastered.wav -ar 16000 -ac 1 -c:a pcm_s16le /tmp/EP001_16k.wav

# Transkribieren (Deutsch, mit Timestamps)
whisper-cpp \
  --model /usr/local/share/whisper.cpp/models/ggml-large-v3.bin \
  --language de \
  --output-srt \
  --output-txt \
  --threads 8 \
  /tmp/EP001_16k.wav
```

**Output:**
- `EP001_16k.srt` — Untertitel mit Zeitstempeln
- `EP001_16k.txt` — Reiner Text

### Filler-Wörter entfernen ("Ähm", "Also", etc.)

```bash
# Whisper mit Wort-Level-Timestamps
whisper mastered/EP001_mastered.wav \
  --model large-v3 --language de \
  --output_format json --word_timestamps True \
  --output_dir transcripts/
```

Dann über ein Script die Filler erkennen und per ffmpeg rausschneiden:
- "ähm", "äh", "uhm", "halt", "sozusagen", "quasi", "also" (als Füllwort)
- Pausen über 0.8 Sekunden werden auf 0.3 Sekunden gekürzt

### Content-Generierung mit Claude API

Aus dem Transkript generiert Claude automatisch:
- **Kapitelmarken** (Timestamps für YouTube + Podcast)
- **Show Notes** (Zusammenfassung, Key Takeaways, Links)
- **Social Media Posts** (5-10 Posts pro Episode)
- **Newsletter-Text** (Top 3 Insights)
- **Blog-Artikel** (SEO-optimiert, 1000-2000 Wörter)
- **Clip-Vorschläge** (Timestamps der besten Stellen für Shorts/Reels)

---

## Phase 4: Video-Produktion

### YouTube Full Episode

**Option A — Riverside-Video verwenden (mit Gesicht):**
```bash
# Video + gemastertes Audio zusammenführen
ffmpeg -i raw/EP001/video.mp4 -i exports/youtube/EP001.m4a \
  -c:v copy -c:a aac -b:a 192k \
  -map 0:v:0 -map 1:a:0 \
  exports/youtube/EP001_final.mp4
```

**Option B — Audiogramm (ohne Gesicht):**
```bash
ffmpeg -i exports/youtube/EP001.m4a -loop 1 -i thumbnails/EP001_bg.png \
-filter_complex \
"[0:a]showwaves=s=1080x200:mode=cline:rate=30:colors=0xE8A838:scale=sqrt[waves];
 [1:v]scale=1920:1080[bg];
 [bg][waves]overlay=420:800[v1];
 [v1]subtitles=transcripts/EP001.srt:force_style='FontSize=24,FontName=Helvetica Neue,PrimaryColour=&H00FFFFFF,Outline=2,Alignment=2,MarginV=100'[outv]" \
-map "[outv]" -map 0:a \
-c:v libx264 -preset medium -crf 23 \
-c:a aac -b:a 192k \
exports/youtube/EP001_audiogram.mp4
```

### Thumbnail (via Canva MCP)
- **Grösse:** 1280 x 720 Pixel (16:9)
- **Elemente:** Ausdrucksstarkes Gesicht + max 5 Wörter + Kontrast-Farben
- **Konsistenz:** Gleiche Template-Vorlage für jede Episode (Branding)
- **Ziel-CTR:** 6-10%

### Clips für Social Media (10-15 pro Episode)

**Optimale Clip-Längen:**

| Plattform | Länge | Format |
|-----------|-------|--------|
| YouTube Shorts | 45-58 Sek. | 1080x1920 (9:16) |
| TikTok | 60-75 Sek. | 1080x1920 (9:16) |
| Instagram Reels | 30-45 Sek. | 1080x1920 (9:16) |
| LinkedIn | 60-90 Sek. | 1080x1080 (1:1) |
| X/Twitter | 30-45 Sek. | 1080x1080 (1:1) |

**Clip-Produktion:**
```bash
# Clip ausschneiden (ab 12:30, 58 Sekunden)
ffmpeg -ss 00:12:30 -i exports/youtube/EP001_final.mp4 -t 58 \
  -c:v libx264 -c:a aac clips/EP001_clip01_raw.mp4

# Vertikal machen (9:16) mit Untertiteln
ffmpeg -i clips/EP001_clip01_raw.mp4 -filter_complex \
"[0:v]scale=1080:1920:force_original_aspect_ratio=decrease,pad=1080:1920:(ow-iw)/2:(oh-ih)/2[v1];
 [v1]subtitles=clips/EP001_clip01.srt:force_style='FontSize=32,FontName=Helvetica Neue Bold,PrimaryColour=&H0000FFFF,OutlineColour=&H00000000,Outline=3,Alignment=2,MarginV=350'[outv]" \
-map "[outv]" -map 0:a \
-c:v libx264 -crf 20 -c:a aac -b:a 192k \
exports/social/EP001_clip01_short.mp4
```

### 5 Virale Clip-Formeln

1. **Kontroverse Aussage** — "Die meisten denken X, aber in Wahrheit ist es Y"
2. **Emotionale Story** — Persönlich, verletzlich, mit Höhepunkt
3. **Taktischer Nugget** — Ein konkreter, sofort umsetzbarer Tipp
4. **Jaw-Drop Fakt** — Überraschende Statistik oder Erkenntnis
5. **Hitzige Diskussion** — Respektvolle Meinungsverschiedenheit

**Clip-Checkliste:**
- [ ] Hook in den ersten 2 Sekunden
- [ ] Untertitel (fett, zentriert, Key-Words farbig hervorgehoben)
- [ ] Progress Bar am oberen Rand
- [ ] CTA in den letzten 3 Sekunden ("Ganze Episode: Link in Bio")
- [ ] Titel-Text oben im Bild (3-7 Wörter)

---

## Phase 5: Distribution

### Podcast-Plattformen via Transistor.fm

**Warum Transistor ($19/Mo):**
- Saubere REST API (vollautomatisch)
- Unlimited Episodes
- Verteilt automatisch an: Spotify, Apple Podcasts, Amazon Music, iHeartRadio, Samsung, Pocket Casts, Overcast, Castro, 15+ weitere
- Eigene Podcast-Website
- Analytics

**API-Upload (automatisiert):**
```bash
curl -X POST https://api.transistor.fm/v1/episodes \
  -H "x-api-key: YOUR_KEY" \
  -F "episode[show_id]=SHOW_ID" \
  -F "episode[title]=EP001: Titel der Episode" \
  -F "episode[description]=Show Notes hier..." \
  -F "episode[audio_url]=https://storage.url/EP001.mp3"
```

### YouTube

**Upload-Checkliste:**
- [ ] Video: MP4 (H.264 + AAC), 1080p+, 48kHz
- [ ] Thumbnail: 1280x720, unter 2MB
- [ ] Titel: Max 60 Zeichen, Keyword vorne, emotional
- [ ] Beschreibung: Hook in Zeile 1-2, Timestamps, Links, Tags
- [ ] Kapitel: Mindestens 8-15, erste bei 0:00
- [ ] Tags: 10-15 relevante Keywords
- [ ] Untertitel: Korrigierte SRT-Datei hochladen
- [ ] End Screen: Nächste Episode + Abo-Button
- [ ] Cards: 2-4 Links zu verwandten Episoden
- [ ] Playlist: Episode zur Podcast-Playlist hinzufügen

**Titel-Formate die funktionieren:**
- "[Emotionaler Hook] — [Gast] | Magic Podcast #001"
- "Warum 95% bei [Thema] scheitern (und wie Du es richtig machst)"
- "[Gast]: [Kontroverse Aussage] | Magic Podcast"

**Beschreibungs-Template:**
```
[Hook-Satz mit Haupt-Keyword]. [Zweiter Satz mit Mehrwert-Versprechen].

⏱ Kapitel:
0:00 Intro
2:15 [Thema 1]
8:30 [Thema 2]
15:45 [Thema 3]
...

🎧 Überall hören:
Spotify: [Link]
Apple Podcasts: [Link]

📧 Newsletter: [Link]

🔗 Gast:
[Website, Social Media]

📚 Erwähnte Ressourcen:
[Bücher, Tools, Links]

👉 Folge Magic Podcast:
Instagram: @handle
TikTok: @handle

#podcast #keyword1 #keyword2
```

### Social Media Clips

**Posting-Zeiten (DACH-Raum):**

| Plattform | Beste Tage | Beste Zeiten |
|-----------|-----------|-------------|
| YouTube (Full) | Di, Do | 08:00-10:00 |
| YouTube Shorts | Täglich | 12:00-14:00, 19:00-21:00 |
| TikTok | Di-Fr | 10:00, 14:00, 19:00 |
| Instagram Reels | Mo-Fr | 11:00-13:00, 19:00-21:00 |
| LinkedIn | Di-Do | 07:30-08:30, 12:00 |
| X/Twitter | Mo-Fr | 08:00-10:00, 12:00-13:00 |

**Posting-Frequenz:**

| Plattform | Minimum | Optimal |
|-----------|---------|---------|
| YouTube Full | 1x/Woche | 2x/Woche |
| YouTube Shorts | 3x/Woche | 1x/Tag |
| TikTok | 1x/Tag | 2-3x/Tag |
| Instagram Reels | 3x/Woche | 5-7x/Woche |
| LinkedIn | 2x/Woche | 3-5x/Woche |
| X/Twitter | 1x/Tag | 3-5x/Tag |

---

## Phase 6: Growth Engine

### Newsletter-Funnel

```
Podcast-Hörer
  → Clip auf Social Media
    → Ganze Episode auf YouTube/Spotify
      → Newsletter Opt-in (Link in Bio / Show Notes)
        → Welcome Sequence (5 Emails: Beste Episoden + Mehrwert + Angebot)
          → Zahlender Kunde / Community-Mitglied / Superfan
```

**Email-Capture Touchpoints:**
- In der Episode: 2x erwähnen (bei ~15 Min + am Ende), mit konkretem Lead Magnet
- Show Notes: Opt-in-Form above the fold
- Link in Bio (alle Plattformen): Email-Capture als erstes
- Clip-CTA: "Ganzes Breakdown gratis — Link in Bio"
- YouTube-Beschreibung: Newsletter-Link in den ersten 2 Zeilen

**Newsletter-Rhythmus:** 1-2x pro Woche
**Inhalt:** Top 3 Insights + persönliche Story + CTA (unter 500 Wörter)

### Community aufbauen

| Stufe | Plattform | Zugang | Zweck |
|-------|-----------|--------|-------|
| Gratis | YouTube, Instagram, X | Offen | Reichweite, Casual Fans |
| Engaged | Telegram / Discord | Gratis, Email-Gate | Core Fans, Diskussion |
| Premium | Skool / Patreon | Bezahlt ($5-50/Mo) | Superfans, exklusiver Content |

### Monetarisierung nach Grösse

**0-1'000 Downloads/Episode:**
- Eigene Produkte/Services verkaufen
- Affiliate Marketing ($200-1'000/Mo)

**1'000-10'000 Downloads/Episode:**
- Sponsorings ($500-5'000/Mo, CPM $18-50)
- Premium Content via Patreon ($500-3'000/Mo)
- Digitale Produkte ($2'000-10'000/Mo)

**10'000-50'000 Downloads/Episode:**
- Sponsorings ($5'000-30'000/Mo)
- YouTube AdSense ($3'000-15'000/Mo)
- Premium Community ($5'000-20'000/Mo)
- Live Events ($10'000-50'000/Event)

**50'000+ Downloads/Episode:**
- Sponsorings ($50'000-200'000+/Mo)
- YouTube AdSense ($20'000-100'000+/Mo)
- Buch-Deals, Live-Tours, Equity-Deals

**Revenue-Stack Beispiel (10K Downloads):**
```
Sponsorings (2 Sponsoren):    CHF 8'000/Mo
YouTube AdSense:              CHF 4'000/Mo
Patreon (150 Members, CHF 10): CHF 1'500/Mo
Affiliate Deals:              CHF 2'000/Mo
Eigenes Produkt:              CHF 5'000/Mo
──────────────────────────────────────────
Total:                        CHF 20'500/Mo ≈ CHF 246'000/Jahr
```

---

## Tech-Stack Übersicht

| Layer | Tool | Kosten/Mo |
|-------|------|-----------|
| **Aufnahme** | Riverside.fm | $15-24 |
| **Hosting/Distribution** | Transistor.fm | $19 |
| **Transkription** | Whisper (lokal) | Gratis |
| **Content-Generierung** | Claude API | ~$0.10-0.20/Episode |
| **Clip-Erstellung** | Opus Clip (API) | $19-49 |
| **Thumbnails** | Canva (MCP) | Gratis/Pro |
| **Social Scheduling** | Buffer oder Publer | $15-25 |
| **Newsletter** | GetResponse (bereits vorhanden) | — |
| **Audio-Mastering** | ffmpeg + sox (lokal) | Gratis |
| **Total** | | **~$70-120/Mo** |

---

## Automatisierter Workflow (Ziel)

```
Du nimmst auf in Riverside
        │
        ▼
Export auf Mac (manuell oder Auto-Export)
        │
        ▼
Du sagst mir: "Episode liegt bereit"
        │
        ▼
Ich mache ALLES automatisch:
  ├─ Audio mastern (ffmpeg + sox)
  ├─ Transkribieren (Whisper)
  ├─ Filler-Wörter entfernen
  ├─ Kapitel + Show Notes generieren (Claude API)
  ├─ Thumbnail erstellen (Canva MCP)
  ├─ YouTube-Video bauen (ffmpeg)
  ├─ 10-15 Clips schneiden (ffmpeg)
  ├─ Untertitel brennen
  ├─ Upload: Transistor (alle Podcast-Plattformen)
  ├─ Upload: YouTube (Full + Shorts)
  ├─ Social Posts generieren
  └─ Newsletter-Draft erstellen
        │
        ▼
Du prüfst + gibst frei
        │
        ▼
Live auf allen Plattformen 🎙
```

---

## Phase 7: DACH Growth Hacking Playbook

### Schnelles Wachstum im deutschsprachigen Raum

**Vorteil DACH:** Weniger Konkurrenz als US-Markt, aber kaufkräftige Zielgruppe mit hoher Podcast-Affinität. Deutschland hat 15+ Mio. regelmässige Podcast-Hörer, Schweiz ~2 Mio., Österreich ~1.5 Mio.

### Growth Hack #1 — YouTube Shorts → Podcast Funnel
- Jeden Tag 1-3 Shorts aus der Episode auf @millionenchemiker posten
- Shorts mit Hook-Untertiteln in Gelb/Gold (Branding)
- CTA: "Ganze Episode im Podcast — Link in Bio"
- Shorts sind der #1 Discovery-Kanal für neue Hörer im DACH-Raum

### Growth Hack #2 — Gast-Netzwerk-Effekt
- Gäste mit eigener Community einladen (10K+ Follower)
- Gast teilt Episode → sofortiger Zugang zu deren Audience
- Jeder Gast bringt 5-15% seiner Follower als neue Hörer
- Nach dem Interview: Clips taggen, Gast taggen → viraler Loop

### Growth Hack #3 — SEO-Transkripte
- Jede Episode als Blog-Artikel auf eigener Website veröffentlichen
- 5'000-15'000 Wörter SEO-Content PRO EPISODE (gratis!)
- Google rankt Podcast-Transkripte für Long-Tail Keywords
- Nach 50 Episoden: 500'000+ Wörter indexiert = organischer Traffic-Maschine

### Growth Hack #4 — Cross-Posting Strategie
- Gleicher Clip, 6 Plattformen, leicht angepasst:
  - YouTube Shorts: Hook + CTA zum Kanal
  - TikTok: Trending Sound unterlegen wenn passend
  - Instagram Reels: Carousel-Slide am Ende mit Key Takeaway
  - LinkedIn: Business-Framing, Text-Post + Video
  - X/Twitter: Kontroverser Ausschnitt, Thread dazu
  - Facebook Reels: Gleich wie Instagram, extra Reichweite

### Growth Hack #5 — Newsletter als Retention-Waffe
- GetResponse Welcome Sequence: 5 Emails mit besten Episoden
- Jede Woche: "Top 3 Insights" aus der neuen Episode
- Hörer die Email öffnen = 3x wahrscheinlicher regelmässige Hörer
- Newsletter-Abonnenten als Custom Audience für Ads (wenn gewünscht)

### Growth Hack #6 — Algorithmus-Hacks pro Plattform

**YouTube:**
- Erste 48h entscheiden über den Erfolg → alle Kanäle gleichzeitig pushen
- Community Tab nutzen: Poll vor der Episode ("Welches Thema interessiert Euch?")
- Playlists nach Themen, nicht nach Nummern
- Thumbnail A/B Testing (nach 48h ändern wenn CTR < 4%)

**Spotify:**
- Episode-Titel mit Keywords die Leute suchen
- In die richtige Kategorie eintragen (Charts-Ranking)
- Spotify Q&A Feature nutzen für Listener-Engagement
- Erste 30 Sekunden = Hook, kein langweiliges Intro

**Apple Podcasts:**
- Bewertungen aktiv in jeder 3. Episode anfragen
- "Neuer & Bemerkenswerter" Bereich = erste 8 Wochen sind kritisch
- Konsistenter Release-Schedule signalisiert dem Algorithmus Aktivität

**TikTok (DACH):**
- Deutsche Hashtags: #podcast #podcastdeutsch #motivation #persönlichkeitsentwicklung
- Posting-Zeit: 10:00 + 19:00 MEZ (Mittagspause + Feierabend)
- Reply auf Kommentare mit Video (Algorithmus-Boost)
- Duett/Stitch mit anderen Creators im gleichen Themenfeld

### KPIs & Meilensteine

| Meilenstein | Downloads/Ep | YouTube Views/Mo | Timeline |
|------------|-------------|-----------------|----------|
| Launch | 0 | 0 | Monat 1 |
| Traction | 100-500 | 5'000-20'000 | Monat 2-3 |
| Growth | 500-2'000 | 20'000-100'000 | Monat 4-6 |
| Momentum | 2'000-5'000 | 100'000-500'000 | Monat 6-12 |
| Scale | 5'000-20'000 | 500'000-2'000'000 | Monat 12-24 |
| Authority | 20'000+ | 2'000'000+ | Monat 24+ |

**Wichtigste Regel:** Jede Woche eine Episode. 52 Wochen am Stück. Keine Ausnahmen. Konsistenz schlägt Perfektion.

---

## Nächste Schritte

1. **Podcast-Name + Branding** festlegen
2. **Transistor.fm Account** erstellen
3. **Automations-Scripts** bauen (ich mache das)
4. **Erste Episode** aufnehmen und durch die Pipeline jagen
