# Apuração 2026 — painel público (1º e 2º turnos)

Painel estático que lê os resultados oficiais do TSE diretamente no navegador
(`https://resultados.tse.jus.br/oficial/ele2026/`) e se atualiza sozinho a cada ~45 s enquanto a apuração está aberta.
Não exige login, não depende de servidor próprio: cada visitante busca os mesmos
arquivos públicos do TSE (eleições **6257/6259** = 1º turno, **6258/6260** = 2º turno).

Site: https://moysespimenta.github.io/apuracao-2026/

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | Painel completo: seletor **2º turno / 1º turno** (a eleição mais recente é o padrão), mapa com drilldown Brasil → estado → município → local de votação / bairro → seção, tabela por UF, comparação com o 1º turno. Tudo inline: CSS, JS e resumo do 1º turno. |
| `geo/br_uf.min.json` | Contorno simplificado dos estados (coordenadas inteiras ×100). |
| `geo/mun_<uf>.json.gz` | Contornos dos municípios de cada estado (GeoJSON gzipado; a página descomprime no navegador com `DecompressionStream`). |
| `lv/<uf>.json.gz` | Locais de votação de cada estado (nome, bairro, endereço, lat/lon, eleitores e as seções de cada zona). Fonte: dados abertos do TSE (`eleitorado_local_votacao_2026`). **Necessário para o nível município → locais/bairros.** |
| `sec/<uf>.json.gz` | Opcional: resultado por seção já extraído dos boletins de urna (2º turno). Quando existe, o município abre na hora; o que faltar é lido ao vivo do TSE. |
| `sec_t1/<uf>.json.gz` | Opcional: o mesmo para o 1º turno (pleito 3220). |
| `.nojekyll` | Evita processamento Jekyll no GitHub Pages. |

## Como funciona

1. Na abertura, o painel tenta `api/snapshot` (só existe quando hospedado com uma função serverless);
   no GitHub Pages isso retorna 404 e ele passa ao modo direto.
2. Modo direto: busca os 27 arquivos de presidente + os de governador em paralelo (6 por vez),
   com recuo exponencial em `429 Too Many Requests`. O 1º turno é carregado da mesma forma quando selecionado.
3. Municípios e zonas de um estado só são baixados quando o estado é aberto no mapa (cache de 4 min).
4. Ao abrir um **município**, o painel carrega `lv/<uf>.json.gz` e `sec/<uf>.json.gz` (se existir). Para as seções
   que ainda não têm resultado, ele lê os **boletins de urna direto do TSE**
   (`arquivo-urna/<pleito>/dados/<uf>/<mun>/<zona>/<seção>/…-aux.json` → `<hash>/…-bu.dat`), decodifica o
   ASN.1/DER no navegador e vai pintando locais, bairros e seções conforme chegam (8 leituras em paralelo,
   recuo em 429; a nota do mapa mostra "lendo boletins direto do TSE… X de Y"). Sair do município cancela a leitura.
5. No nível município há dois modos: **Pontos** (um círculo por local, tamanho = eleitorado) e **Áreas**
   (células de Voronoi recortadas pelo contorno do município, pintadas como os municípios no mapa do estado).

## Limitações conhecidas

- O TSE limita requisições (429); se muitos visitantes abrirem ao mesmo tempo, a atualização pode atrasar alguns segundos.
  Capitais grandes (milhares de seções) levam alguns minutos para completar a leitura ao vivo; o mapa é preenchido progressivamente.
- ~2% das seções votam em local diferente do cadastrado nos dados abertos (remanejamento); nesses casos a seção é atribuída
  ao local informado no próprio boletim de urna.
- Zonas eleitorais não têm malha oficial: aparecem como lista, não no mapa.

## Fonte

Dados: Tribunal Superior Eleitoral — Divulgação de Resultados e boletins de urna (`resultados.tse.jus.br`), dados abertos do eleitorado (`cdn.tse.jus.br`).
Malhas municipais: IBGE (simplificadas). Painel gerado a partir do projeto privado `eleicoes2026` (build `build_pub.py`).
