# @paperjsx/templates

Document templates for invoices, reports, and contracts usually live in proprietary SaaS dashboards or as untyped JSON blobs with no validation. This package provides 13 domain-specific templates as Zod schemas with TypeScript types, validation functions, and realistic sample data — ready to feed into `@paperjsx/json-to-pptx` locally, `@paperjsx/json-to-docx`, or the hosted `protocol_v2` runtime.

[![npm](https://img.shields.io/npm/v/@paperjsx/templates)](https://www.npmjs.com/package/@paperjsx/templates)
[![CI](https://github.com/paperjsx/templates/actions/workflows/ci.yml/badge.svg)](https://github.com/paperjsx/templates/actions/workflows/ci.yml)

## Install

```bash
npm install @paperjsx/templates
```

Requires Node.js >= 18. Zero runtime dependencies beyond `zod`, `qrcode`, and `bwip-js`.

## Quick Start

```typescript
import {
  sampleInvoiceData,
  validateInvoiceData,
  InvoiceDataSchema,
  type InvoiceData,
} from "@paperjsx/templates";

// 1. Use sample data to test your pipeline
console.log(sampleInvoiceData);
// { company: { name: "...", ... }, customer: { ... }, items: [...], ... }

// 2. Validate your own data
const result = validateInvoiceData(myData);
// Throws ZodError if invalid

// 3. Get the TypeScript type
const invoice: InvoiceData = {
  company: { name: "Acme Corp", address: "123 Main St" },
  customer: { name: "TechCorp", address: "456 Oak Ave" },
  items: [
    { description: "Enterprise License", quantity: 1, unitPrice: 12000 },
    { description: "Support Package", quantity: 1, unitPrice: 2400 },
  ],
  subtotal: 14400,
  taxRate: 0.08875,
  tax: 1278,
  total: 15678,
  invoiceNumber: "INV-2026-001",
  date: "2026-03-15",
  dueDate: "2026-04-14",
};
```

## Templates

| Template | Export Prefix | Category | Key Fields |
| --- | --- | --- | --- |
| SaaS Invoice | `InvoiceData` | Financial | company, customer, items, tax, total, currency |
| Medical Lab Report | `LabReportData` | Healthcare | patient, physician, lab, tests (with normal/high/low/critical flags) |
| Bill of Lading | `BillOfLadingData` | Logistics | shipper, consignee, cargo, freight details, hazmat |
| Analytics Report | `AnalyticsData` | Reporting | metrics, charts, time range, data sections |
| Legal Contract | `LegalContractData` | Legal | parties, clauses (numbered), signatures, effective date |
| Event Ticket | `EventTicketData` | Entertainment | event, venue, seat, attendee, QR code, barcode |
| Order Confirmation | `OrderConfirmationData` | E-commerce | order, items, shipping, tracking |
| Startup Pitch Deck | `PitchDeckData` | Presentation | problem, solution, traction, market (TAM/SAM/SOM), team, ask |
| Quarterly Review | `QuarterlyReviewData` | Presentation | KPIs, revenue, initiatives, goals, highlights |
| Project Status | `ProjectStatusData` | Presentation | milestones, risks, timeline, budget, action items |
| Meeting Minutes | `MeetingMinutesData` | Business | attendees, agenda, decisions, action items, next meeting |
| Employee Handbook | `EmployeeHandbookData` | HR | policies, benefits, workplace guidelines, acknowledgment |
| Technical Proposal | `TechnicalProposalData` | Business | scope, approach, timeline (phases), pricing, deliverables |

## Per-Template Exports

Every template exports 4 items following the same naming pattern:

```typescript
// Schema (Zod)
import { InvoiceDataSchema } from "@paperjsx/templates";

// Sample data (realistic fixture)
import { sampleInvoiceData } from "@paperjsx/templates";

// Validator function (throws on invalid input)
import { validateInvoiceData } from "@paperjsx/templates";

// TypeScript type
import { type InvoiceData } from "@paperjsx/templates";
```

Replace `Invoice` with any template name: `LabReport`, `BillOfLading`, `Analytics`, `LegalContract`, `EventTicket`, `OrderConfirmation`, `PitchDeck`, `QuarterlyReview`, `ProjectStatus`, `MeetingMinutes`, `EmployeeHandbook`, `TechnicalProposal`.

## Utilities

### QR Code Generation

```typescript
import {
  generateQRCodeDataURL,  // async: string -> PNG data URL
  generateQRCodeSVG,      // async: string -> SVG data URL
  generateQRCodeSync,     // sync (pre-cached)
} from "@paperjsx/templates";

const qr = await generateQRCodeDataURL("https://example.com/ticket/ABC123");
// "data:image/png;base64,iVBORw0KGgo..."
```

### Barcode Generation

```typescript
import {
  generateBarcode128,  // CODE128 (general purpose)
  generateBarcode39,   // CODE39 (industrial)
  generateBarcodeITF,  // Interleaved 2 of 5 (logistics)
} from "@paperjsx/templates";

const barcode = await generateBarcode128("INV-2026-001");
// SVG data URL
```

### i18n Formatting

```typescript
import {
  formatCurrency,
  formatDate,
  formatNumber,
  formatPercent,
  formatDateRange,
  createFormatter,
} from "@paperjsx/templates";

formatCurrency(1234.56, "EUR", "de-DE");     // "1.234,56 €"
formatCurrency(1234.56, "JPY", "ja-JP");     // "￥1,235"
formatDate("2026-03-15", "en-US");           // "March 15, 2026"
formatPercent(0.342, "en-US");               // "34.2%"
formatDateRange("2026-01-01", "2026-03-31", "en-US"); // "Jan 1 – Mar 31, 2026"

// Reusable formatter for a specific locale
const fmt = createFormatter("de-DE", "EUR");
fmt.currency(99.99);  // "99,99 €"
fmt.date("2026-03-15"); // "15. März 2026"
fmt.number(1234567);    // "1.234.567"
```

**Supported locales:** `en-US`, `en-GB`, `de-DE`, `fr-FR`, `ja-JP`, `zh-CN`, `pt-BR`, `es-ES`, `it-IT`

**Supported currencies:** `USD`, `EUR`, `GBP`, `JPY`, `CNY`, `INR`, `BRL`, `CAD`, `AUD`, `CHF` (plus any valid ISO 4217 code)

## Template Registry

Browse and filter templates at runtime:

```typescript
import { templateRegistry } from "@paperjsx/templates";

// List all templates
templateRegistry.forEach((t) => {
  console.log(`${t.id}: ${t.name} (${t.category})`);
  console.log(`  Tags: ${t.tags.join(", ")}`);
  console.log(`  Features: ${t.features.join(", ")}`);
});

// Filter by category
const financialTemplates = templateRegistry.filter(
  (t) => t.category === "Financial"
);
```

Each registry entry includes: `id`, `name`, `category`, `difficulty` (beginner/intermediate/advanced), `description`, `tags[]`, `features[]`.

## Examples

### Validate and Render an Invoice

```typescript
import { validateInvoiceData, sampleInvoiceData } from "@paperjsx/templates";
import { renderToDocx } from "@paperjsx/json-to-docx";

// Validate input data
const data = validateInvoiceData(myInvoiceData);

// Pass to rendering engine
const result = await renderToDocx({
  type: "DocxDocument",
  template: "invoice",
  data,
});
```

### Generate a Pitch Deck with Sample Data

```typescript
import { samplePitchDeckData } from "@paperjsx/templates";

// samplePitchDeckData includes realistic startup data:
// company name, problem, solution, TAM/SAM/SOM, traction metrics, team bios, funding ask
console.log(samplePitchDeckData.company); // "NeuralFlow"
console.log(samplePitchDeckData.market.tam); // "$48B"

// Use the sample data with @paperjsx/json-to-pptx locally
// or map it into the hosted protocol_v2 payload.
```

### QR Code for Event Tickets

```typescript
import {
  sampleEventTicketData,
  generateQRCodeDataURL,
  generateBarcode128,
} from "@paperjsx/templates";

const qr = await generateQRCodeDataURL(
  `https://tickets.example.com/${sampleEventTicketData.ticketId}`
);

const barcode = await generateBarcode128(sampleEventTicketData.ticketId);

// Use in document generation as image src
```

### Multi-Locale Currency Formatting

```typescript
import { createFormatter, LOCALES, CURRENCIES } from "@paperjsx/templates";

const locales = ["en-US", "de-DE", "ja-JP", "pt-BR"];

for (const locale of locales) {
  const fmt = createFormatter(locale, "USD");
  console.log(`${locale}: ${fmt.currency(2499.99)}`);
}
// en-US: $2,499.99
// de-DE: 2.499,99 $
// ja-JP: $2,499.99
// pt-BR: US$ 2.499,99
```

## Limitations

- Templates provide schemas and sample data only — they do not render documents themselves. Pair with `@paperjsx/json-to-pptx` or `@paperjsx/json-to-docx` for local rendering, or send equivalent structured data to the hosted protocol_v2 runtime.
- QR code and barcode generation is async (except `generateQRCodeSync` for pre-cached values).
- i18n formatting uses the `Intl` API. Locale support depends on the Node.js ICU data available in your runtime.

## Documentation

[paperjsx.com/docs/templates](https://paperjsx.com/docs/templates)

## License

Apache-2.0. See `LICENSE`.
