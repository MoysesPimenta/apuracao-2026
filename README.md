# Apuração 2026 — 2º turno (painel público)

Painel estático que lê os resultados oficiais do TSE diretamente no navegador
(`https://resultados.tse.jus.br/oficial/ele2026/`) e se atualiza sozinho a cada ~45 s.
Não exige login, não depende de servidor próprio: cada visitante busca os mesmos
arquivos públicos do TSE (eleição **6258** = presidente 2º turno, **6260** = governador 2º turno
nos estados com segundo turno).

Site: https://moysespimenta.github.io/apuracao-2026/

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | Painel completo (mapa, tabela por UF, comparação com o 1º turno, municípios/zonas sob demanda). Tudo inline: CSS, JS e dados do 1º turno. |
| `geo/br_uf.min.json` | Contorno simplificado dos estados (coordenadas inteiras ×100). |
| `geo/mun_<uf>.json.gz` | Contornos dos municípios de cada estado (GeoJSON gzipado; a página descomprime no navegador com `DecompressionStream`). |
| `.nojekyll` | Evita processamento Jekyll no GitHub Pages. |

## Como funciona

1. Na abertura, o painel tenta `api/snapshot` (só existe quando hospedado com uma função serverless);
   no GitHub Pages isso retorna 404 e ele passa ao modo direto.
2. Modo direto: busca os 27 arquivos de presidente + os de governador em paralelo (6 por vez),
   com recuo exponencial em `429 Too Many Requests`.
3. Municípios e zonas de um estado só são baixados quando o estado é aberto no mapa (cache de 4 min).
4. Os dados do 1º turno (resultado final, 100% das seções) estão embutidos para a coluna de comparação.

## Limitações conhecidas

- O TSE limita requisições (429); se muitos visitantes abrirem ao mesmo tempo, a atualização pode atrasar alguns segundos.
- Fonte única: totalização oficial do TSE. Não há contagem paralela por boletim de urna.

## Fonte

Dados: Tribunal Superior Eleitoral — Divulgação de Resultados (`resultados.tse.jus.br`).
Malhas municipais: IBGE (simplificadas). Painel gerado a partir do projeto privado `eleicoes2026` (build `build_pub.py`).
