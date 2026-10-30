# Dados Jurimetria — Decisões Automatizadas e IA (TJSP)

Repositório de jurimetria geoespacializado, construído no âmbito da pesquisa de doutorado em andamento na FDUSP sobre padrões técnicos e autoridade jurídica na regulação da inteligência artificial.

**Dashboard:** publicar via GitHub Pages a partir de `index.html` (Settings → Pages → Branch: main, pasta raiz).

## Escopo atual

- **Tribunal:** TJSP (apenas — TRF3, TRE-SP, TRT02 e TRT15 ficam para rodadas futuras)
- **Período:** a partir de 2018 (recorte do projeto de doutorado)
- **Fonte:** busca pública de jurisprudência no Jusbrasil
- **Casos únicos coletados:** 782 (após deduplicação por número de processo e filtro temporal)

## Metodologia

A busca cruzou pares de termos dos eixos definidos no Quadro 1 do projeto de doutorado — (Eixo B *Tecnologia* ou Eixo C *Situação de fato*) × (Eixo A *Dispositivo legal* ou Eixo D *Critério técnico de conformidade*) — usando a sintaxe de frases entre aspas da interface do Jusbrasil (que não interpreta operadores booleanos OR/AND).

Dez consultas curadas foram coletadas de forma **exaustiva** (todas as páginas de resultado):

| Consulta | Eixos | Resultados |
|---|---|---|
| `"reconhecimento facial" "norma técnica"` | C×D | 303 |
| `"bloqueio de conta" "decisão automatizada"` | C×A | 239 |
| `"algoritmo" "decisão automatizada"` | B×A | 142 |
| `"banimento" "decisão automatizada"` | C×A | 150 |
| `"inteligência artificial" "decisão automatizada"` | B×A | 34 |
| `"reconhecimento facial" "decisão automatizada"` | C×A | 30 |
| `"inteligência artificial" "norma técnica"` | B×D | 15 |
| `"moderação de conteúdo" "decisão automatizada"` | C×A | 13 |
| `"score de crédito" "decisão automatizada"` | C×A | 2 |
| `"perfilamento" "decisão automatizada"` | C×A | 5 |

## Processamento

1. **Extração:** raspagem paginada da interface pública de busca (id do processo, data de publicação, comarca), com pausas entre requisições para respeitar a proteção anti-bot do site.
2. **Deduplicação:** por número de identificação do processo (há sobreposição natural entre consultas).
3. **Filtro temporal:** exclusão de casos anteriores a 2018.
4. **Geocodificação:** a comarca é extraída do título do resultado e normalizada para o município de SP correspondente. 103 municípios únicos identificados, cobrindo **100% dos 782 casos** (nenhum caso perdido). O caso de comarca "Mogi Mirim" foi inicialmente perdido por uma variante ortográfica não tratada: a fonte geoespacial (`tbrugz/geodata-br`) registra esse município com a grafia histórica "Moji Mirim" (código IBGE 3530805); corrigido via tabela de alias.

## Estrutura de arquivos

```
index.html              — dashboard (Leaflet, mapa coroplético + ranking + KPIs)
sp_municipios.json       — GeoJSON simplificado dos 645 municípios de SP (fonte: tbrugz/geodata-br)
leaflet.css, images/     — assets da biblioteca Leaflet 1.9.4
data/raw_results.csv     — dados brutos coletados (933 linhas, antes de dedup/filtro)
data/clean_results.csv   — após deduplicação e filtro 2018+ (782 linhas)
data/geo_input.csv       — clean_results.csv + coluna de município normalizado
data/municipio_counts.json — contagem agregada por município (id IBGE)
data/meta.json           — metadados e resumos por consulta/ano
```

## Limitações desta amostra preliminar

- Cobre apenas TJSP; os demais tribunais (TRF3, TRE-SP, TRT02, TRT15) estão pendentes.
- A comarca é inferida do título do resultado de busca, não do inteiro teor da decisão — pode haver imprecisão residual em casos de formatação atípica do processo.
- Cada caso está mapeado a um único município (o da comarca de origem do processo); não há desagregação por vara/foro regional dentro do município.
- Não houve leitura do mérito das decisões; a amostra reflete ocorrência de termos nos resultados de busca, não necessariamente a aplicação efetiva de "decisão automatizada" no sentido jurídico do art. 20 da LGPD.

---
**AVISO:** Este repositório foi gerado com auxílio de IA. Revisão humana e validação metodológica são necessárias antes de uso em publicação acadêmica.
