# BlitztextMac — Domain Context

## Transkriptions-Backend

**CloudTranscriptionRouter** — Logik in `TranscriptionService`, die Groq-first mit OpenAI-Fallback kombiniert. Kein eigener Typ, aber konzeptuell der Router.

**Groq-Kontingent** — Tägliches Free-Tier-Budget bei Groq (Audio-Sekunden). Kommt aus HTTP-Response-Headern `x-ratelimit-remaining-audio-seconds` und `x-ratelimit-reset-audio`. Wird persistent in `UserDefaults` via `GroqQuotaStore` gespeichert.

**Groq-Fallback** — Zustand, in dem der Router nach Groq-429 dauerhaft auf OpenAI umschaltet. Bleibt aktiv bis `rateLimitResetAt` überschritten ist (auch nach App-Neustart). Gespeichert in `GroqQuotaStore.shared`.

**Paid Mode** — Wenn Online-Transkription aktiv UND (kein Groq-Key konfiguriert ODER Groq-Fallback aktiv). Wird im Menüleisten-Icon als kleiner Punkt angezeigt.

## ADR-0001: Groq-Fallback ist persistent, nicht session-basiert

**Kontext:** Quota wird täglich zurückgesetzt. Bei App-Neustart während laufendem Training würde ein session-basierter Fallback erneut Groq versuchen, obwohl das Kontingent noch nicht zurückgesetzt ist.

**Entscheidung:** `GroqQuotaStore` persistiert `fallbackActive` + `rateLimitResetAt` in `UserDefaults`. Beim App-Start prüft `clearIfExpired()` ob das Reset-Datum überschritten ist.

**Konsequenz:** Ein paar Anfragen die direkt nach Reset kommen könnten theoretisch nochmal 429 bekommen — kein Problem, der Fallback aktiviert sich einfach erneut bis zum nächsten Reset.
