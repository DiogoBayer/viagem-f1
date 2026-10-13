# Interlagos 2026 — contexto do projeto

Site da viagem de Curitiba ao GP de São Paulo de Fórmula 1 (6 a 8/11/2026). Mostra roteiro, carro e estrada, gastos compartilhados com divisão e acerto de contas, quem já pagou cada gasto e um mapa com a fase da viagem e a posição do celular.

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
interlagos-firebase/
├── CLAUDE.md          este arquivo
├── LEIA-ME.md         passo a passo de publicação, para o Diogo
├── firestore.rules    regras do banco (colar no console do Firebase)
├── firebase.json      só se um dia publicar pela CLI do Firebase; hoje não é usado
└── public/
    ├── index.html     o site inteiro (HTML, CSS e JS num arquivo só)
    └── config.js      configuração do Firebase do Diogo
```

- **Hospedagem:** Netlify, publicando a pasta `public` por arrastar e soltar em Deploys. Não há build. O Netlify guarda todos os deploys, e voltar uma versão é "Publish deploy" num deploy antigo.
- **Banco:** Firestore (plano gratuito Spark), região southamerica-east1.
- **Sem login e sem código de acesso.** Decisão do Diogo: quem tem o link lê, lança, edita e apaga gastos. As regras só validam o formato dos dados e restringem o caminho.
- **Bibliotecas:** Firebase JS SDK 10.12.2 (módulos ES do gstatic), Leaflet 1.9.4 (unpkg), mapas CARTO (light_all/dark_all). Fontes Google: Saira Condensed, Barlow, JetBrains Mono.
- **`config.js`** exporta `firebaseConfig` (`export const firebaseConfig = {...}`). Ele é separado para que atualizações troquem só o `index.html`. No repositório ele fica com `COLE_AQUI`; a cópia real com os valores está só no computador do Diogo e no Netlify. **Nunca sobrescreva o `config.js` dele.** A `apiKey` do Firebase não é segredo, mas não há motivo para versioná-la.

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

- Rota aproximada por ~26 pontos da BR-116 (Régis Bittencourt), do Aeroporto Afonso Pena até Interlagos. O traçado mede ~356 km, então as distâncias exibidas são escaladas para os 405 km reais (`KM_ESTRADA`).
- Fases: Antes → Ida (05/11 20:00) → Em São Paulo (06/11 08:00) → Volta (08/11 17:00, suposto) → Concluída (09/11 03:00). Com a posição ligada, o site corrige: dentro de 30 km do centro de SP conta como "Em São Paulo"; na volta, dentro de 15 km do aeroporto de Curitiba conta como "Concluída".
- A posição usa `watchPosition`, só com o site aberto, e **não é enviada ao banco**.

## Como publicar uma mudança

1. Editar `public/index.html` (o `config.js` do Diogo continua o mesmo).
2. Se mudou o formato dos dados: atualizar `firestore.rules` e o Diogo cola no console do Firebase.
3. No Netlify: site `viagem-f1` → Deploys → arrastar a pasta `public` inteira (com `index.html` e `config.js`).
4. Abrir o site com Ctrl+F5 e testar.

## Como testar sem publicar

O site depende do Firebase, então abrir o `index.html` direto no navegador não carrega os gastos. Para conferir layout e lógica, já usamos Playwright com o Firebase simulado (respostas falsas para `firebase-app.js` e `firebase-firestore.js`), geolocalização falsa e relógio fixo. Para mudanças só de lógica, extraia as funções (`calc`, `projetar`, `faseAtual`) e rode no Node com dados de exemplo.

## Histórico do que foi feito

1. Doc de planejamento no Claude, com roteiro e orçamento.
2. Site interativo dentro do Claude (artifact), depois **apagado** a pedido do Diogo para não haver duas fontes de gastos.
3. Migração para Firebase: primeiro com código de grupo, depois **sem código**, por decisão do Diogo.
4. Hospedagem no Netlify por arrastar e soltar, sem instalar Node nem a CLI do Firebase.
5. Botões de "quem já pagou" em cada gasto, com `config.js` separado.
6. Mapa "Onde estamos", com fases da viagem e posição do celular.
7. Repositório Git iniciado; versão atual marcada como `v1-mapa`.

Também existe um doc "Planejamento de Viagem" no Claude, do começo da conversa. Ele está desatualizado em relação ao site e não é usado.
