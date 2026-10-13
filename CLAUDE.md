# Interlagos 2026 — contexto do projeto

Site da viagem de Curitiba ao GP de São Paulo de Fórmula 1 (6 a 8/11/2026). Mostra roteiro, carro e estrada, gastos compartilhados com divisão e acerto de contas, quem já pagou cada gasto e um mapa com a rota e a fase da viagem. Um celular pode compartilhar a posição ao vivo para todos verem no mapa (ver "Mapa").

- **No ar:** https://viagem-f1.netlify.app
- **Dono:** Diogo. Escreva sempre em português do Brasil.

## Como trabalhar com o Diogo

- Explique passo a passo, em linguagem simples, dizendo **onde clicar** e **qual arquivo abrir**. Ele se perde quando muitas coisas mudam de uma vez.
- Ele pediu para **não alterar nada sem explicar antes** quando estiver confuso. Na dúvida, explique primeiro e mude depois.
- Ele quer um parceiro crítico: aponte o ponto fraco de uma ideia logo de início, mas sem travar o trabalho com perguntas desnecessárias.
- A cada mudança, diga exatamente **quais arquivos trocar** e **como publicar**.
- Ele usa Windows e edita arquivos no Bloco de Notas ou no VS Code.

## A viagem

| Item | Detalhe |
| --- | --- |
| Viajantes (4) | Diogo, Antonio (pai do Diogo), João Vitor, Augusto |
| GP de São Paulo | 6 a 8/11/2026, Interlagos. Largada oficial domingo às 14:00 (Brasília). Ingressos já comprados |
| Carro | Hyundai HB20S, Movida do Aeroporto de Curitiba (loja 24h). R$ 569,37 o período |
| Retirada / devolução | Qui 05/11 20:00 / Seg 09/11 03:00, na mesma loja |
| Estrada (Qualp) | 810 km ida e volta, ~9h15 ao volante, pedágio R$ 58,90, combustível R$ 458,81 (12 km/l a R$ 6,80/l) |
| Transporte total | R$ 1.087,08 (R$ 271,77 por pessoa) |

### Roteiro definido pelo grupo

| Dia | Horário | Atividade |
| --- | --- | --- |
| Qui 05/11 | 20:00 | Retirada do carro e estrada Curitiba → SP |
| Sex 06/11 | 08:00 | Chegada em SP |
| | 09:00 | Casa do Vídeo Game |
| | 12:00 | Almoço |
| | 14:00 | Check-in no hotel |
| | 15:00 | Parque Ibirapuera |
| | 16:00 | Treino Livre 2 (Interlagos) |
| | 19:00 | Barbacoa |
| Sáb 07/11 | 08:00 | Feira da Liberdade |
| | 11:00 | Almoço |
| | 13:00 | Qualificação (Interlagos) |
| | 19:00 | Restaurante japonês |
| Dom 08/11 | 08:00 | Avenida Paulista |
| | 10:00 | Ida para a corrida (largada às 14:00) |
| | — | Volta para Curitiba |
| Seg 09/11 | 03:00 | Devolução do carro |

### Pontos em aberto

- Valor dos ingressos e do hotel ainda não foram lançados no site.
- Hotel ainda não definido no roteiro (só o horário de check-in).
- Sexta: Ibirapuera às 15:00 e Treino Livre 2 às 16:00 é apertado (~15 km em dia de GP). Já foi avisado; o site mostra um alerta nesse item.
- O início da volta foi **suposto** para domingo 08/11 às 17:00 (usado na fase do mapa). Confirmar o horário real com o grupo.
- Quem dirige cada trecho ainda não foi decidido.

## Arquitetura

```
viagem-f1/             repositório Git (GitHub: DiogoBayer/viagem-f1)
├── CLAUDE.md          este arquivo
├── .gitignore         deixa o public/config.js fora do Git
├── LEIA-ME.md         passo a passo de publicação, para o Diogo
├── firestore.rules    regras do banco (colar no console do Firebase)
├── firebase.json      só se um dia publicar pela CLI do Firebase; hoje não é usado
└── public/
    ├── index.html     o site inteiro (HTML, CSS e JS num arquivo só)
    ├── config.js      configuração do Firebase do Diogo
    └── apple-touch-icon.png   ícone 180x180 da tela inicial do celular e da aba do navegador
```

- **Hospedagem:** Netlify, publicando a pasta `public` por arrastar e soltar em Deploys. Não há build. O Netlify guarda todos os deploys, e voltar uma versão é "Publish deploy" num deploy antigo.
- **Banco:** Firestore (plano gratuito Spark), região southamerica-east1.
- **Sem login e sem código de acesso.** Decisão do Diogo: quem tem o link lê, lança, edita e apaga gastos. As regras só validam o formato dos dados e restringem o caminho.
- **Bibliotecas:** Firebase JS SDK 10.12.2 (módulos ES do gstatic), Leaflet 1.9.4 (unpkg), mapa escuro da Esri (World_Dark_Gray_Base + Reference, sem chave). Os mapas da CARTO passaram a exigir chave de API e não servem mais. Fontes Google: Saira Condensed, Barlow, JetBrains Mono.
- **`config.js`** exporta `firebaseConfig` (`export const firebaseConfig = {...}`). Ele é separado para que atualizações troquem só o `index.html`. Ele está no `.gitignore`: a cópia real com os valores fica só no computador do Diogo e no Netlify, e não vai para o GitHub. **Nunca sobrescreva o `config.js` dele.** A `apiKey` do Firebase não é segredo, mas não há motivo para versioná-la.

### Dados (Firestore)

Coleção `viagens/interlagos-2026/gastos`, um documento por gasto:

| Campo | Tipo | Observação |
| --- | --- | --- |
| `descricao` | string | 1 a 80 caracteres |
| `valor` | int | em **centavos** (R$ 569,37 = 56937), até 10.000.000 |
| `categoria` | string | Transporte, Hospedagem, Alimentação, Ingressos, Passeios, Outros |
| `pagoPor` | string | `""` (a definir) ou um dos 4 nomes |
| `divididoEntre` | lista | 1 a 4 nomes |
| `criadoEm` | string | data ISO, usada para ordenar |
| `quitados` | lista (opcional) | quem já pagou a própria parte para o `pagoPor` |

As regras em `firestore.rules` validam esses campos e só liberam esse caminho. **Qualquer campo novo precisa entrar nas regras**, e o Diogo precisa colar as regras de novo no console (Firestore Database → Regras → Publicar). Senão o site mostra "O banco recusou a gravação".

### Lógica de custos

- Cada gasto é dividido igualmente entre `divididoEntre`.
- Saldo de cada pessoa = pagou − sua parte + ajustes de `quitados`. Quando alguém marca que pagou sua parte de um gasto, o saldo dele sobe e o do pagador desce no mesmo valor.
- Gastos sem pagador entram no total e no custo por pessoa, mas ficam fora do acerto.
- O acerto sugere o mínimo de transferências (algoritmo guloso). O centavo de arredondamento vai para o maior saldo, para as contas fecharem.
- Trocar o pagador de um gasto zera os `quitados` dele.

### Mapa

Seção "Onde estamos", a primeira da página. O código fica num `<script>` próprio, separado do script do Firebase, para um não derrubar o outro.

**Feito (etapa 1):**

- Rota real da estrada em 323 pontos (`ROTA`), do Aeroporto Afonso Pena até Interlagos, tirada do OSRM/OpenStreetMap. O traçado simplificado mede ~392 km; o OSRM dá 398,6 km e o Qualp, 405 km.
- Fases só pelo relógio (`faseAtual`): Antes → Ida (05/11 20:00) → Em São Paulo (06/11 08:00) → Volta (08/11 17:00, suposto) → Fim (09/11 03:00).
- No celular, arrastar com um dedo fica desligado para não prender a rolagem da página; dois dedos aproximam.

**Feito (etapa 2), inspirado no RatoNet (github.com/Captando/RatoNet):**

- Só o celular do Diogo compartilha (`QUEM = "Diogo"` no código, sem seletor de pessoa; decisão do Diogo, que descartou o rastreio de cada um). O botão aparece para todos, então outra pessoa que tocar nele envia a própria posição como se fosse a do Diogo.
- Botão "Compartilhar posição": o celular envia a posição com `watchPosition`, no máximo a cada 10 s (parado, reenvia a cada 30 s), e pede Wake Lock para a tela não apagar.
- Dados em `viagens/interlagos-2026/aovivo`: documento `posicao` (`lat`, `lng`, `precisao` em metros, `vel` em km/h ou -1, `por`, `em`) e documento `trilha` (`pontos`: lista de textos `"lat,lng"`, um a cada 500 m, até 4.000).
- Todos veem o ponto verde, a trilha verde e a velocidade. Sem envio há mais de 90 s, vira "Última posição há X" e o ponto fica cinza.
- "Ver o carro" centraliza e segue o ponto; arrastar o mapa ou "Ver rota inteira" para de seguir. "Apagar posição e trilha" limpa os dois documentos.
- A ponte entre os dois scripts é `window.mapaViagem` (`map`, `enquadrar`).
- Só foi testado com Firestore e GPS simulados. Falta o teste real: regras publicadas, site no Netlify e um celular de verdade.

**Feito (etapa 3):**

- Quadro de dados embaixo do mapa, visível quando há posição: local, velocidade, km até o destino e clima.
- Km: `projetar` acha o ponto mais próximo da rota e escala para os 405 km (`KM_ESTRADA`). O destino é Interlagos até a fase "Volta" e Curitiba depois. A mais de 5 km da estrada mostra "—".
- Local: Nominatim (OpenStreetMap), consultado de novo só depois de 3 km e 60 s. Clima: Open-Meteo, a cada 20 km ou 15 min. Cada pessoa que abre o site faz as próprias consultas; nada disso vai para o banco.
- Sem sinal: o próprio SDK do Firestore guarda as gravações e envia quando a internet volta, desde que a página não seja recarregada. O site só mostra um aviso para quem está compartilhando.
- Sem mudança nas regras do banco.

**Feito (roteiro no mapa):**

- Botões embaixo do mapa: "Rota inteira" e um por dia. Clicar num dia (ali ou nas abas do Roteiro) troca os dois juntos. Quinta mostra a rota inteira; os outros dias mostram as paradas numeradas, ligadas por uma linha tracejada que só indica a ordem (não é o caminho pelas ruas).
- A localização de cada parada fica no próprio `DIAS`, no campo `ll: [lat, lng]`. Parada com `semLocal: true` aparece na lista com a etiqueta "Sem local no mapa" e fica fora do mapa.
- Casa do Vídeo Game: R. Geórgia, 224, Brooklin (CEP 04559-010). O OpenStreetMap não tem o número da casa, então o ponto foi estimado a ~224 m do começo da rua; pode estar um ou dois quarteirões fora. Barbacoa: unidade do Itaim Bibi, R. Dr. Renato Paes de Barros, 65 (ponto exato do OpenStreetMap).
- **Ainda sem lugar definido pelo grupo:** hotel, os dois almoços e o restaurante japonês. Com o endereço, basta trocar `semLocal:true` por `ll:[lat,lng]`. Para achar as coordenadas, usar o Nominatim (pelo WebFetch, porque o terminal do Diogo bloqueia).
- O script do mapa expõe `mostrar(paradas, rotaInteira)` em `window.mapaViagem`.

**Limites e pendências:**

- Limites já explicados ao Diogo: só funciona com o site aberto e a tela ligada, e, sem login, quem tem o link vê (e poderia gravar) a posição. Testar em casa grava o endereço de casa no banco; apagar depois.

## Como publicar uma mudança

1. Editar `public/index.html` (o `config.js` do Diogo continua o mesmo).
2. Se mudou o formato dos dados: atualizar `firestore.rules` e o Diogo cola no console do Firebase.
3. No Netlify: site `viagem-f1` → Deploys → arrastar a pasta `public` inteira (com `index.html` e `config.js`).
4. Abrir o site com Ctrl+F5 e testar.

## Como testar sem publicar

O site depende do Firebase, então abrir o `index.html` direto no navegador não carrega os gastos nem o roteiro (o mapa carrega). Para conferir o layout, dá para tirar fotos com o Edge sem janela: `msedge --headless=new --allow-file-access-from-files --window-size=1200,1100 --virtual-time-budget=15000 --screenshot=foto.png file:///.../public/index.html`. Para largura de celular, abra o site dentro de um `<iframe>` de 390px, porque o Edge não aceita janela tão estreita. O computador do Diogo não tem Node, e a rede dele bloqueia alguns endereços no terminal (Netlify, OSRM).

## Histórico do que foi feito

1. Doc de planejamento no Claude, com roteiro e orçamento.
2. Site interativo dentro do Claude (artifact), depois **apagado** a pedido do Diogo para não haver duas fontes de gastos.
3. Migração para Firebase: primeiro com código de grupo, depois **sem código**, por decisão do Diogo.
4. Hospedagem no Netlify por arrastar e soltar, sem instalar Node nem a CLI do Firebase.
5. Botões de "quem já pagou" em cada gasto, com `config.js` separado.
6. Repositório Git com a versão sem mapa marcada como `v1-site-no-ar` (ponto de retorno).
7. Mapa "Onde estamos", etapa 1: rota real e fases da viagem, sem posição ao vivo (`v2-mapa-rota`).
8. Mapa, etapa 2: posição ao vivo compartilhada e trilha (`v3-mapa-ao-vivo`). Exige as regras novas no Firebase.
9. Mapa, etapa 3: local, velocidade, km até o destino e clima (`v4-mapa-dados`).
10. Seletor de pessoa removido, só o celular do Diogo compartilha (`v5-so-meu-celular`); segundos na contagem (`v6-contagem-segundos`).
11. Roteiro no mapa: paradas de cada dia (`v7-roteiro-no-mapa`), Casa do Vídeo Game e Barbacoa (`v8-locais-sexta`).
12. Ícone da tela inicial (`v9-icone`). Foi desenhado em HTML, fotografado com o Edge sem janela em 512x512 e reduzido para 180x180. O iPhone guarda o ícone no momento de adicionar à tela inicial; para trocar, é preciso remover o atalho e adicionar de novo.

Também existe um doc "Planejamento de Viagem" no Claude, do começo da conversa. Ele está desatualizado em relação ao site e não é usado.
