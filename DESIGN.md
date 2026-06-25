---
version: alpha
name: ZeroDrop
description: >
  Master payment gateway aggregator for Israeli eCommerce.
  Single-contract, multi-gateway failover routing with automated insurance.
  The visual language is clean, technical, and trustworthy — built around
  the idea of a single drop of clarity in a complex payment pipeline.

colors:
  background: "#FFFFFF"
  surface: "#F8FAFC"
  surface-elevated: "#FFFFFF"
  primary: "#0F172A"
  secondary: "#0EA5E9"
  accent: "#22D3EE"
  text: "#0F172A"
  text-muted: "#64748B"
  text-on-primary: "#FFFFFF"
  border: "#E2E8F0"
  border-subtle: "#F1F5F9"
  success: "#22C55E"
  warning: "#F59E0B"
  danger: "#EF4444"
  info: "#3B82F6"

typography:
  display:
    fontFamily: "Inter"
    fontSize: "3rem"
    fontWeight: 700
    lineHeight: 1.1
    letterSpacing: "-0.03em"
  h1:
    fontFamily: "Inter"
    fontSize: "2.25rem"
    fontWeight: 700
    lineHeight: 1.15
    letterSpacing: "-0.02em"
  h2:
    fontFamily: "Inter"
    fontSize: "1.75rem"
    fontWeight: 600
    lineHeight: 1.2
    letterSpacing: "-0.02em"
  h3:
    fontFamily: "Inter"
    fontSize: "1.25rem"
    fontWeight: 600
    lineHeight: 1.3
    letterSpacing: "-0.01em"
  body:
    fontFamily: "Inter"
    fontSize: "1rem"
    fontWeight: 400
    lineHeight: 1.6
    letterSpacing: "0"
  body-sm:
    fontFamily: "Inter"
    fontSize: "0.875rem"
    fontWeight: 400
    lineHeight: 1.5
    letterSpacing: "0"
  label:
    fontFamily: "JetBrains Mono"
    fontSize: "0.75rem"
    fontWeight: 600
    lineHeight: 1
    letterSpacing: "0.08em"
    fontFeature: "'case' on, 'calt' on"

rounded:
  sm: "4px"
  md: "8px"
  lg: "12px"
  xl: "16px"
  pill: "999px"

spacing:
  xs: "4px"
  sm: "8px"
  md: "16px"
  lg: "24px"
  xl: "32px"
  2xl: "48px"
  3xl: "64px"

components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.text-on-primary}"
    typography: "{typography.label}"
    rounded: "{rounded.md}"
    padding: "10px 18px"
  button-primary-hover:
    backgroundColor: "#1E293B"
  button-secondary:
    backgroundColor: "{colors.surface-elevated}"
    textColor: "{colors.text}"
    rounded: "{rounded.md}"
    padding: "10px 18px"
  button-secondary-hover:
    borderColor: "{colors.secondary}"
    textColor: "{colors.secondary}"
  card:
    backgroundColor: "{colors.surface}"
    borderColor: "{colors.border}"
    rounded: "{rounded.lg}"
    padding: "{spacing.lg}"
  input:
    backgroundColor: "{colors.surface-elevated}"
    borderColor: "{colors.border}"
    textColor: "{colors.text}"
    rounded: "{rounded.md}"
    padding: "10px 14px"
  input-focus:
    borderColor: "{colors.secondary}"
  tag:
    backgroundColor: "{colors.border-subtle}"
    textColor: "{colors.text-muted}"
    typography: "{typography.label}"
    rounded: "{rounded.pill}"
    padding: "4px 10px"
  badge-success:
    backgroundColor: "#F0FDF4"
    textColor: "#15803D"
    rounded: "{rounded.pill}"
  badge-warning:
    backgroundColor: "#FFFBEB"
    textColor: "#B45309"
    rounded: "{rounded.pill}"
  badge-danger:
    backgroundColor: "#FEF2F2"
    textColor: "#B91C1C"
    rounded: "{rounded.pill}"
---

## Overview

ZeroDrop is a master payment gateway aggregator. The interface should feel exact,
fast, and calm under complexity — routing billions in transactions while making
every routing decision visible and traceable.

- **Visual style:** minimal, technical, fluid
- **Color stance:** high-contrast neutrals with a single cyan "drop" accent
- **Design intent:** Reduce cognitive load. Use whitespace, clear hierarchy,
  and restrained color to guide operators through high-stakes payment operations.

## Colors

- **Primary (`#0F172A`):** Deep ink for headlines, primary buttons, and key UI chrome.
- **Secondary (`#0EA5E9`):** Sky blue for interactive accents, focus states, and links.
- **Accent (`#22D3EE`):** Cyan "drop" for highlights, live indicators, and data signals.
- **Background (`#FFFFFF`):** Clean white foundation.
- **Surface (`#F8FAFC`):** Slightly tinted gray for panels, cards, and sidebars.
- **Text (`#0F172A`):** Primary reading color on light surfaces.
- **Text-muted (`#64748B`):** Secondary text, labels, metadata.
- **Border (`#E2E8F0`):** Structural dividers and input borders.
- **Success / Warning / Danger:** Standard semantic greens, ambers, and reds used
  for status badges and alerts only.

## Typography

- **Primary family:** Inter — used for all headings, body, and UI labels.
- **Mono family:** JetBrains Mono — used for technical labels, tags, timestamps,
  and data-dense metrics.
- **Display:** Large, tight leading, negative letter-spacing for hero headlines.
- **Body:** Comfortable 1.6 line-height for readability in long-form content.
- **Labels:** Uppercase mono, wide letter-spacing for tags and section headers.

## Layout

- **Container max-width:** 1280px
- **Grid:** 12-column with 24px gutters
- **Section padding:** 64px vertical on desktop, 32px on mobile
- **Card spacing:** 24px internal padding, 16px gaps
- **Dense dashboards:** 16px gaps, 16px card padding

## Elevation & Depth

ZeroDrop uses flat, bordered surfaces. Shadows are reserved for overlays
and floating elements.

- **Card:** 1px border, no shadow by default.
- **Hover lift:** subtle `translateY(-1px)` on interactive cards.
- **Dropdown / modal:** `0 20px 40px rgba(15, 23, 42, 0.12)` shadow.
- **Focus ring:** 2px cyan outline offset by 2px.

## Shapes

- **Small radius (4px):** tags, small buttons, inputs
- **Medium radius (8px):** primary buttons, cards, panels
- **Large radius (12px):** modals, large cards
- **Pill radius (999px):** status badges, user avatars, filter chips

## Components

### Buttons

- **Primary:** dark ink background, white text, 8px radius. Hover darkens to `#1E293B`.
- **Secondary:** white background, gray border, dark text. Hover turns the border and text cyan.
- **Icons:** always 16–20px, centered with text.

### Cards

- Light surface background, 1px border, 12px radius.
- Optional 1px top or left accent border using cyan for feature cards.

### Inputs

- White or surface background, 1px border, 8px radius.
- Focus state uses cyan border and a subtle outer glow.
- Labels are mono uppercase with generous spacing.

### Badges

- Soft tinted backgrounds with matching text colors.
- Pill shape. Use only for status: success, warning, danger, info.

### Navigation

- Sidebar or top bar in surface color with subtle borders.
- Active item uses primary background with white text, or cyan left-border accent.

## Do's and Don'ts

**Do**

- Use whitespace to separate sections before adding borders.
- Reserve cyan for interactive signals, live states, and the occasional accent.
- Use JetBrains Mono for metrics, timestamps, tags, and technical labels.
- Keep button text concise and action-oriented.

**Don't**

- Use more than one accent color in the same view.
- Apply shadows to static cards.
- Use decorative gradients that do not serve data hierarchy.
- Mix rounded styles arbitrarily — pick a radius and stay consistent per component type.
