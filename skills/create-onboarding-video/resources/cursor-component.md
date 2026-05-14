# Cursor component reference

Bundled resource dla skilla `create-onboarding-video`. **Wczytaj tylko gdy dodajesz lub edytujesz beat z tapem.** Czysto wizualne beaty (glow rings, animowane state changes, lądujące rezultaty) nie potrzebują cursora i nie potrzebują tego pliku.

Skill dostarcza trzy primitivy, wszystkie w jednym pliku `src/components/Cursor.tsx`:

| Primitive | Do czego |
|-----------|----------|
| `Pointer` | Persistent translucent kropka która **rusza się** żeby prowadzić wzrok. Wymagana przy każdym tapie. |
| `TapDot` | Krótki ripple w momencie tapu. Zawsze nakładaj **na** `Pointer` na tych samych coords. |
| `GlowRing` | Pulsująca poświata wokół chipa/przycisku do *highlightowania* feature'u bez tapu. Tylko na czysto wizualnych beatach. |

## Zasada której pilnuje ten resource

> **Cursor prowadzi każdy tap.** Sam tap ripple to za mało — viewer musi zobaczyć jak cursor doleciał do targetu **przed** ripplem. Brak teleportacji, brak jump-cutów.

Jeśli beat ma jakikolwiek tap/klik/selekcję, render `Pointer` którego coords interpolują z off-target start position do targetu, i nałóż `TapDot` w momencie kontaktu. Zawsze para — dzielą `x`/`y`.

## Source komponentu

```tsx
// src/components/Cursor.tsx — wklej verbatim do projektu.
import React from "react";
import { Easing, interpolate, useCurrentFrame } from "remotion";

/**
 * iOS-style touch indicator — translucent kółko które pojawia się w punkcie
 * tapu i pulsuje na zewnątrz.
 */
export const TapDot: React.FC<{
  tapAt: number;
  x: number;
  y: number;
  size?: number;
  color?: string;
}> = ({ tapAt, x, y, size = 110, color = "rgba(59, 130, 246, 0.55)" }) => {
  const frame = useCurrentFrame();
  const fade = interpolate(
    frame,
    [tapAt - 4, tapAt, tapAt + 18, tapAt + 28],
    [0, 1, 0.35, 0],
    { extrapolateLeft: "clamp", extrapolateRight: "clamp" },
  );
  const ringScale = interpolate(frame, [tapAt, tapAt + 24], [0.4, 1.8], {
    easing: Easing.bezier(0.16, 1, 0.3, 1),
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
  const ringFade = interpolate(frame, [tapAt, tapAt + 24], [0.7, 0], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
  const dotScale = interpolate(
    frame,
    [tapAt - 4, tapAt, tapAt + 8],
    [1, 0.78, 1],
    {
      easing: Easing.bezier(0.34, 1.56, 0.64, 1),
      extrapolateLeft: "clamp",
      extrapolateRight: "clamp",
    },
  );
  return (
    <div
      style={{
        position: "absolute",
        left: x - size / 2,
        top: y - size / 2,
        width: size,
        height: size,
        pointerEvents: "none",
      }}
    >
      <div
        style={{
          position: "absolute",
          inset: 0,
          borderRadius: "50%",
          border: `3px solid ${color}`,
          opacity: ringFade,
          transform: `scale(${ringScale})`,
        }}
      />
      <div
        style={{
          position: "absolute",
          inset: 0,
          borderRadius: "50%",
          background: color,
          opacity: fade,
          transform: `scale(${dotScale})`,
        }}
      />
    </div>
  );
};

/**
 * Persistent translucent kropka. Używaj żeby *prowadzić* wzrok do tap targetu.
 * Wymagana na każdym beacie który ma tap lub selekcję.
 */
export const Pointer: React.FC<{
  x: number;
  y: number;
  size?: number;
  opacity?: number;
}> = ({ x, y, size = 64, opacity = 1 }) => (
  <div
    style={{
      position: "absolute",
      left: x - size / 2,
      top: y - size / 2,
      width: size,
      height: size,
      borderRadius: "50%",
      background: "rgba(15, 23, 42, 0.42)",
      border: "4px solid rgba(255, 255, 255, 0.85)",
      boxShadow: "0 8px 24px rgba(15, 23, 42, 0.28)",
      opacity,
      pointerEvents: "none",
    }}
  />
);

/**
 * Look-here pulse dla *wizualnych* beatów. Bez implikowanego tapu.
 */
export const GlowRing: React.FC<{
  x: number;
  y: number;
  width: number;
  height: number;
  startAt: number;
  duration?: number;
  color?: string;
  radius?: number;
}> = ({
  x,
  y,
  width,
  height,
  startAt,
  duration = 36,
  color = "rgba(59, 130, 246, 0.85)",
  radius = 999,
}) => {
  const frame = useCurrentFrame();
  const progress = interpolate(
    frame,
    [startAt, startAt + duration / 2, startAt + duration],
    [0, 1, 0],
    {
      easing: Easing.bezier(0.45, 0, 0.55, 1),
      extrapolateLeft: "clamp",
      extrapolateRight: "clamp",
    },
  );
  const scale = 1 + progress * 0.08;
  return (
    <div
      style={{
        position: "absolute",
        left: x,
        top: y,
        width,
        height,
        borderRadius: radius,
        boxShadow: `0 0 0 4px ${color}, 0 0 30px 6px ${color}`,
        opacity: progress,
        transform: `scale(${scale})`,
        transformOrigin: "center",
        pointerEvents: "none",
      }}
    />
  );
};
```

## Canonical pattern A — pojedynczy tap (fade-in w centrum, glide do targetu)

Każdy single-tap beat ma ten dwufazowy kształt. Pointer **fade-in w wizualnym centrum** focal area, potem glide w **jednej prostej linii** (dowolny kierunek — pionowy, poziomy, diagonal) do punktu interakcji. Tap odpala gdy dotrze. Pointer fade-out w miejscu.

```tsx
// src/scenes/BeatX.tsx — single-tap interactive beat
import { AbsoluteFill, Easing, interpolate, useCurrentFrame } from "remotion";
import { Pointer, TapDot } from "../components/Cursor";
import { Slice } from "../components/Slice";
import { TopCaption } from "../components/Caption";
import { CAPTION_BAND, COLORS, FONT, p } from "../theme";

export const BeatX: React.FC = () => {
  const frame = useCurrentFrame();

  // 1. Single source of truth dla *kiedy* tap odpala.
  const tapAt = p(48);

  // 2. Centrum focal area jako start + target jako end, oba w tym samym
  //    coord space (slice-local albo container-local).
  const sliceWidth = 820;
  const sliceHeight = 808;
  const startX = sliceWidth / 2;
  const startY = sliceHeight / 2;
  const targetX = 220;
  const targetY = 580;

  // 3. Opacity: fade-in PRZED rozpoczęciem ruchu żeby pointer widocznie
  //    materializował się w centrum. Hold full opacity przez tap, potem
  //    fade-out w miejscu.
  const fadeIn = interpolate(frame, [p(6), p(16)], [0, 1], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
  const fadeOut = interpolate(
    frame,
    [tapAt + p(10), tapAt + p(20)],
    [0, 1],
    { extrapolateLeft: "clamp", extrapolateRight: "clamp" },
  );
  const pointerOpacity = fadeIn * (1 - fadeOut);

  // 4. JEDEN prosty ruch start → target. Pojedynczy normalised progress
  //    napędza x i y, więc ścieżka to prawdziwa prosta linia. Ruch zaczyna
  //    się PO zakończeniu fade-in żeby materializacja była widoczna.
  const moveProgress = interpolate(frame, [p(16), tapAt], [0, 1], {
    easing: Easing.bezier(0.16, 1, 0.3, 1),
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });
  const pointerX = interpolate(moveProgress, [0, 1], [startX, targetX]);
  const pointerY = interpolate(moveProgress, [0, 1], [startY, targetY]);

  return (
    <AbsoluteFill
      style={{
        background: COLORS.bg,
        fontFamily: FONT,
        alignItems: "center",
        justifyContent: "center",
        paddingTop: CAPTION_BAND,
      }}
    >
      <div style={{ position: "relative" }}>
        <Slice src="screen.png" sx={42} sy={150} sw={1095} sh={sliceHeight} width={sliceWidth} />
        {/* Zawsze renderuj Pointer + TapDot razem w tym samym coord space. */}
        <Pointer x={pointerX} y={pointerY} opacity={pointerOpacity} />
        <TapDot tapAt={tapAt} x={targetX} y={targetY} size={120} />
      </div>
      <TopCaption>Wybierz datę</TopCaption>
    </AbsoluteFill>
  );
};
```

## Canonical pattern B — wiele tapów na tym samym UI (jeden ciągły cursor)

Gdy pojedynczy beat ma dwa lub więcej tapów na **tym samym UI** (np. wybór segmentu, potem tap przycisku na tym samym formularzu), trzymaj **jeden persistent pointer** który fade-in raz w centrum, glide do pierwszego targetu, potem glide **bezpośrednio** z każdego targetu do następnego bez resetu. Fade-out dopiero po ostatnim tapie.

```tsx
const tapAAt = p(40);
const tapBAt = p(90);
const A = { x: 600, y: 60 };
const B = { x: 600, y: 250 };

// JEDEN pointer. JEDEN fade-in (w centrum). JEDEN fade-out (po ostatnim tapie).
const pointerOpacity = interpolate(
  frame,
  [p(8), p(20), tapBAt + p(15), tapBAt + p(28)],
  [0, 1, 1, 0],
  { extrapolateLeft: "clamp", extrapolateRight: "clamp" },
);

// x i y interpolują przez *jeden keyframe per stage*. Każda kolejna para
// keyframe'ów opisuje pojedynczy prosty segment (dowolny kierunek).
// Keyframe'y "hold" (target = poprzedni) parkują pointer podczas tap ripple
// zanim zacznie się kolejny ruch.
const pathFrames = [
  p(20),            // start glide-from-center
  tapAAt,           // dolot do A
  tapAAt + p(20),   // hold na A podczas ripple
  tapBAt,           // dolot do B (jeden prosty glide z A)
];
const pointerX = interpolate(
  frame,
  pathFrames,
  [centerX, A.x, A.x, B.x],
  { easing: Easing.bezier(0.16, 1, 0.3, 1), extrapolateLeft: "clamp", extrapolateRight: "clamp" },
);
const pointerY = interpolate(
  frame,
  pathFrames,
  [centerY, A.y, A.y, B.y],
  { easing: Easing.bezier(0.16, 1, 0.3, 1), extrapolateLeft: "clamp", extrapolateRight: "clamp" },
);

<Pointer x={pointerX} y={pointerY} opacity={pointerOpacity} />
<TapDot tapAt={tapAAt} x={A.x} y={A.y} />
<TapDot tapAt={tapBAt} x={B.x} y={B.y} />
```

**Kluczowe invarianty:**
- **Jeden** pointer instance, **jeden** fade-in w centrum, **jeden** fade-out po ostatnim tapie. Nigdy nie fade-out + back-in między tapami na tym samym UI.
- Każdy segment między kolejnymi keyframe'ami to pojedyncza prosta linia (interpolacja x i y linearnie między tą samą parą klatek). Kierunek dowolny — pionowy, poziomy, diagonal wszystkie OK.
- Używaj par `[..., A, A, ...]` "hold" żeby parkować pointer podczas tap ripple zanim zacznie się kolejny ruch. Bez holdu pointer by dryftował w środku tapu.
- Diagonale są **explicitly dozwolone** — zakazane są multi-segment paths w jednym ruchu (jeden ruch, jedna prosta linia) lub fade-out + back-in między tapami na tym samym UI.

## Kiedy resetować cursor (inne UI / nowy ekran)

Jeśli kolejna interakcja ląduje na *innym* UI (nowy ekran, inny formularz, kolejny beat) — wtedy i tylko wtedy pointer się resetuje. Fade-out obecnego pointera, render nowego z własnym fresh fade-in w centrum, glide do nowego targetu. Reset to wizualny sygnał że viewer jest gdzieś indziej.

## Częste błędy

| Błąd | Fix |
|------|-----|
| Render tylko `TapDot` więc tap pojawia się znikąd | Dodaj `Pointer` którego coords interpolują do targetu. Cursor *musi* być widoczny przed tapem. |
| Pointer znika w momencie tapu | Trzymaj Pointer widoczny ~10 klatek *po* `tapAt` żeby oko zobaczyło kontakt, potem fade-out. |
| Pointer w innym coord space niż TapDot | Oba muszą mieszkać w tym samym `position: relative` parent i dzielić `x`/`y`. |
| Dodanie Pointera do czysto wizualnego beata | Użyj `GlowRing`. Cursor implikuje user input. |
| Pojedynczy keyframe (prosta linia start → target) | Użyj 2-3 keyframe'ów żeby ścieżka się naturalnie naginała — czyste linie czują się mechaniczne. |

## Kiedy NIE używać

- Beaty highlightujące feature bez tapu (glow rings, animowany state, lądowanie rezultatu): użyj `GlowRing` albo bez indykatora.
- Beaty pokazujące pasywne przejście między ekranami bez implikowanej akcji usera.
- Pierwszy lub ostatni beat wideo który ustanawia kontekst albo pokazuje final result.

W tych przypadkach **nie** wczytuj tego pliku i nie dodawaj cursora — motion sam niesie wzrok.
