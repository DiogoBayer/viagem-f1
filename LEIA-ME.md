# Interlagos 2026 no Firebase + Netlify

O site fica hospedado no Netlify e os gastos ficam no Firestore, do Firebase. Tudo é gratuito e feito pelo navegador, sem instalar nada. Não há código de acesso: quem tem o link vê e lança gastos.

## O que tem na pasta

| Arquivo | Para que serve |
| --- | --- |
| `public/index.html` | O site (roteiro, carro, custos e acerto) |
| `public/config.js` | A configuração do seu Firebase. Cole uma vez e não mexa mais |
| `public/apple-touch-icon.png` | O ícone que aparece ao adicionar o site à tela inicial do celular |
| `firestore.rules` | Regras do banco, para colar no console do Firebase |
| `firebase.json` | Só é usado se um dia publicar pela ferramenta do Firebase. Pode ignorar |

## Passo a passo

### 1. Criar o banco
1. Em <https://console.firebase.google.com>, abra o seu projeto e clique em **Firestore Database** no menu lateral.
2. Se aparecer **Criar banco de dados**, clique. Escolha a região **southamerica-east1 (São Paulo)** e o modo **produção**.

Não precisa criar coleção nem documento. O site cria tudo no primeiro gasto.

### 2. Colar as regras
1. Ainda no Firestore Database, abra a aba **Regras** (Rules).
2. Abra `firestore.rules` no Bloco de Notas, **Ctrl+A**, **Ctrl+C**.
3. No Firebase, clique no texto das regras, **Ctrl+A**, **Ctrl+V** e clique em **Publicar**.

### 3. Colar a configuração no site
1. No Firebase: engrenagem → **Configurações do projeto** → **Seus apps** → procure o bloco **Config** (`const firebaseConfig = { ... };`).
2. Abra `public/config.js` no Bloco de Notas ou VS Code e substitua os seis `COLE_AQUI` pelos valores do seu bloco. Mantenha `export const firebaseConfig = {` no começo.
3. Salve. Procure `COLE_AQUI` de novo: não pode sobrar nenhum.

### 4. Publicar no Netlify
1. Acesse <https://app.netlify.com/drop> e entre com uma conta gratuita (dá para usar a conta Google).
2. Arraste a **pasta `public`** inteira para a área indicada.
3. Abra o link que aparecer, desça até **Custos e divisão** e clique em **Lançar aluguel, pedágio e combustível**.

Para trocar o nome do link: no painel do Netlify, abra o site → **Site configuration** → **Change site name**.

## Para mudar algo depois
Troque só o `public/index.html` (o `config.js` fica como está) e arraste a pasta `public` de novo no Netlify, na aba **Deploys** do site. Os gastos não são afetados, porque ficam no Firebase.

## Marcar quem já pagou
Em cada gasto com pagador definido aparece a linha "Já pagaram ...", com um botão por pessoa da divisão. Quem acertou sua parte toca no próprio nome. O botão fica verde e o acerto de contas lá embaixo já desconta. Tocar de novo desmarca. Trocar o pagador de um gasto apaga as marcações dele.

## Mapa e posição ao vivo
A seção "Onde estamos" mostra a rota e a fase da viagem. Para o grupo ver onde o carro está:

1. No celular do Diogo, abra o site e toque em **Compartilhar posição**. Aceite o pedido de localização. Só esse celular deve compartilhar.
2. Deixe o site aberto, com a tela ligada e o celular carregando. Se a tela apagar ou o site fechar, o envio para.
3. Quem abrir o site vê o ponto verde e a trilha. **Ver o carro** aproxima e acompanha o ponto.
4. Embaixo do mapa aparecem a cidade onde o carro está, a velocidade, quantos km faltam e o clima no local.
5. Sem internet na estrada, o site guarda as posições e envia quando o sinal volta. Não feche nem recarregue o site nesse meio-tempo.
6. Para encerrar, toque em **Parar de compartilhar**. **Apagar posição e trilha** limpa tudo do banco.

Os botões embaixo do mapa (**Rota inteira**, **Qui 05**, **Sex 06**...) mostram as paradas de cada dia do roteiro, numeradas na ordem. As abas do Roteiro fazem o mesmo. Paradas com a etiqueta "Sem local no mapa" ainda não têm endereço definido.

Na primeira vez, é preciso colar de novo as regras (passo 2), porque a posição usa um lugar novo no banco.

## Se der problema

| Sintoma | Causa provável |
| --- | --- |
| "Falta colar a configuração do Firebase" | O `config.js` não está na pasta `public` ou sobrou algum `COLE_AQUI` |
| Página sem a parte de gastos | O `config.js` perdeu a palavra `export` |
| "O banco recusou o acesso" | As regras do passo 2 não foram publicadas |
| Fica em "Carregando gastos…" | O banco do passo 1 não foi criado |
| "A posição ao vivo ainda não foi liberada no banco" | As regras novas do passo 2 não foram publicadas |
| "A localização está bloqueada para este site" | Libere a localização para o site nas configurações do navegador do celular |
| O ponto parou e mostra "Última posição há X" | A tela do celular apagou, o site foi fechado ou acabou o sinal |

## Segurança
Quem tem o link pode lançar, editar e apagar qualquer gasto, e também ver, gravar e apagar a posição ao vivo. Passe o link só para o grupo. As regras impedem que alguém grave dados malformados ou use outras partes do banco.
