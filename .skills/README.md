# Skills locais do projeto

Skills que valem só para este repositório ficam aqui, uma pasta por skill, com `SKILL.md`
dentro:

```
.skills/
  <nome-da-skill>/
    SKILL.md
```

`SKILL.md` começa com frontmatter:

```yaml
---
name: <nome-da-skill>
description: <quando usar, em uma frase; é o que dispara a leitura>
---
```

Regras:

- Uma skill por assunto; se virar dois assuntos, divide.
- Fato verificável com fonte vai na skill ou em `docs/pesquisa.md`, nunca duplicado nos dois.
- Skill que deixa de valer é apagada, não comentada.
- `AGENTS.md` aponta para cada skill — skill sem ponteiro não é encontrada.
