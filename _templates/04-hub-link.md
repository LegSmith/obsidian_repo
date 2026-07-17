<%*
const target = tp.config.target_file;
const folder = target.parent;
const parent = folder ? folder.parent : null;

if (folder && parent) {
  const normalize = (s) => s.replace(/[.\s]/g, "").toLowerCase();
  const folderNorm = normalize(folder.name);

  const hub = app.vault.getMarkdownFiles().find(f =>
    f.parent && f.parent.path === parent.path &&
    normalize(f.basename) === folderNorm &&
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
  } else {
    tR += `## 관련 문서\n\n- 상위 목차: [[${folder.name}]] (자동 매칭 실패 - 직접 확인 필요)\n`;
  }
}
-%>
