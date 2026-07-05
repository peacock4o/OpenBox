<%*
/*
 * Templater startup template. _data/lessons.json and _data/concepts.json
 * are the ONLY sources of truth for the graph -- one-way, 1:1 JSON -> MD.
 * Every lesson note and every concept note is fully regenerated on every
 * vault load, except `watched` on lessons (a per-note progress toggle you
 * flip by hand in Obsidian, carried over from the existing note rather
 * than sourced from JSON) and Tier (computed here, not stored in JSON).
 *
 * Pipeline:
 *   1. Read concepts.json ({concept, family, description} objects), write a
 *      preliminary concept file for every declared entry (empty Taught
 *      by/Used by) -- so every concept node exists even if this sync gets
 *      interrupted before step 3.
 *   2. Read lessons.json, compute tier, write every full lesson file,
 *      accumulating Taught-by/Used-by associations in memory as each
 *      lesson is processed (including any concept a lesson references that
 *      concepts.json didn't declare -- auto-registered, blank family/
 *      description, not a hard error).
 *   3. Rewrite every concept file (declared + auto-registered) with its
 *      accumulated Taught by/Used by.
 *
 * Tier: multi-source BFS, first-reached-wins. Root = a lesson with no
 * preconcepts at all. Every other lesson is tiered the first time any of
 * its preconcepts is produced by an already-tiered lesson; the
 * lesson<->concept hop is collapsed into one tier step since Tier is a
 * lesson-only field. Naturally cycle-safe: a node is tiered exactly once
 * and never revisited.
 *
 * Registered under: Settings -> Templater -> Startup templates.
 */
const lessonsPath = "_data/lessons.json";
const conceptsPath = "_data/concepts.json";
const lessonsDir = "lessons";
const conceptsDir = "concepts";

if (!(await app.vault.adapter.exists(lessonsPath))) return;

const lessonsCatalog = JSON.parse(await app.vault.adapter.read(lessonsPath));
const lessons = {};
for (const l of lessonsCatalog.lessons) lessons[l.file] = l;

const declared = {};
if (await app.vault.adapter.exists(conceptsPath)) {
  const declaredList = JSON.parse(await app.vault.adapter.read(conceptsPath)).concepts;
  for (const c of declaredList) declared[c.concept] = c;
}

async function writeConceptFile(c, producerFiles, consumerFiles) {
  const meta = declared[c] || {};
  const path = `${conceptsDir}/concept__${c}.md`;
  const existing = app.vault.getAbstractFileByPath(path);
  // The arrow always points producer -> consumer. The producer -> concept
  // edge already lives on the producing lesson's own Postconcepts link, so
  // Taught by is plain text here (a real link would create a reverse edge
  // and Graph View would show arrowheads on both ends). Used by carries the
  // concept -> consumer edge as a real link, completing the single-direction
  // chain: producer -> concept -> consumer.
  const prodLinks = producerFiles.map((f) => `- ${f}`).join("\n") || "*(none)*";
  const consLinks = consumerFiles.map((f) => `- [[${f}]]`).join("\n") || "*(none)*";

  const fmLines = ["---", `concept: ${c}`];
  if (meta.family) fmLines.push(`family: ${meta.family}`);
  const tags = ["node/concept", `concept/${c}`];
  if (meta.family) tags.push(`family/${meta.family}`);
  fmLines.push("tags:", ...tags.map((t) => `  - ${t}`), "---");

  let body = "";
  if (meta.description) body += `\n## Description\n\n${meta.description}\n`;
  body += `\n## Taught by\n${prodLinks}\n\n## Used by\n${consLinks}\n`;

  const content = fmLines.join("\n") + "\n" + body;
  if (existing) {
    await app.vault.modify(existing, content);
  } else {
    await app.vault.create(path, content);
  }
}

// --- 1. preliminary concept skeletons ---
for (const c of Object.keys(declared)) {
  await writeConceptFile(c, [], []);
}

// --- tier: multi-source BFS, root = no preconcepts ---
const tier = {};
let frontier = Object.keys(lessons).filter((f) => !lessons[f].preconcepts.length);
for (const f of frontier) tier[f] = 0;

let level = 0;
while (frontier.length) {
  const newConcepts = new Set();
  for (const f of frontier) for (const c of lessons[f].postconcepts) newConcepts.add(c);

  const nextFrontier = [];
  for (const [f, l] of Object.entries(lessons)) {
    if (f in tier) continue;
    if (l.preconcepts.some((c) => newConcepts.has(c))) {
      tier[f] = level + 1;
      nextFrontier.push(f);
    }
  }
  frontier = nextFrontier;
  level += 1;
}
for (const f of Object.keys(lessons)) if (!(f in tier)) tier[f] = 99; // unreached, needs data review

// --- 2. write lessons, accumulating producers/consumers as we go ---
const producers = {};
const consumers = {};
function addAssociation(map, concept, file) {
  if (!(concept in map)) map[concept] = [];
  map[concept].push(file);
}

for (const [file, l] of Object.entries(lessons)) {
  const path = `${lessonsDir}/${file}.md`;
  const existing = app.vault.getAbstractFileByPath(path);
  const watched = existing
    ? !!app.metadataCache.getFileCache(existing)?.frontmatter?.watched
    : false;

  const t = tier[file];
  const tags = [
    "node/lesson",
    `type/${l.type}`,
    ...l.preconcepts.map((c) => `preconcept/${c}`),
    ...l.postconcepts.map((c) => `postconcept/${c}`),
    `tier/${t}`,
  ];
  const videoId = (l.url.match(/[?&]v=([^&]+)/) || [])[1] || "";
  const thumb = videoId ? `https://img.youtube.com/vi/${videoId}/hqdefault.jpg` : "";
  // Preconcepts is plain text -- the consumer's edge lives on the concept's
  // Used by side instead, so this lesson doesn't also link back to it.
  const preconceptLinks = l.preconcepts.map((c) => `- ${c}`).join("\n");
  const postconceptLinks = l.postconcepts.map((c) => `- [[concept__${c}]]`).join("\n");

  const frontmatter = [
    "---",
    `watched: ${watched}`,
    `title: "${l.title.replace(/"/g, '\\"')}"`,
    `url: ${l.url}`,
    `type: ${l.type}`,
    `tier: ${t}`,
    "tags:",
    ...tags.map((tg) => `  - ${tg}`),
    "---",
  ].join("\n");

  const body = `
# ${l.title}

![thumbnail](${thumb})

▶ **[Watch on YouTube](${l.url})**

## ⬆ Preconcepts
${preconceptLinks}

## ⬇ Postconcepts
${postconceptLinks}

## Original description

${l.description}
`;

  const content = frontmatter + "\n" + body;
  if (existing) {
    await app.vault.modify(existing, content);
  } else {
    await app.vault.create(path, content);
  }

  for (const c of l.postconcepts) addAssociation(producers, c, file);
  for (const c of l.preconcepts) addAssociation(consumers, c, file);
}

// --- 3. rewrite concept files with accumulated data (declared + auto-registered) ---
const allConcepts = new Set([...Object.keys(declared), ...Object.keys(producers), ...Object.keys(consumers)]);
for (const c of allConcepts) {
  const prodFiles = (producers[c] || []).slice().sort();
  const consFiles = (consumers[c] || []).slice().sort();
  await writeConceptFile(c, prodFiles, consFiles);
}
-%>
