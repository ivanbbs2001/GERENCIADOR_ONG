# GERENCIADOR_ONG

Agenda de atendimentos e entrega de cestas básicas para ONG.

## O que tem

- **Agenda** por dia (uma coluna por profissional, mais a coluna de cestas) e por semana. Clique em um horário vago para agendar; arraste para mudar horário, profissional ou dia; puxe a borda inferior para mudar a duração.
- **Beneficiários**: cadastro, histórico de atendimentos e cestas, taxa de não comparecimento e prontuário (apontamentos médicos).
- **Profissionais**: especialidade, registro, dias de atendimento, duração padrão (15 min para médicos, 30 min para os demais).
- **Cestas**: entregas do mês, com marcação de entregue.
- **WhatsApp**: link `wa.me` com os horários da semana do beneficiário. Texto editável em Ajustes.

## Como funciona

`index.html` é um arquivo único, sem etapa de build. Os dados ficam no Firestore e o acesso é por e-mail e senha (Firebase Authentication). Sem o `firebase-config.js` preenchido, a página abre em modo demonstração e não salva nada.

Perfis (campo `papel` em `usuarios/<uid>`):

| Papel | Pode |
| --- | --- |
| `recepcao` | Agenda, cadastros, cestas. Não vê o prontuário. |
| `clinico` | Tudo da recepção, mais ler o prontuário e registrar apontamentos. |
| `admin` | Tudo, mais alterar os ajustes e liberar usuários. |

As regras em `firestore.rules` é que garantem isso no servidor. Apontamentos do prontuário não podem ser alterados nem apagados depois de gravados.

## Configuração (uma vez)

1. No [Firebase Console](https://console.firebase.google.com/), crie um projeto novo, separado do InvestPro.
2. **Authentication > Sign-in method**: ative **E-mail/senha**.
3. **Firestore Database > Criar banco de dados**, em **modo de produção** (não use o modo de teste: ele deixa o banco aberto para qualquer pessoa).
4. **Firestore > Regras**: cole o conteúdo de `firestore.rules` e publique.
5. **Configurações do projeto > Seus apps > Web**: copie o `firebaseConfig` para `firebase-config.js`.
6. **Authentication > Users > Adicionar usuário**: crie o seu usuário e copie o UID.
7. **Firestore > Dados**: crie a coleção `usuarios`, com um documento cujo ID é o seu UID e o campo `papel` = `admin`.

Para liberar outra pessoa: crie o usuário em Authentication e um documento `usuarios/<UID>` com `papel` igual a `recepcao`, `clinico` ou `admin`. Quem entra sem esse documento vê uma tela com o próprio código (UID) para repassar à administração.

## Publicação

- **Firebase Hosting**: `npm i -g firebase-tools`, `firebase login`, `firebase use <id-do-projeto>`, `firebase deploy`. Publica o site e as regras.
- **GitHub Pages**: Settings > Pages > branch `main`, pasta raiz.

## Pendências antes de usar com dados reais

- Tela de administração de usuários (hoje os perfis são criados pelo Firebase Console).
- Sigilo do prontuário por área (hoje toda a equipe clínica lê todos os apontamentos).
- Adequação à LGPD (dados de saúde são dados sensíveis) e às regras do CFM para prontuário eletrônico.
- Cópia de segurança e exportação dos dados.

## Modelo de dados

| Coleção | Campos |
| --- | --- |
| `beneficiarios` | nome, cpf, nascimento, telefone, endereco, bairro, familia, recebeCesta, obs |
| `profissionais` | nome, especialidade, registro, telefone, dias[], cor, ativo, medico, duracaoPadrao |
| `agendamentos` | tipo (`consulta` ou `cesta`), data, inicio (min), duracao (min), beneficiarioId, profissionalId, status, obs, qtd |
| `prontuarios` | beneficiarioId, profissionalId, agendamentoId, quando, texto, autorId |
| `config/geral` | ong, msg |
| `usuarios` | papel (`admin`, `clinico`, `recepcao`), nome (opcional) |
