---
name: create-onboarding-video
description: Tworzy krótkie, dynamiczne wideo onboardingowe aplikacji mobilnej w Remotion, pokazujące feature w akcji przez animowanie wyciętych fragmentów UI (komponenty, nie pełne ekrany) z płynnymi przejściami w stylu UI. Używaj gdy user prosi o stworzenie onboarding video, App Store preview, feature demo, rolki demo z UI aplikacji, krótkiego wideo do social media (reels/shorts/TikTok), klipu prezentującego funkcję aplikacji, embedowanego wideo z UI, lub gdy wspomina o Remotion w kontekście tworzenia wideo na bazie dostarczonych screenshotów.
---

# Create Onboarding Video

Tworzysz **krótkie, dynamiczne wideo onboardingowe** w Remotion pokazujące feature aplikacji mobilnej w akcji. Efekt ma wyglądać jak App Store preview zoomowane na moment udowadniający że feature działa — nie tutorial, nie nagranie ekranu, nie reklama marketingowa.

## Setup

Working dir: `~/Documents/Kodowanie/onboarding-videos/` — projekt Remotion już zainicjalizowany (Remotion 4.0.461 + React 19 + TS). Komendy:
- `npm start` — Remotion Studio (live preview w przeglądarce)
- `npm run build` — render final do `out/`

Jeśli folder nie istnieje albo jest pusty, scaffolduj: `mkdir -p ~/Documents/Kodowanie/onboarding-videos && cd $_ && npm init -y && npm i remotion @remotion/cli @remotion/player react react-dom && npm i -D @types/react @types/react-dom typescript`.

## Co robisz

- **Długość:** krótko. 3-8 sekund per ekran onboardingowy, sklejone w całość. Całe wideo rzadko przekracza ~30s.
- **Styl:** UI-first, **NIGDY cały ekran**. Każdy beat pokazuje **kawałek feature'u w akcji** — jeden tapowany przycisk, przełączany toggle, przesuwany row, sheet jadący w górę, wypełniający się wykres — animowane przejściami w stylu UI (springi, slide'y, scale, crossfade, masked reveals, shared-element swaps).
- **"Kawałek" znaczy:** wytnij / wymaskuj / wyizoluj **konkretny komponent** ze screenshota — kartę, input, tab bar, empty state przechodzący w filled state. Reszta UI omitted, blurred albo zasugerowana tintowym tłem. Pokazujesz **co feature *robi*,** nie jak wygląda cała aplikacja.
- **Ton:** do rzeczy. Każdy beat komunikuje jedną rzecz którą feature robi.
- **Output:** wideo MP4 w `Marketing/media/<nazwa>.mp4` + stub w `Marketing/projekt-rolki/YYYY-MM-DD-<nazwa>.md` z embedem `![[Marketing/media/<nazwa>.mp4]]`.

## Aspect ratio per platforma

| Platforma | Format | Wymiary |
|-----------|--------|---------|
| App Store / TikTok / Reels / Shorts | portrait 9:16 | 1080×1920 |
| LinkedIn / YouTube / X | landscape 16:9 | 1920×1080 |
| Square (FB feed) | 1:1 | 1080×1080 |

Domyślnie 1080×1920 portrait, ale w intake zawsze pytaj.

## Workflow

Trzymaj się tej pętli. **Nie pomijaj intake'u — zgadywanie flow daje generyczne wideo.**

### 1. Intake — zbierz stille + intencję

Dla każdego ekranu który user chce pokazać, zbierz:

1. **Screenshoty** — **2-4 stille per ekran** żeby pokazać stany interakcji:
   - resting state
   - mid-interaction (przycisk wciśnięty, focused field, sheet w połowie itp.)
   - result state (data loaded, success, kolejny ekran)
   - opcjonalne warianty (empty vs filled, light vs dark itp.)
2. **Co robi feature** — 1-2 zdania o tym co ten ekran daje userowi i co sprawia że feel-good. To decyduje na którym detalu się zoomujesz.
3. **Kolejność** — sekwencja ekranów w onboardingu.
4. **Brand & format:** kolory, font, aspect ratio, end-card / CTA — **zawsze pytaj per projekt**, nie zakładaj defaultów. Każdy projekt (Padlement, Olimp, Akademia Automatyzacji, klient X) ma własną paletę.
5. **Typ UI (driver cursora):** mobile app (iPhone/Android screenshoty) → `Pointer` touch dot. Web/desktop (landing page, dashboard, SaaS) → `MousePointer` (arrow/hand). Trzymaj konsekwencję w obrębie wideo.

Użyj `AskUserQuestion` jak user jest niejasny. Nie zaczynaj renderowania bez stilli + intencji per ekran.

### 2. Zaplanuj shoty

Dla każdego ekranu zidentyfikuj **jeden konkretny kawałek feature'u który udowadnia że feature działa** — tapowany przycisk, wypełniający się progress ring, swipowany row, autouzupełniany field — i przejście do kolejnego beatu. **NIGDY nie animuj całego ekranu.** Naszkicuj timeline (focal element → motion → result → next focal element) zanim napiszesz komponenty. Preferuj:

- izolowanie / cropowanie / maskowanie konkretnego komponentu ze stilla na tintowanym tle
- pokazywanie *samej interakcji* (tap ripple, drag, focus, state change), nie statycznego layoutu
- shared-element transitions między beatami (przycisk z beata 1 staje się headerem beata 2)
- subtelny parallax / depth na warstwach
- spring motion zamiast linear easing

### 3. Buduj w Remotion

**Zawsze wywołaj skill `remotion-best-practices` przed pisaniem kodu Remotion.** To źródło prawdy dla kodu — tam są rules o `useCurrentFrame`, `interpolate`, `<Sequence>`, fontach, ffmpeg, transitions.

Konwencje projektu (`~/Documents/Kodowanie/onboarding-videos/`):
- Stille w `public/<screen-name>/<state>.png`
- Jedna `<Composition>` per onboarding flow, jedna `<Sequence>` per beat
- Komponenty w `src/scenes/`, shared transitions w `src/transitions/`
- 30fps, wymiary z propsów (`width`/`height` jako compositionProps)

### 4. Iteruj

Wyrenderuj preview (`npm start` w projekcie), pokaż userowi, zapytaj które beaty wolniej/szybciej/inaczej. Pierwszy render to draft.

### 5. Output

Po akceptacji:
1. Render final: `npm run build -- --output=out/<nazwa>.mp4`
2. Skopiuj do workspace: `cp out/<nazwa>.mp4 ~/Documents/kacper_trzepiecinski_workspace/Marketing/media/<nazwa>.mp4`
3. Utwórz stub w `Marketing/projekt-rolki/YYYY-MM-DD-<nazwa>.md` z embedem `![[Marketing/media/<nazwa>.mp4]]`

## Operating rules

- **Stille obowiązkowe.** Bez screenshotów stop i pytaj. Nie wymyślaj UI z opisu.
- **Kawałki UI, nie cały UI.** Jak łapiesz się na renderowaniu full-screen mockupu — stop, crop do komponentu który nosi beat.
- **Jeden feature per wideo.** Jak user opisuje 5 niezwiązanych features — zaproponuj podział na 5 wideo.
- **Show, don't narrate.** Bez voiceovera, bez wielkich napisów tłumaczących feature. UI motion niesie znaczenie. Krótki caption per beat OK.
- **Captions na top, widoczne cały beat, animacja rise-in.** Szczegóły, zasady i komponent: `resources/caption-component.md` — wczytaj **gdy** dodajesz lub edytujesz captiony.
- **Cursor prowadzi każdy tap.** Beat z tapem/kliknięciem/selekcją wymaga widocznego cursora który dolatuje do targetu. Dla **mobile UI** użyj `Pointer` (touch dot), dla **web/desktop UI** użyj `MousePointer` (SVG arrow lub hand) — konsekwentnie w obrębie jednego wideo. Szczegóły, kod komponentu i canonical patterns: `resources/cursor-component.md` — wczytaj **gdy** dodajesz lub edytujesz beat z tapem.
- **Match design language stilli.** Używaj kolorów, corner radii, typografii ze stilli. Nie restyluj.
- **Code rules delegowane.** Zasady kodu Remotion (interpolate vs CSS transitions, easing, static files, Sequencing, fonty) → skill `remotion-best-practices`. Nie dubluj.

## Konkretny example — anatomia beata

Tak wygląda pojedynczy beat z working onboarding wideo Olimp (welcome screen z tapem na CTA "ZACZYNAM"). Trzy primitywy: `Slice` (cropowany screenshot), `TopCaption` (rise-in headline), `Pointer + TapDot` (cursor lead + ripple). Wszystko w jednym `position: relative` parent żeby coords cursora były slice-local.

```tsx
// src/scenes/OlimpLogin/Beat1Welcome.tsx
import { AbsoluteFill, Easing, interpolate, spring, useCurrentFrame, useVideoConfig } from 'remotion';
import { Slice } from '../../components/Slice';
import { Pointer, TapDot } from '../../components/Cursor';
import { TopCaption } from '../../components/Caption';
import { CAPTION_BAND, COLORS, p } from '../../theme';

const SCREEN_W = 760;
const SCREEN_H = 1645;
const SCALE = SCREEN_W / 390; // stille z Figmy są 390px logical

// Coords CTA w slice-space (390×844 source → ×SCALE)
const CTA_X = 195 * SCALE;
const CTA_Y = 720 * SCALE;

const TAP_AT = p(95); // tap firing 95 klatek w beat

export const Beat1Welcome: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Screen entry — spring scale + fade żeby beat wjeżdżał z energią
  const entryScale = spring({ frame, fps, config: { damping: 14, mass: 0.7 } });
  const sliceScale = interpolate(entryScale, [0, 1], [0.95, 1]);
  const sliceOpacity = interpolate(frame, [0, 14], [0, 1], { extrapolateRight: 'clamp' });

  // Pointer fade-in w centrum slice'a, glide do CTA
  const fadeIn = interpolate(frame, [p(40), p(55)], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });
  const fadeOut = interpolate(frame, [TAP_AT + p(10), TAP_AT + p(22)], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });
  const pointerOpacity = fadeIn * (1 - fadeOut);

  // Jeden prosty ruch z centrum slice'a do CTA — single normalised progress driving x i y
  const moveProgress = interpolate(frame, [p(55), TAP_AT], [0, 1], {
    easing: Easing.bezier(0.16, 1, 0.3, 1),
    extrapolateLeft: 'clamp',
    extrapolateRight: 'clamp',
  });
  const pointerX = interpolate(moveProgress, [0, 1], [SCREEN_W / 2, CTA_X]);
  const pointerY = interpolate(moveProgress, [0, 1], [SCREEN_H / 2, CTA_Y]);

  return (
    <AbsoluteFill style={{ background: COLORS.bg, paddingTop: CAPTION_BAND, alignItems: 'center' }}>
      <TopCaption>30 sekund</TopCaption>
      <div
        style={{
          position: 'relative',     // wszystkie cursor coords liczone od tego div'a
          width: SCREEN_W,
          height: SCREEN_H,
          opacity: sliceOpacity,
          transform: `scale(${sliceScale})`,
          transformOrigin: 'center top',
        }}
      >
        <Slice src="olimp-login/onboarding-1.png" width={SCREEN_W} />
        <Pointer x={pointerX} y={pointerY} opacity={pointerOpacity} />
        <TapDot tapAt={TAP_AT} x={CTA_X} y={CTA_Y} size={130} />
      </div>
    </AbsoluteFill>
  );
};
```

Co warto wyłapać:
- **`paddingTop: CAPTION_BAND`** — rezerwuje 240px na górze pod caption, slice ląduje pod nim
- **`position: relative` parent** — Pointer i TapDot to siblings Slice'a w tym samym coord space, dzielą `x`/`y`
- **Spring entry + interpolate scale** — beat ma "wjazd" z 0.95 do 1, czuje się jak otwierający transition
- **Pointer fade-in PRZED move** — viewer widzi że cursor *materializuje się* w centrum, potem płynnie glide do CTA. Bez teleportacji
- **TapDot `tapAt={TAP_AT}` exactly w tej samej klatce** w której Pointer dolatuje — kontakt fires gdy cursor jest tam

Dla beata bez tapu (illustrative, np. dashboard reveal): pomiń `Pointer + TapDot`, użyj `GlowRing` z `resources/cursor-component.md`. Dla multi-tap na tym samym UI: zobacz pattern B w `resources/cursor-component.md`.

## W razie wątpliwości

Pytaj. 10-sekundowe pytanie wyjaśniające oszczędza 2-minutowy render który chybił.
