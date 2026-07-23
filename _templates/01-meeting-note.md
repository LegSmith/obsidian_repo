<%*
// 1. 파일명을 회의정리_yyMMDD_HHmm 형식으로 자동 지정
const now = tp.date.now("YYMMDD_HHmm");
const newName = `회의정리_${now}`;
await tp.file.rename(newName);

const target = tp.config.target_file;
const folder = target.parent;

// 2. 같은 폴더에서 가장 최근 이전 회의 문서 탐색
const siblings = folder
  ? app.vault.getMarkdownFiles().filter(f =>
      f.parent && f.parent.path === folder.path &&
      /^회의정리_\d{6}_\d{4}$/.test(f.basename) &&
      f.path !== target.path
    )
  : [];
siblings.sort((a, b) => b.basename.localeCompare(a.basename));
const prev = siblings.length > 0 ? siblings[0] : null;

if (prev) {
  // 이전 회의 문서에 "다음 회의" 링크 추가
  let prevContent = await app.vault.read(prev);
  const nextLinkLine = `- 다음 회의: [[${target.basename}]]`;
  if (!prevContent.includes(nextLinkLine)) {
    if (/## 관련 문서/.test(prevContent)) {
      prevContent = prevContent.replace(
        /(## 관련 문서\s*\n)/,
        `$1\n${nextLinkLine}\n`
      );
    } else {
      prevContent += `\n\n## 관련 문서\n\n${nextLinkLine}\n`;
    }
    await app.vault.modify(prev, prevContent);
  }
}

// 3. 본문 스켈레톤
tR += `### 스몰토크\n- \n\n\n\n---\n### 관광공사\n- \n\n\n\n---\n### 고용부\n- \n\n`;

// 4. 관련 문서(상위 목차 / 이전 회의) 링크 섹션
if (folder) {
  const hub = app.vault.getMarkdownFiles().find(f =>
    f.parent && f.parent.path === folder.path &&
    /^00\./.test(f.basename) &&
    f.path !== target.path
  );

  tR += `## 관련 문서\n\n`;

  if (hub) {
    tR += `- 상위 목차: [[${hub.basename}]]\n`;

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

  if (prev) {
    tR += `- 이전 회의: [[${prev.basename}]]\n`;
  }
}
-%>
