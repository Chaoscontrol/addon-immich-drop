## v0.5.0-beta - 2025-10-03

Upstream release: https://github.com/Nasogaa/immich-drop/releases/tag/v0.5.0-beta

<html>
<body>
<!--StartFragment--><html><head></head><body><h1>v0.5.0 –&nbsp;Manage Links (2025‑10‑03)</h1>
<blockquote>
<p><strong>Status:</strong> Stable · <strong>Release type:</strong> Feature &amp; maintenance</p>
</blockquote>
<hr>
<h2>✨&nbsp;Highlights</h2>

Category | Item
-- | --
New | • Manage Links footer keeps Delete/Enable/Disable inside the panel
• Icon‑only row actions (Open, Copy, Details, QR, Save) with pop‑up tooltips |  
• Save button change‑detection – greyed out until a row is edited |  
• Per‑row QR modal; Details modal close is now reliable |  
• Auto‑refresh after creating an invite; new row smooth‑scrolls & briefly highlights |  
Fixed | • Expiry date is stored at end‑of‑day → no more “minus one day”
• Off‑by‑one close button selector in Details modal |  
Repo Hygiene | • Added .gitignore entries for .env, data/, state.db, build & IDE artifacts


<h2>🔒&nbsp;Security / Hardening</h2>
<ul>
<li>
<p><strong>Production tip:</strong> set <code inline="">SESSION_SECRET</code>, restrict CORS to your domain, serve over HTTPS.</p>
</li>
</ul>
<h2>♻️&nbsp;Other Improvements</h2>
<ul>
<li>
<p>README simplified; new <em>What’s&nbsp;New</em> section.</p>
</li>
<li>
<p>Road‑mapped: per‑user UI &amp; user‑provided Immich API tokens (removes fixed key dependency).</p>
</li>
</ul>
<h2>🛠&nbsp;Upgrade Notes</h2>
<ol>
<li>
<p><strong>Pull</strong> latest <code inline="">main</code>.</p>
</li>
<li>
<p>Copy <code inline="">.env.example</code> → <code inline="">.env</code> &amp; set real <code inline="">SESSION_SECRET</code>, <code inline="">IMMICH_API_KEY</code>.</p>
</li>
<li>
<p>Rebuild / restart (<code inline="">docker compose up -d --build</code> or restart service).</p>
</li>
<li>
<p>Clear browser cache to load the new Manage UI.</p>
</li>
</ol>
<p><em>No schema changes; database unchanged.</em></p>
<h2>🚧&nbsp;Breaking Changes</h2>
<ul>
<li>
<p>Manage Links markup/classes changed. Custom themes or browser user‑styles may need an update.</p>
</li>
</ul>
<hr>
<p>Enjoy the streamlined Manage experience!  Feedback &amp; PRs welcome.</p></body></html><!--EndFragment-->
</body>
</html>

## v0.4.1-beta - 2025-09-22

Upstream release: https://github.com/Nasogaa/immich-drop/releases/tags/v0.4.1-beta

No upstream release notes were found for v0.4.1-beta. See the tag for details.

# Changelog — Immich Drop Home Assistant Add-on

This file is maintained automatically by CI when upstream releases are synced.
Manual entries are allowed; CI will prepend new upstream sections above.

Format:
- Each upstream sync adds a section:
  - Header: `## <tag> - <date>`
  - Link to the upstream release
  - The upstream release notes body (Markdown), or a stub if unavailable

Initial entry (placeholder)
- No upstream sync has been applied by CI yet for this repository version.