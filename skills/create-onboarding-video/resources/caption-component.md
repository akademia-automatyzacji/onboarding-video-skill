# Caption component reference

Bundled resource dla skilla `create-onboarding-video`. **Wczytaj tylko gdy dodajesz lub edytujesz captiony.** Beaty bez captionów (czysto wizualne, sam UI motion niesie) nie potrzebują tego pliku.

## Zasady

Captiony to headline-size callouty na górze ramki — nie subtitle-size labels. Identyfikują CO user właśnie zobaczył, niesione przez UI motion poniżej. Stosuj te zasady żeby były spójne między beatami:

1. **Widoczne cały beat.** Fade-in w pierwszych ~10-14 klatkach beata, zostają do końca sekwencji. Nigdy nie opóźniaj entry do połowy beata, nigdy nie fade-out przed końcem beata. Crossfade scen między beatami przenosi zmianę captionu.

2. **Rise-in z dołu.** Start ~60px pod rest position z opacity 0. Slide w górę + fade-in jednocześnie, mocny UI ease-out (`Easing.bezier(0.16, 1, 0.3, 1)`). Nigdy nie pojawiają się w miejscu, nigdy nie spadają z góry — ruch w górę to część visual identity.

3. **Na górze, zawsze ten sam spot.** Anchor każdy caption do fixed top-of-frame position (np. ~100px od góry, horizontally centered). Reuse tę pozycję we wszystkich beatach. **Nigdy** poniżej focal UI, nigdy nie dryftuje między beatami. Zarezerwuj "caption band" (~200-240px) na górze i layoutuj focal slice poniżej. Zbuduj jeden `TopCaption` wrapper component i używaj wszędzie — nie pozycjonuj inline per scena.

4. **Duże.** Default font size dla 1080-wide canvas: ~54px, weight 700, z `maxWidth` żeby długie linie wrapowały zamiast wybiegać poza ramkę.

5. **Ten sam caption między connected beatami zostaje w miejscu.** Gdy dwa kolejne beaty są częściami tego samego momentu i dzielą **dokładnie ten sam caption text** (np. beat 1 pokazuje tap na dzień, beat 2 pokazuje otwarcie formularza — ten sam headline), caption **nie może** re-animować przy cięciu. Rise i fade-in raz na pierwszym beacie, na każdym continuation beacie render z `staticEntry` (czyli: instant full opacity, rest position, brak slide'a). Dwa captiony composite identically podczas scene crossfade, więc viewer widzi jeden caption persisting cross-cut, nie flicker. **Tylko** gdy text jest *dokładnie* taki sam — jak caption się zmienia, niech nowy rise-and-fade-in normalnie, żeby zmiana była czytelna.

## Source komponentu

```tsx
// src/components/Caption.tsx — wklej verbatim do projektu.
import React from "react";
import { Easing, interpolate, useCurrentFrame } from "remotion";

const CAPTION_TOP = 100;          // px od góry ramki
const CAPTION_RISE_DISTANCE = 60; // px slide w górę przy entry
const FADE_IN_FRAMES = 14;        // długość rise + fade-in

/**
 * Caption headline na górze ramki. Rise-in z dołu + fade-in jednocześnie.
 * Zostaje widoczny do końca beata — scena-level crossfade obsługuje przejście.
 */
export const TopCaption: React.FC<{
  children: React.ReactNode;
  staticEntry?: boolean; // true gdy continuation beat z tym samym textem
  fontSize?: number;
  color?: string;
  maxWidth?: number;
}> = ({
  children,
  staticEntry = false,
  fontSize = 54,
  color = "rgba(15, 23, 42, 0.95)",
  maxWidth = 920,
}) => {
  const frame = useCurrentFrame();

  const progress = staticEntry
    ? 1
    : interpolate(frame, [0, FADE_IN_FRAMES], [0, 1], {
        easing: Easing.bezier(0.16, 1, 0.3, 1),
        extrapolateLeft: "clamp",
        extrapolateRight: "clamp",
      });

  const translateY = CAPTION_RISE_DISTANCE * (1 - progress);

  return (
    <div
      style={{
        position: "absolute",
        top: CAPTION_TOP,
        left: 0,
        right: 0,
        display: "flex",
        justifyContent: "center",
        pointerEvents: "none",
      }}
    >
      <div
        style={{
          fontSize,
          fontWeight: 700,
          color,
          textAlign: "center",
          maxWidth,
          lineHeight: 1.2,
          opacity: progress,
          transform: `translateY(${translateY}px)`,
        }}
      >
        {children}
      </div>
    </div>
  );
};

// Eksportuj też stałą żeby scene'y mogły layoutować slice poniżej caption band.
export const CAPTION_BAND = 220; // ~caption_top + miejsce na 2 linie
```

## Canonical pattern — beat z captionem

```tsx
// src/scenes/BeatX.tsx
import { AbsoluteFill } from "remotion";
import { TopCaption, CAPTION_BAND } from "../components/Caption";
import { Slice } from "../components/Slice";

export const BeatX: React.FC = () => {
  return (
    <AbsoluteFill
      style={{
        background: "#F8F2E5",
        paddingTop: CAPTION_BAND, // rezerwuj górny pas dla captionu
        alignItems: "center",
      }}
    >
      <TopCaption>Wybierz dzień</TopCaption>
      <Slice src="calendar.png" sx={42} sy={150} sw={1000} sh={800} />
    </AbsoluteFill>
  );
};
```

## Canonical pattern — continuation beat (ten sam text)

```tsx
// Beat 1 — tap na dzień
<TopCaption>Wybierz dzień</TopCaption>

// Beat 2 — formularz się otwiera, ten sam headline kontekstowo
// staticEntry=true → brak re-animacji, viewer widzi jeden caption cross-cut
<TopCaption staticEntry>Wybierz dzień</TopCaption>

// Beat 3 — caption SIĘ ZMIENIA → normalna animacja
<TopCaption>Potwierdź wybór</TopCaption>
```

## Częste błędy

| Błąd | Fix |
|------|-----|
| Caption pojawia się w miejscu zamiast rise-in | Sprawdź `translateY` interpolation — start 60px below, end 0px |
| Caption fade-outuje przed końcem beata | Nie animuj fade-out w komponencie — niech scene crossfade to obsłuży |
| Caption dryftuje między beatami (różny top) | Hardcode `CAPTION_TOP` w komponencie. Nigdy nie pozycjonuj inline w scenie |
| Continuation beat re-animuje ten sam text → flicker | Dodaj `staticEntry` na drugim beacie |
| Caption pod focal UI zamiast nad | Anchor `position: absolute, top: 100` — focal slice musi być w `paddingTop: CAPTION_BAND` parent |
| Długi text wybiega poza ramkę | Zostaw default `maxWidth: 920` albo zmniejsz dla długich PL fraz |

## Kiedy NIE używać captionu

- Pierwszy beat establishing context — niech focal UI sam się wprowadzi.
- Ostatni beat z result/success state — wynik mówi sam za siebie.
- Beat czysto wizualny (parallax, glow, shared-element morph bez interakcji) — motion niesie znaczenie.

W tych przypadkach **nie** renderuj `TopCaption` — pusty caption band też nie jest potrzebny, ustaw mniejszy `paddingTop`.
