# create-onboarding-video — Claude Code skill

> Tworzy krótkie, dynamiczne wideo onboardingowe aplikacji mobilnej w **Remotion**, animując wycięte fragmenty UI (komponenty, nie pełne ekrany) z transitions w stylu App Store preview.

Output: MP4 9:16 portrait (1080×1920) lub landscape 16:9, sklejony z 3-5 beatów po 3-8s każdy. Idealne do **rolek na Reels / TikTok / Shorts / X / LinkedIn**, App Store previews, prezentacji feature'ów na live'ach.

> Skill jest po polsku (description + workflow + komentarze). Kod TypeScript w resources/ pozostaje po angielsku. Forked z [bidah/skill-set](https://github.com/bidah/skill-set) + zoptymalizowany pod polski workflow content marketingowy.

---

## Szybka instalacja (Claude Code)

Wymaga zainstalowanego CLI [`skills`](https://github.com/anthropics/skills) (`npx skills`) — opisany krócej w dokumencji Anthropic Agent Skills.

**Krok 1 — zainstaluj ten skill (globalnie lub per-project):**

```bash
# Globalnie (dostępny w każdej sesji Claude Code)
npx skills add akademia-automatyzacji/onboarding-video-skill --global --yes --agent claude-code

# Albo per-project (tylko w aktualnym workspace)
npx skills add akademia-automatyzacji/onboarding-video-skill --yes --agent claude-code
```

**Krok 2 — zainstaluj companion skill `remotion-best-practices`:**

Ten skill **wymaga** [`remotion-best-practices`](https://github.com/remotion-dev/skills) jako źródła prawdy dla kodu Remotion (interpolate, useCurrentFrame, Sequence, fontów, ffmpeg, transitions). Skill `create-onboarding-video` deleguje wszystkie zasady kodu do tego companion skill'a.

```bash
npx skills add remotion-dev/skills --global --yes --agent claude-code
```

Po instalacji oba skille są widoczne w liście dostępnych skilli sesji Claude Code:
- `create-onboarding-video` — workflow (intake → plan beat-by-beat → render → output)
- `remotion-best-practices` — zasady kodu Remotion + 33 plików rules (transitions, audio, captions, ffmpeg, fonts, lottie, timing, transcribe-captions)

---

## Wymagania systemowe

| Co | Wersja | Po co |
|----|--------|-------|
| **Node.js** | 18+ | Runtime Remotion |
| **npm** lub **bun** | — | Package manager |
| **ffmpeg** | 4.0+ | Encoding MP4 (Remotion go używa) |
| **macOS / Linux / Windows** | — | Cross-platform |
| **Chromium / Chrome Headless Shell** | — | Pobierany automatycznie przy pierwszym renderze Remotion (~92 MB) |

Sprawdzenie:

```bash
node -v && npm -v && ffmpeg -version | head -1
```

---

## Setup projektu Remotion (jednorazowo)

Skill zakłada że projekt Remotion istnieje w wybranym katalogu. Jeśli nie masz — scaffolduj:

```bash
# Wybierz lokalizację working dir (skill domyślnie wskazuje ~/Documents/Kodowanie/onboarding-videos/)
mkdir -p ~/Documents/Kodowanie/onboarding-videos
cd ~/Documents/Kodowanie/onboarding-videos

# Init projektu + dependencies
npm init -y
npm install remotion @remotion/cli @remotion/player react react-dom
npm install -D @types/react @types/react-dom typescript
```

Minimalna struktura folderu:

```
onboarding-videos/
├── package.json
├── remotion.config.ts
├── tsconfig.json
├── public/                 ← screenshoty wrzucasz tutaj
│   └── moj-projekt/
│       ├── ekran-1.png
│       └── ekran-2.png
└── src/
    ├── index.ts            ← registerRoot
    ├── Root.tsx            ← <Composition id="..." />
    ├── theme.ts            ← brand colors, fonts, helpers
    ├── components/         ← Slice, Caption, Cursor
    └── scenes/             ← Beat1.tsx, Beat2.tsx, orchestrator
```

Skill samodzielnie zbuduje strukturę gdy wywołasz go z working dir.

Komendy po setupie:

```bash
npm start                    # Remotion Studio (live preview w przeglądarce)
npx remotion render <CompositionId> out/output.mp4
```

---

## Pierwsze użycie

W sesji Claude Code:

> "Stwórz mi onboarding video dla apki [X]. Mam screeny w `[ścieżka]`."

Skill triggeruje się na frazy typu:
- "onboarding video", "App Store preview", "feature demo", "demo clip"
- "rolka demo z UI aplikacji", "krótkie wideo do social media (reels/shorts/TikTok)"
- "Remotion + screenshoty"

Workflow który skill prowadzi:

1. **Intake** — zbiera od ciebie 2-4 stille per ekran (resting + mid-interaction + result state), intencję feature'u, kolejność, brand (kolory/font/aspect ratio)
2. **Plan beat-by-beat** — identyfikuje *focal element* per beat (przycisk, toggle, sheet, chart), długości w klatkach, captiony, użycie Pointer+TapDot vs GlowRing
3. **Implementacja** — scene'y Remotion z komponentami `Slice` (crop ze stilla), `TopCaption` (rise-in headline), `Pointer + TapDot` (cursor lead + ripple), `GlowRing` (illustrative pulse)
4. **Iteracja** — preview MP4, feedback, popraw, render final

---

## Struktura skilla

```
skills/create-onboarding-video/
├── SKILL.md                          ← workflow + operating rules + konkretny example beat
└── resources/
    ├── caption-component.md          ← TopCaption: rise-in, top-anchor, staticEntry pattern (~150 linii)
    └── cursor-component.md           ← Pointer + TapDot + GlowRing: cursor-leads-tap, canonical patterns (~330 linii)
```

Resources ładują się **on-demand** — tylko gdy Claude dodaje/edytuje beat z captionem lub tapem. SKILL.md sam ma ~270 linii, opisuje workflow i delegowane reguły.

---

## Output convention (opcjonalna)

Skill ma zaszyte konwencje dla workflow Obsidian/Marketing (Akademia Automatyzacji). Jeśli używasz innej struktury — Claude doadaptuje:

- Render final → `~/Documents/Kodowanie/onboarding-videos/out/<nazwa>.mp4`
- Publikacja → `Marketing/media/<nazwa>.mp4`
- Hub rolki → `Marketing/projekt-rolki/YYYY-MM-DD-<nazwa>.md` z embedem `![[Marketing/media/<nazwa>.mp4]]`

Można dostosować w SKILL.md (sekcja "Output").

---

## Wymagania dla agenta (Claude Code)

Skill jest projektowany tak, że agent samodzielnie:
- pyta o brakujące stille / brand / aspect ratio (`AskUserQuestion`)
- buduje strukturę projektu Remotion jeśli folder pusty
- wywołuje companion skill `remotion-best-practices` przed pisaniem kodu
- iteruje na bazie feedbacku użytkownika
- nie zaczyna renderowania bez kompletu stilli

Twoja rola: dostarczyć **2-4 screenshoty per ekran** + **opisać feature** + **wybrać aspect ratio**.

---

## Licencja

MIT — zobacz [LICENSE](LICENSE).

---

## Credits

- Forked z [bidah/skill-set](https://github.com/bidah/skill-set) (skill `create-onboarding-video`)
- Companion `remotion-best-practices` z [remotion-dev/skills](https://github.com/remotion-dev/skills)
- Built na [Remotion](https://www.remotion.dev/) (React → MP4)
- Polski workflow + optymalizacje pod content marketing [Akademia Automatyzacji](https://akademiaautomatyzacji.com)
