<%*
/*
 * Templater startup template.
 * Restores .obsidian/graph.json from the golden copy (graph.default.json)
 * every time the vault loads, so ad-hoc UI fiddling (color groups, filters,
 * zoom/scale) during a session never becomes the new permanent default.
 *
 * Registered under: Settings -> Templater -> Startup templates.
 */
const configDir = app.vault.configDir; // ".obsidian"
const adapter = app.vault.adapter;

const goldenPath = `${configDir}/graph.default.json`;
const livePath = `${configDir}/graph.json`;

if (await adapter.exists(goldenPath)) {
  const golden = await adapter.read(goldenPath);
  await adapter.write(livePath, golden);

  // If a Graph View pane was already restored as part of the workspace
  // layout before this script ran, refresh it in place too.
  const parsed = JSON.parse(golden);
  for (const leaf of app.workspace.getLeavesOfType("graph")) {
    const engine = leaf.view && leaf.view.dataEngine;
    if (engine && typeof engine.setOptions === "function") {
      engine.setOptions(parsed);
      if (typeof engine.render === "function") engine.render();
    }
  }
}
-%>
