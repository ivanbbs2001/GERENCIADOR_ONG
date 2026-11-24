# GERENCIADOR_ONG

Agenda de atendimentos e entrega de cestas básicas para ONG.

## O que tem

- **Agenda** por dia (uma coluna por profissional, mais a coluna de cestas) e por semana. Clique em um horário vago para agendar; arraste para mudar horário, profissional ou dia; puxe a borda inferior para mudar a duração.
- **Beneficiários**: cadastro, histórico de atendimentos e cestas, taxa de não comparecimento e prontuário (apontamentos médicos). Telefone com máscara `(00) 00000-0000`, ou número internacional (sem máscara, sem o prefixo 55 no WhatsApp).
- **Inatividade**: beneficiário sem comparecimento há mais de 8 meses recebe o selo *Inativo* na lista. Comparecimento é consulta com chegada ou atendimento, ou cesta entregue. Se a pessoa já compareceu, o prazo conta do último comparecimento. Se nunca compareceu, conta do último registro passado (falta incluída; desmarcados não contam). Quem não tem nenhum registro passado não é marcado. O filtro *Só inativos* mostra só esses cadastros. A regra não altera o cadastro nem arquiva.
- **Profissionais**: especialidade, registro, dias de atendimento, duração padrão (30 min, ajustável por profissional).
- **Cestas**: entregas do mês, com marcação de entregue. Avisa quando a família já recebeu entrega no mês, e quando um beneficiário vinculado tem entrega no mesmo dia ou com menos de 30 dias de diferença. Os avisos não bloqueiam o agendamento.
- **Famílias e vínculos** (no cadastro do beneficiário): *Mesma família de* agrupa pessoas da mesma família (regra mensal de cestas). *Vínculos* liga pessoas que podem receber cesta juntas ou em datas próximas; o vínculo é gravado nos dois cadastros.
- **Relatórios**: por mês, com atendimentos, realizados, faltas, taxa de não comparecimento, cestas, famílias e beneficiários atendidos, por especialidade e por profissional. Exporta CSV (separado por ponto e vírgula, abre no Excel).
- **Backup**: na tela de Auditoria, o administrador baixa um JSON com beneficiários, profissionais, agendamentos, prontuários e usuários. A exportação fica registrada na auditoria. Guarde o arquivo com controle de acesso: contém dados de saúde.
- **WhatsApp**: link `wa.me` com os horários da semana do beneficiário. Texto editável em Ajustes. Na ficha do beneficiário, o botão *ENVIAR NO WHATS* abre a conversa com as orientações dos benefícios marcados (Bolsa Família, BPC/LOAS, Tarifa Social de Energia, Gás do Povo, Pé-de-Meia). A conversa só é enviada quando você confirmar no WhatsApp.
- **Busca geral**: campo no cabeçalho, ao lado de Relatórios. Procura beneficiários e profissionais.

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

- Auditoria gravada pelo próprio navegador: quem contorna o sistema pode deixar de registrar. A solução definitiva é uma Cloud Function que grave a auditoria no servidor.
- Tela de administração de usuários (hoje os perfis são criados pelo Firebase Console).
- Sigilo do prontuário por área (hoje toda a equipe clínica lê todos os apontamentos).
- Adequação à LGPD (dados de saúde são dados sensíveis) e às regras do CFM para prontuário eletrônico.
- Cópia de segurança automática. Hoje o backup é manual, pela tela de Auditoria. Backup agendado exige o plano Blaze e uma conta de serviço no Google Cloud.
- Validação de conflitos de horário, cestas por família e vínculos no servidor.
- Textos de benefícios (requisitos, documentos, passos e canais) são de referência geral. Conferir nas fontes oficiais antes de uso, e revisar periodicamente. Hoje os avisos são feitos no navegador e podem falhar com duas pessoas agendando ao mesmo tempo.

## Modelo de dados

| Coleção | Campos |
| --- | --- |
| `beneficiarios` | nome, cpf, nascimento, telefone, telefoneInternacional, telefoneDeContato, contatoNome, contatoRelacao, endereco, bairro, familia, familiaId (grupo da família), vinculos[] (IDs de outros beneficiários, gravados nos dois lados), recebeCesta, beneficios (mapa com bolsaFamilia, bpc, tarifaSocial, gasDoPovo, peDeMeia), arquivado, obs |
| `profissionais` | nome, especialidade, registro, telefone, dias[], cor, ativo, medico, duracaoPadrao |
| `agendamentos` | tipo (`consulta` ou `cesta`), data, inicio (min), duracao (min), beneficiarioId, profissionalId, status, obs, qtd, criadoEm (data em que o agendamento foi feito; não é a data da consulta) |
| `prontuarios` | beneficiarioId, profissionalId, agendamentoId, quando, texto, autorId |
| `config/geral` | ong, msg |
| `usuarios` | papel (`admin`, `clinico`, `recepcao`), nome (opcional) |
| `auditoria` | uid, email, papel, acao (`criar`, `alterar`, `excluir`, `exportar`), colecao, docId, campos, valores (só campos de situação e horário), quando (servidor) |
