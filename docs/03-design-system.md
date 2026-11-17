# tinyphoto — Neo-Brutalism × Retro Analog

Version 1.0.0. contracts/design-tokens.json is the single source of truth. JSON is the project's explicit token format, not an implementation of the DTCG token specification. Values are complete and require no design service or paid fonts.

## Visual language

Use warm paper, near-black ink, hard rectangular panels, 3-unit borders and 6-unit unblurred offset shadows. Yellow marks the primary action, blue marks room codes, mint marks ready status, pink accents stamp labels. Photography is the focal point; decoration stays outside faces and controls. Rounded shapes are limited to the shutter button, avatar crop and physical stamp perforations. Do not simulate paper with large downloaded image textures. Use vector linework and a lightweight deterministic grain shader only if it does not reduce legibility or output performance.

The main palette is paper #FFF8E7, ink #181818, white #FFFFFF, muted #53504A, yellow #FFD447, pink #FF91AF, mint #A8E6CF and blue #BCD5FF. Error red #A61B29 and success green #1F633E communicate status alongside an icon and text. Primary buttons use ink on yellow; destructive buttons use white on dark red. Pastel colors are backgrounds, not small text colors. Focus uses #1749B5 with a 3-unit ring and 3-unit offset. Verify WCAG AA contrast ≥4.5:1 for body text and ≥3:1 for large text/UI boundaries in the actual composition; do not draw labels directly on arbitrary photos without a solid backing.

## Typography and scale

Package Be Vietnam Pro for display/body and IBM Plex Mono for codes/countdown with their license files in each app. Include Vietnamese glyph subsets and the required 400/700/800 sans and 600 mono weights; do not request Google Fonts at runtime. System sans-serif/monospace is a resilient fallback while packaged fonts load. Avoid faux bold. UI is Vietnamese first; use sentence case except brief analog labels such as LIVE or PHOTO NO. Keep accents intact.

| Role | Size / line height | Weight | Tracking |
| --- | --- | --- | --- |
| Display | 40 / 48 | 800 | -0.8 |
| Heading | 28 / 36 | 700 | -0.28 |
| Title | 20 / 28 | 700 | 0 |
| Body | 16 / 24 | 400 | 0 |
| Label | 14 / 20 | 700 | 0.28 |
| Caption | 12 / 18 | 400 | 0.12 |
| Room code | 32 / 40 | 600 mono | 4 |
| Countdown | 80 / 88 | 600 mono | 0 |

Base spacing is four units: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80 and 96. Web layout values map to CSS px; typography size, line height and tracking divide by 16 for rem. Android layout values map to dp and typography to sp; preserve system font scaling. Export raster geometry uses physical pixels and never device density or accessibility font scaling. A font can grow without shrinking a touch target or hiding action labels.

## Responsive layout and flow

At widths below 768, use a single column, 16-unit gutters, full-width actions and an 80-unit shutter. At 768+, use 32-unit gutters and an optional status side panel. At 1024+, a two-column camera/room layout fits within 1120 units; camera remains ≤640 and forms ≤480. Android chooses the corresponding available dp width, not the CSS/device-pixel width. Respect safe-area/system insets and keyboard resizing. Shadows and perforation cutouts reserve their own outside margin so they never clip at the viewport edge.

Home offers “Chụp một mình”, “Tạo phòng” and “Tham gia phòng”. Login explains camera privacy and seven-day memories before capture. Camera asks permission in context and shows clear denied, unavailable, busy and stopped states. A room lobby shows code as a single six-character text value, a copy button, 2–4 named member tiles and ready controls. Paste accepts exactly six ASCII digits after trimming outer whitespace; preserve leading zeros, do not treat the code as a number. Only host sees the start action enabled once everyone is ready. Copy uses accessible confirmation and copies only the room code, never a signed URL.

During countdown, put numerals inside a solid high-contrast badge over the viewfinder. The preview remains live; disable capture/room mutations that conflict with the current round. Group members see capture/upload progress such as “Đã nhận 6/8 ảnh”, followed by host composition progress. An upload error offers an in-RAM retry while the reservation is valid, otherwise “Chụp lại”; never imply an offline copy exists. Cold start displays “Đang kết nối máy ảnh nhóm…” and a bounded retry state before anyone becomes ready. The memories grid loads only metadata and fetches images near the viewport; release offscreen images under memory pressure. Expired items disappear with an explanatory empty state. Memory controls support deleting one's reference and re-shooting; the strict mode has no disk save or share-to-files action.

## Components and interactions

| Component | Geometry / states |
| --- | --- |
| BrutalButton | 52 high minimum, 16 horizontal padding, 3 border, 6×6 solid shadow; default, hover, pressed, focus-visible, loading, disabled |
| Press response | Translate 6×6 and remove shadow over 80ms; layout footprint stays fixed; keyboard and touch get the same feedback |
| Shutter | 80 diameter, 4 border, inner 56 circle, ≥48 target; label “Chụp ảnh”; disabled while capturing |
| Viewfinder | 4:3 sensor crop, 3 border, ink letterbox, LIVE label; mirrored preview with an explicit mirror toggle for final output |
| RoomCode | Blue paper panel, mono digits, copy action; one labeled input, inputmode=numeric on web |
| MemberTile | Name, ready icon + text, host badge; connection warning does not imply membership removal |
| StampFrame | 4:5 outer shape, perforation mask, inner photo crop, caption/postmark in safe inset |
| MemoryCard | Stamp or strip ratio preserved; expiry label; fetch/error/deleted states and accessible action menu |
| StatusBanner | Solid surface, icon, concise text, optional retry; no diagnosis based solely on color |
| Dialog | White paper, 4 border, 8×8 shadow, focus trap and return focus; destructive action clearly labeled |

Disabled controls use disabledSurface/disabledInk, no press translation, and an accessible disabled property. Loading keeps the button label plus a small indicator so width is stable. Error messages are inline near the failed operation; reserve toasts for noncritical confirmation. Focus order follows visual order. Android uses semantics/contentDescription and role/button; web uses native buttons and labeled inputs. All interactive targets are ≥48×48 units. Do not create a separate focusable button for each digit. Announce each countdown number once in a polite live region; do not repeat every animation frame. Respect system text size, reduced motion and high-contrast modes.

## Authentic stamp geometry

Stamp perforations are transparent semicircular cutouts from all four paper edges, not scalloped colored dots pasted on top. At UI scale, radius=4, pitch=16 and centers lie on the outer edge (inset=0). At export scale, radius=12 and pitch=48. For an edge length L and radius r, choose n=floor((L−2r)/pitch)+1 and start=(L−(n−1)×pitch)/2, then place centers at start+i×pitch for i=0..n−1. Mirror center placement on opposite edges; never clip an extra half-hole at a corner. Subtract these circles from the outer rectangle using an SVG mask/Canvas destination-out on web and a Path difference on Android. Keep inner art at safeInset=16 UI units / 48 export pixels. A two-unit inner border follows the safe rectangle. Draw shadow from the finished perforated silhouette, not its rectangular bounding box. The exported stamp keeps the cutouts transparent: encode it as AVIF with alpha or PNG, never JPEG, and never flatten the holes onto a background colour before upload.

UI stamp rotation defaults to −2° and never exceeds ±3°; exported images stay axis-aligned. Postmark uses ink at 0.65 opacity, 2-unit strokes, positioned in the caption/edge region away from faces. Physical shadow is UI chrome and is not baked into final photo output. Stamp/photo artwork is drawn into an in-RAM Canvas/Bitmap using the same layout formulas on both platforms. No SVG/raster download is required for the shape.

## Photo and strip composition

Aspect ratios are exact: stamp 4:5, camera/photo 4:3, group tile 4:3, strip 1:3 and avatar 1:1. Capture policy lives in the API/master plan; the duplicate capture values in tokens must be parity-checked by contract CI before release.

Export a stamp at 1095×1500, 1×3 strip at 1500×3285, 2×3 grid at 3000×3285, 2×2 grid at 3000×3000, and 1×4 / group strip at 1500×4380 (keeping each single photo unit ≤1500px on both axes, and total composite product within 6000×6000 / 24M pixels). The header contains the session label; the footer contains the creation date without location. Fonts are rasterized at explicit export sizes; date formatting follows the selected app locale and is resolved before composition.

Each group shot first becomes a 1500×1125 (4:3) tile. Frozen participants are ordered by joined_at, breaking ties by user UUID; this order stays fixed through all four shots. Two participants use two columns and one row. Three or four use two columns and two rows; an unused fourth cell contains paper plus “tinyphoto”, never another person's old image. Use 12px gaps between tile cells. Center-crop each source within its cell, then fit the resulting 4:3 tile into its corresponding strip panel. Draw each source into the tile sequentially and release its decoded bitmap. Strip composition never loads 16 decoded frames simultaneously.

Preserve original orientation using sensor rotation before cropping; preview mirroring defaults on and output mirroring defaults off. Warn/offer a toggle before capture rather than silently reversing readable text. Remove EXIF, GPS and original filenames. JPEG quality begins at 0.85; reduce quality to satisfy byte caps while keeping the fixed output geometry. If the minimum acceptable quality still exceeds the cap, fail with a clear retry action. PNG is reserved for stamp transparency and must still fit its byte cap. No EXIF restoration, disk thumbnail, permanent base64 string or canvas data URL in persisted state.

## Flash and reduced motion

Flash is a single white overlay: opacity rises 0→0.65 in 40ms then falls to 0 in 160ms. Trigger once per captured shot, four seconds apart; keep the overlay noninteractive and decorative. It never drives shutter timing. Reduced-motion preference disables the flash and countdown scaling; a static “Đã chụp” status replaces them. Provide an explicit flash-effects toggle because photosensitivity can exist without a system preference. Do not use full-screen strobing or vibration loops. Standard transitions are 160ms with cubic-bezier(0.2,0,0,1); Compose uses CubicBezierEasing(0.2f,0f,0f,1f).

## Tailwind adapter

Generate CSS custom properties from literal token values and a Tailwind theme mapping. Resolve semantic color names through color before emission. Example generated declarations:

```css
:root {
  --tp-paper: #fff8e7;
  --tp-ink: #181818;
  --tp-action-primary: #ffd447;
  --tp-border-standard: 3px;
  --tp-shadow-standard: 6px 6px 0 0 #181818;
  --tp-font-body: "Be Vietnam Pro", sans-serif;
  --tp-body-size: 1rem;
  --tp-body-leading: 1.5rem;
}
.tp-button {
  min-height: 52px;
  padding-inline: 16px;
  border: var(--tp-border-standard) solid var(--tp-ink);
  border-radius: 0;
  color: var(--tp-ink);
  background: var(--tp-action-primary);
  box-shadow: var(--tp-shadow-standard);
}
.tp-button:active:not(:disabled) {
  transform: translate(6px, 6px);
  box-shadow: none;
}
.tp-button:focus-visible {
  outline: 3px solid #1749b5;
  outline-offset: 3px;
}
```

For Tailwind 3 extend theme.colors, spacing, borderWidth, boxShadow, fontFamily and aspectRatio with tp-prefixed keys. For Tailwind 4 emit corresponding @theme variables; do not copy a v3 JavaScript config into a v4 setup without adapting it. Consumers pin their chosen major. Be explicit with arbitrary aspect ratios when plugin support differs: aspect-[4/5] and aspect-[1/3]. Generate pressed/focus state CSS from the same numbers. No hex literals in screen components.

## Compose adapter

Generator converts #RRGGBB to Color(0xFFRRGGBB) and #RRGGBBAA to Color(0xAARRGGBB); for example scrim #181818B3 becomes Color(0xB3181818). Map body size=16.sp, lineHeight=24.sp and letterSpacing=0.sp; borders/spacing are Dp. Map typography weights explicitly, including 800 display and 600 mono. Use RoundedCornerShape(0.dp) for square controls and a circle for shutter. Material elevation cannot reproduce a hard solid shadow: draw the offset silhouette behind the component in a custom modifier/Canvas, reserving external space. Press moves by the token shadow offset and removes the silhouette. A perforated stamp uses the same geometric Path for clipping, border and shadow. Compose aspectRatio receives width/height as Float, not the inverse. Do not use dp for output Bitmap dimensions.

## Acceptance checklist

Verify web/Android at compact and wide widths, Vietnamese long labels, 200% font scaling, keyboard navigation, TalkBack/screen reader, reduced motion, denied camera, host/member states, poor network, expired URL and empty memories. Compare exported stamp/strip golden images for crop, participant order, perforation spacing and panel geometry with tolerance for font rasterization. Check no image writes with actual camera use, not only mock API responses. This document defines the target; visual QA and implementation remain future work.
