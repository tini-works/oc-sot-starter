## Tools

### Shell Commands
- git: read logs, check structure
- rg (ripgrep): fast codebase search for patterns, imports, exports
- fd: fast file finder by name/extension
- find: recursive file discovery with filters
- cat/head/tail: read file contents
- wc: count files, lines, characters
- grep/sed/sort/uniq: text processing

### Package Managers
- bun: /Users/hiep/.bun/bin/bun (install, build, run)
- npm: /opt/homebrew/bin/npm (install, build, run)

### Analysis Tools
- c3x: /Users/hiep/Projects/agentee/c3-skill/skills/c3/bin/c3x.sh
  - c3x check: validate .c3/ docs against schema
  - c3x list: show topology of .c3/ docs
  - c3x lookup <file>: map file to component

### Working Directory
- Source projects are read from local paths (never modified)
- Write generated artifacts to $OUTPUT_DIR (set in Phase 2 of AGENTS.md)

### Analysis Patterns
- Trace imports: rg "^import|^from" -g '*.ts' -g '*.tsx' -l
- Find components: rg "export (default |)function|export (default |)const" -g '*.tsx' -g '*.jsx' -g '*.vue' -g '*.svelte'
- Find routes: rg "Route|path:|createBrowserRouter|useRouter" -g '*.ts' -g '*.tsx' -g '*.js' -g '*.jsx'
- Find theme: rg "theme|colors|palette|--[a-z]+-" -g '*.css' -g '*.scss'
- Find config: fd "tailwind.config|theme|tokens|\.prev" --type f
- Count files: find . -type f -name "*.tsx" | wc -l

### SFT Output Format

The SFT (Screens, Flows, Transitions) YAML schema for spec.sft.yaml:

```
app:
  name, description
  regions: [{ name, description, tags?, events? }]
  screens:
    [{ name, description, route, file, tags?,
       regions: [{ name, description, file, tags?, events? }],
       states?: [{ on, from?, action: "navigate(Screen)" }] }]
  flows: [{ name, description?, on?, sequence }]
```

Naming conventions:
- Screens: PascalCase from route segment (/payments → Payments)
- Regions: PascalCase from component name
- Events: kebab-case from interaction handlers (select-payment, submit-form)
- Flows: PascalCase (EntityFromSource or VerbEntity)

Arrow notation for flow sequences:
- ScreenName → navigate to screen
- [Back] → back navigation
- Screen(H) → history re-entry (restore scroll/selection)
- Step{data} → data annotation

### Framework Detection

Use this table to adapt screen/route/navigation searches per framework:

| Framework      | Screen files              | Route patterns                          | Navigation patterns                  |
|----------------|---------------------------|-----------------------------------------|--------------------------------------|
| React Router   | pages/, routes/           | createBrowserRouter, Route, path:       | useNavigate, navigate(), Link        |
| Next.js App    | app/**/page.tsx           | directory-based routing                 | useRouter, router.push, Link         |
| Next.js Pages  | pages/**/*.tsx            | file-based routing                      | useRouter, router.push, Link         |
| Vue Router     | views/, pages/            | routes array, path:                     | useRouter, router.push, RouterLink   |
| Angular        | **/component.ts           | path:.*component:                       | Router.navigate, routerLink          |
| SvelteKit      | routes/**/+page           | directory-based routing                 | goto(), a href                       |
| Go (Echo/Gin)  | handlers/, templates/     | e.GET, r.GET, HandleFunc               | http.Redirect, c.Redirect            |
| Django         | views.py, templates/      | urlpatterns, path()                     | redirect(), HttpResponseRedirect     |
| Flask/FastAPI  | routes/, views/           | @app.route, @router.get                | redirect(), RedirectResponse         |
| Rails          | views/, controllers/      | resources, get/post in routes.rb        | redirect_to, link_to                 |
| Generic        | find template/view files  | search for URL path definitions         | search for redirect/navigation calls |

For unknown frameworks, search generically for URL path definitions and redirect/navigation calls.
