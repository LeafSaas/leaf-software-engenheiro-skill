# leaf-software-engenheiro-skill

Skill instalavel do Codex para trabalhar em projetos Leaf, especialmente `leaf-delivery`.

## O que esta skill cobre

- versionamento e branch workflow
- roadmap e `test.map.md`
- padroes de implementacao e arquitetura modular
- validacao local e gates de entrega
- documentacao obrigatoria
- deploy GCP e operacao real

## Estrutura

O caminho instalavel da skill neste repo e:

`skills/leaf-software-engenheiro`

## Instalacao

Se a pessoa ja tiver Codex e o skill installer disponivel:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo ramosbezzu/leaf-software-engenheiro-skill \
  --path skills/leaf-software-engenheiro
```

Ou por URL:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --url https://github.com/ramosbezzu/leaf-software-engenheiro-skill/tree/main/skills/leaf-software-engenheiro
```

Depois da instalacao:

```bash
Restart Codex to pick up new skills.
```

## Uso

No prompt:

```text
$leaf-software-engenheiro
```

Ou:

```text
Use $leaf-software-engenheiro para trabalhar no leaf-delivery
```
