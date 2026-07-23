<%*
const target = tp.config.target_file;
const folder = target.parent;

if (folder) {
  const hub = app.vault.getMarkdownFiles().find(f =>
    f.parent && f.parent.path === folder.path &&
    /^00\./.test(f.basename) &&
    f.path !== target.path
  );

  if (hub) {
    tR += `## 관련 문서\n\n- 상위 목차: [[${hub.basename}]]\n`;

    let hubContent = await app.vault.read(hub);
    const linkLine = `- [[${target.basename}]]`;

    if (!hubContent.includes(linkLine)) {
      if (/## 하위 문서/.test(hubContent)) {
        hubContent = hubContent.replace(
          /(## 하위 문서\s*\n)/,
          `$1\n${linkLine}\n`
        );
      } else {
        hubContent += `\n\n## 하위 문서\n\n${linkLine}\n`;
      }
      await app.vault.modify(hub, hubContent);
    }
  }
}
-%>
