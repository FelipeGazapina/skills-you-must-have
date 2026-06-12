---
name: design
description: Implement visual/UI changes in the codebase (views, partials, Stimulus controllers, Tailwind CSS). Optionally uses Pencil MCP for mockups.
argument-hint: "[what to design or change]"
---

# Design

Implement visual and UI changes directly in the project code (ERB views, partials, layouts, Stimulus controllers, Tailwind CSS). Uses Pencil MCP tools when a `.pen` mockup is available or needs to be created.

## Arguments

$ARGUMENTS — What to design or change (e.g., "criar tela de login", "trocar os botões por um dropdown de ações", "adicionar modal de confirmação", "redesenhar o header do dashboard")

## Rules

- This is a **Rails 8 project** using **Hotwire (Turbo + Stimulus)**, **Tailwind CSS 4.2**, **Alpine.js** (for dropdowns/toggles), and **Importmap** (no npm/yarn).
- Follow existing codebase conventions:
  - **Theme**: Orange accent (`bg-orange-600`), gray neutrals, red for destructive actions
  - **Language**: pt-BR for all user-facing labels
  - **Dropdowns**: Alpine.js pattern (`x-data="{ open: false }"`, `@click`, `x-show`, `@click.away`)
  - **Buttons**: `button_to` with `turbo_confirm` for destructive actions; `link_to` with `data: { turbo_method: }` inside dropdowns
  - **Icons**: Heroicons SVG inline (`w-4 h-4`)
  - **Layout**: `layout "dashboard"` for all authenticated pages
  - **Partials**: Extract to `_partial.html.erb` when a block is reused across views
- **Pencil MCP** (`.pen` files): Use only when explicitly asked for mockups or when a `.pen` file exists. Contents are encrypted — NEVER use Read/Grep on `.pen` files, only Pencil MCP tools.

## Steps

### 1. Understand the Request

If $ARGUMENTS is empty, use AskUserQuestion to ask what to design before proceeding.

Restate what is being designed or changed in 1-2 sentences. If the scope is ambiguous, use AskUserQuestion to clarify.

### 2. Gather Context

- Use Glob and Grep to locate the relevant views, partials, layouts, and Stimulus controllers
- Read the affected views in full to understand current structure
- Check `config/routes.rb` to understand which controller actions serve the page
- Read the controller to understand what data is available (`@instance_variables`)
- Check `app/javascript/controllers/` for related Stimulus controllers
- Check `app/views/layouts/dashboard.html.erb` for layout-level patterns (nav, Alpine.js dropdowns)
- Look at similar pages in the project to find UI patterns to follow (e.g., how other pages handle dropdowns, modals, tables)
- If a `.pen` mockup is referenced, use `get_editor_state`, `batch_get`, and `get_screenshot` from Pencil MCP to understand the design intent

### 3. Implement

Make changes directly in the codebase:

- **Views** (`app/views/**/*.html.erb`): Edit ERB templates with Tailwind classes, Alpine.js directives, Turbo attributes
- **Partials**: Extract shared UI blocks to `_partial.html.erb` when reused
- **Stimulus controllers** (`app/javascript/controllers/`): Create or modify when interactive behavior is needed beyond Alpine.js
- **Importmap** (`config/importmap.rb`): Pin new JS dependencies if needed
- Follow existing patterns:
  - Dropdowns: Alpine.js `x-data`, `x-show`, `@click.away` with transitions (see `app/views/layouts/dashboard.html.erb`)
  - Modals: `fixed inset-0 bg-gray-600 bg-opacity-50 hidden z-50` pattern (see existing modals in views)
  - Tables: `min-w-full divide-y divide-gray-200` with hover states
  - Forms: `focus:ring-2 focus:ring-orange-500 focus:border-orange-500` pattern
  - Badges/Status: `inline-flex px-3 py-1 text-sm font-semibold rounded-full` with color-coded backgrounds

### 4. Validate

- Run `bundle exec rubocop` on changed Ruby files to check for style issues
- Verify routes still work: `bin/rails routes | grep <resource>`
- If a `.pen` file was created/updated, use `get_screenshot` to visually validate

### 5. Summary

Produce a brief summary:

**Changes made**: List of files changed with one-line description of each change.

**Design decisions**: Notable choices (layout, colors, interaction pattern) and why, referencing which existing page/pattern was followed.

**Key files**:

| File | Role |
|------|------|
| `path/to/file.html.erb` | What was changed in this file |
