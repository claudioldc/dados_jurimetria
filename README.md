# Dados Jurimetria — Decisões Automatizadas e IA (TJSP)

Repositório de jurimetria geoespacializado, construído no âmbito da pesquisa de doutorado em andamento na FDUSP sobre padrões técnicos e autoridade jurídica na regulação da inteligência artificial.

**Dashboard:** publicar via GitHub Pages a partir de `index.html` (Settings → Pages → Branch: main, pasta raiz).

## Escopo atual

- **Tribunal:** TJSP (apenas — TRF3, TRE-SP, TRT02 e TRT15 ficam para rodadas futuras)
- **Período:** a partir de 2018 (recorte do projeto de doutorado)
- **Fonte:** busca pública de jurisprudência no Jusbrasil
- **Casos únicos coletados:** 782 (após deduplicação por número de processo e filtro temporal), cada um com o **número de processo (CNJ) completo** e o **link direto para a decisão no Jusbrasil**

## Metodologia

A busca cruzou pares de termos dos eixos definidos no Quadro 1 do projeto de doutorado — (Eixo B *Tecnologia* ou Eixo C *Situação de fato*) × (Eixo A *Dispositivo legal* ou Eixo D *Critério técnico de conformidade*) — usando a sintaxe de frases entre aspas da interface do Jusbrasil (que não interpreta operadores booleanos OR/AND).

Dez consultas curadas foram coletadas de forma **exaustiva** (todas as páginas de resultado):

| Consulta | Eixos | Resultados (pós-dedup/filtro) |
|---|---|---|
| `"reconhecimento facial" "norma técnica"` | C×D | 303 |
| `"bloqueio de conta" "decisão automatizada"` | C×A | 196 |
| `"algoritmo" "decisão automatizada"` | B×A | 131 |
| `"banimento" "decisão automatizada"` | C×A | 65 |
| `"inteligência artificial" "decisão automatizada"` | B×A | 34 |
| `"reconhecimento facial" "decisão automatizada"` | C×A | 27 |
| `"inteligência artificial" "norma técnica"` | B×D | 12 |
| `"moderação de conteúdo" "decisão automatizada"` | C×A | 10 |
| `"perfilamento" "decisão automatizada"` | C×A | 3 |
| `"score de crédito" "decisão automatizada"` | C×A | 1 |

## Processamento

1. **Extração:** raspagem paginada da interface pública de busca, capturando para cada resultado o **id interno do Jusbrasil**, a **data**, o **número de processo completo no padrão CNJ** (`NNNNNNN-DD.AAAA.J.TR.OOOO`, extraído do título do resultado ou da página individual do caso quando o título não trazia o número formatado) e a **comarca de origem**, com pausas entre requisições para respeitar a proteção anti-bot do site.
2. **Deduplicação:** por número de identificação do processo (há sobreposição natural entre consultas).
3. **Filtro temporal:** exclusão de casos anteriores a 2018.
4. **Geocodificação:** a comarca é extraída do título/página do resultado e normalizada para o município de SP correspondente. 103 municípios únicos identificados, cobrindo **100% dos 782 casos** (nenhum caso perdido). O caso de comarca "Mogi Mirim" foi inicialmente perdido por uma variante ortográfica não tratada: a fonte geoespacial (`tbrugz/geodata-br`) registra esse município com a grafia histórica "Moji Mirim" (código IBGE 3530805); corrigido via tabela de alias.
5. **Link direto:** o link para a decisão no Jusbrasil é reconstruído a partir do id interno (`https://www.jusbrasil.com.br/jurisprudencia/tj-sp/{id}`), disponível para os 782 casos.

## Dashboard interativo

O mapa coroplético mostra a concentração de casos por município. **Clicando em qualquer município no mapa, ou em qualquer linha do ranking lateral**, abre-se um quadro com a lista numerada de todos os processos daquele município — número CNJ, data, **valor da causa** e link direto para a decisão no Jusbrasil — para os 103 municípios do estado, não apenas São Paulo, com o **somatório total do valor da causa em destaque** na linha final de cada quadro.

## Valor da causa

Para cada um dos 782 processos, foi feita uma consulta pública a duas fontes, conforme o sistema de origem do processo (identificável pelo dígito inicial do número CNJ):

- **e-SAJ/TJSP** (1º grau, subsistema `cpopg`) para processos com CNJ iniciado em "0" ou "1" — a grande maioria dos casos.
- **eproc/TJSP** (Juizados Especiais Cíveis) para processos com CNJ iniciado em "4", que não tramitam no e-SAJ e por isso não retornavam valor na primeira consulta.

Em ambas, o campo "valor da causa"/"valor da ação" foi extraído a partir do número CNJ completo, na página de consulta processual.

- **Cobertura:** 768 dos 782 casos (98,2%) tiveram o valor da causa localizado; somatório geral de **R$ 18.552.987,71** entre os casos com valor encontrado.
- **Os 14 casos restantes (1,8%)** não retornaram valor da causa em nenhuma das duas consultas públicas.
- **Tratamento dos casos sem valor:** marcados como "não disponível" no dashboard e na planilha, e **excluídos do somatório** — nunca presumidos como zero. O quadro de cada município sinaliza quando o somatório cobre apenas parte dos processos daquele município.
- **Dados brutos da coleta:** `data/valor_causa.csv` (consulta e-SAJ, um valor por CNJ, status `OK`/`NOT_FOUND`/`NO_FIELD`); `data/eproc_valor_causa.csv` (consulta eproc complementar, mesmo formato, 233 CNJs); `data/municipio_sums.json` (somatório final combinando as duas fontes, nº de casos com valor e nº total de casos, por município).

## Estrutura de arquivos

```
index.html                    — dashboard (Leaflet, mapa coroplético + ranking + quadro de processos por município, com valor da causa e somatório)
dados_jurimetria_782_casos.xlsx — planilha com os 782 casos (CNJ + valor da causa + link), resumo por município (com somatório) e por consulta
sp_municipios.json             — GeoJSON simplificado dos 645 municípios de SP (fonte: tbrugz/geodata-br)
leaflet.css, images/           — assets da biblioteca Leaflet 1.9.4
data/raw_results_v2.csv        — dados brutos coletados (933 linhas, antes de dedup/filtro), com CNJ completo
data/clean_results_v2.csv      — após deduplicação e filtro 2018+ (782 linhas), com CNJ completo
data/geo_input_v2.csv          — clean_results_v2.csv + município normalizado + código IBGE + valor da causa
data/municipio_counts.json     — contagem agregada por município (id IBGE)
data/municipio_cases.json      — lista de processos (CNJ + link + data + valor da causa) por município, usada pelo quadro do dashboard
data/municipio_sums.json       — somatório do valor da causa, nº de casos com valor e nº total de casos, por município
data/valor_causa.csv           — coleta bruta do valor da causa por CNJ (consulta e-SAJ/TJSP), com status OK/NOT_FOUND/NO_FIELD
data/eproc_valor_causa.csv     — coleta bruta complementar do valor da causa por CNJ (consulta eproc/TJSP, Juizados Especiais Cíveis), 233 CNJs, mesmo formato
data/meta.json                 — metadados e resumos por consulta/ano
data/raw_results.csv, clean_results.csv, geo_input.csv — versões anteriores (sem CNJ/link), mantidas para histórico
```

## Limitações desta amostra preliminar

- Cobre apenas TJSP; os demais tribunais (TRF3, TRE-SP, TRT02, TRT15) estão pendentes.
- A comarca é inferida do título do resultado de busca (ou da página individual do caso, nos ~2% de casos em que o título não trazia o número formatado), não do inteiro teor da decisão — pode haver imprecisão residual em casos de formatação atípica do processo.
- Cada caso está mapeado a um único município (o da comarca de origem do processo); não há desagregação por vara/foro regional dentro do município.
- Não houve leitura do mérito das decisões; a amostra reflete ocorrência de termos nos resultados de busca, não necessariamente a aplicação efetiva de "decisão automatizada" no sentido jurídico do art. 20 da LGPD.
- O valor da causa está disponível para 98,2% dos casos (768 de 782; ver seção acima), combinando consulta ao e-SAJ/TJSP (CNJ iniciado em "0"/"1") e ao eproc/TJSP (CNJ iniciado em "4", Juizados Especiais Cíveis). Os 14 casos restantes (1,8%) não retornaram valor da causa em nenhuma das duas consultas públicas. Os somatórios apresentados no dashboard e na planilha refletem apenas os processos com valor efetivamente localizado — não são proxies do valor total real da amostra.

---
**AVISO:** Este repositório foi gerado com auxílio de IA. Revisão humana e validação metodológica são necessárias antes de uso em publicação acadêmica.
