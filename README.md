# Lichtgewicht NLDS-thema's — een merkessentie-laag

> Een voorstel: hoe je een eigen-componenten-app lichtgewicht thematiseerbaar maakt
> én interoperabel houdt met de officiële NL Design System-thema's van andere overheden.

## Het probleem

NLDS-thema's worden geleverd als design-tokens (JSON → CSS-variabelen). Dat is krachtig,
maar de gepubliceerde sets zijn enorm: de `voorbeeld`-tokens bevatten **~2.500 waarden**
en de gecompileerde CSS mengt al **907 `--utrecht-*`, 611 `--basis-*`, 358 `--nl-*`,
241 `--denhaag-*`, 238 `--ams-*`** variabelen dooreen. Wie geen NLDS-componenten gebruikt
maar zélf componenten bouwt, wil dat niet als theming-contract.

## Het inzicht: er zijn drie lagen, geen één

| Laag | Voorbeeld | Aantal | Voor wie |
| --- | --- | --- | --- |
| **Component-tokens** | `--utrecht-button-…`, `--denhaag-…` | ~1.400 | Alleen wie hún componenten gebruikt |
| **Basis (semantisch)** | `--basis-color-action-1-bg-default` | 611 | Iedereen — merk-agnostisch |
| **Merkessentie** | `--brand-primary` | ~14 | Wat een gemeente wil invullen |

De component-laag mag je **volledig negeren** als je eigen componenten bouwt. De **Basis-laag**
is de bestaande, merk-agnostische semantische laag — je interop-ankerpunt. Wat *niet* als nette
losse laag bestaat is de **merkessentie**: een handvol kleuren + font + radius. Die laag voegt
dit voorstel toe — als dunne adapter bóvenop Basis, niet als geïsoleerd eiland.

## Hoe het werkt

```
app-contract.css        ← levert je app mee. Je componenten lezen alleen --color-*/--font-*.
                          Elke var leidt af uit: --brand-* → --basis-* → hardcoded default.
themes/<gemeente>.css   ← optioneel. De gemeente zet --brand-* (licht) óf levert een
                          volledig NLDS Basis-thema dat --basis-* zet. Beide werken.
```

Fallback-keten per variabele:

```css
--color-primary: var(--brand-primary,                 /* 1. lichtgewicht override   */
                 var(--basis-color-action-1-bg-default,/* 2. officieel NLDS-thema     */
                 #154273));                            /* 3. veilige standaard        */
```

Drie scenario's, één app:

1. **Gemeente wil simpel** → schrijft een bestand van ~6 regels (`themes/utrecht.css`).
2. **Gemeente investeert in NLDS** → dropt haar officiële Basis-thema; laag 2 pakt het op.
3. **Geen thema** → de app draait op de hardcoded standaarden.

## Gebruik

```html
<link rel="stylesheet" href="app-contract.css" />
<link rel="stylesheet" href="themes/utrecht.css" /> <!-- of een NLDS Basis-thema -->
```

In je componenten:

```css
.button { background: var(--color-primary); color: var(--color-on-primary); border-radius: var(--border-radius); }
.button:hover { background: var(--color-primary-dark); }
a { color: var(--color-secondary); }
:focus-visible { outline: 2px solid var(--color-focus); background: var(--color-focus-bg); }
```

Open **`index.html`** om alle drie de scenario's naast elkaar te zien.

## De `--basis-*`-namen die dit gebruikt

Gemeten uit `@nl-design-system-unstable/voorbeeld-design-tokens` (v10):

| App-contract | NLDS Basis-token |
| --- | --- |
| `--color-primary` | `--basis-color-action-1-bg-default` |
| `--color-primary-dark` | `--basis-color-action-1-bg-hover` |
| `--color-on-primary` | `--basis-color-action-1-color-default` |
| `--color-secondary` | `--basis-color-action-2-bg-default` |
| `--color-secondary-dark` | `--basis-color-action-2-bg-hover` |
| `--color-text` | `--basis-color-default-color-default` |
| `--color-bg` | `--basis-color-default-bg-default` |
| `--color-border` | `--basis-color-default-border-default` |
| `--color-focus` | `--basis-focus-color` |
| `--color-focus-bg` | `--basis-focus-background-color` |
| `--font-body` | `--basis-text-font-family-default` |
| `--font-heading` | `--basis-heading-font-family` |
| `--border-radius` | `--basis-border-radius-md` |

## Eerlijke kanttekeningen

- **`--basis-*` zit in `@nl-design-system-unstable/*`** — letterlijk *unstable*. De interop-belofte
  bouwt op namen die kunnen schuiven. Daarom isoleert dit voorstel ze in **één bestand**
  (`app-contract.css`): verandert een naam, dan pas je één regel aan, niet je hele app.
- **De semantische koppeling is een keuze, geen wet.** Of "secundair/links" nu `action-2`,
  `accent-1` of iets anders is, hangt af van hoe een gemeente Basis invult. `app-contract.css`
  is de plek om dat per project bij te stellen.
- **Je geeft de getoetste toegankelijkheid van de NLDS-componenten op.** De échte prijs van
  "geen externe componenten" zit niet in de tokens, maar in focus-states, toetsenbordgedrag,
  `forced-colors` en ARIA die je nu zelf moet borgen.
- **Merk ≠ alleen kleur.** Kleur + font + radius dekt ~80% van de herkenbaarheid; spacing en
  typografische schaal (die Basis óók biedt) vallen hier bewust buiten, om het licht te houden.
