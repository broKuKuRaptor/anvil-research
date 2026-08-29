# RULES — sources

`sources/` — source/provenance registry проекта.

## Категории
- `documentation.md`
- `repositories.md`
- `tools.md`
- `patches.md`
- `images.md`

## Статусы
`PRIMARY`, `VERIFIED`, `REFERENCE`, `HISTORICAL`, `COMMUNITY`, `UNVERIFIED`, `REJECTED`.

## Правила
- регистрировать источник при первом существенном использовании;
- фиксировать exact commit/version/hash, когда возможно;
- предпочитать primary source;
- mirror не выдавать за primary;
- rejected source сохранять, если причина отклонения полезна;
- большие binaries здесь не хранить;
- images/OTA/firmware metadata хранить в `images.md`, сами файлы — в local artifact storage.
