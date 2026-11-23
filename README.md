# GERENCIADOR_ONG

Agenda de atendimentos e entrega de cestas básicas para ONG.

## O que tem

- **Agenda** por dia (uma coluna por profissional, mais a coluna de cestas) e por semana. Clique em um horário vago para agendar; arraste para mudar horário, profissional ou dia; puxe a borda inferior para mudar a duração.
- **Beneficiários**: cadastro, histórico de atendimentos e cestas, taxa de não comparecimento e prontuário (apontamentos médicos).
- **Profissionais**: especialidade, registro, dias de atendimento, duração padrão (15 min para médicos, 30 min para os demais).
- **Cestas**: entregas do mês, com marcação de entregue.
- **WhatsApp**: link `wa.me` com os horários da semana do beneficiário. Texto editável em Ajustes.

## Estado atual: protótipo

`index.html` é um arquivo único, sem etapa de build. Ele foi escrito para rodar como artefato do Claude e grava os dados pelo banco do artefato (`claude.use("db")`).

**Aberto fora do Claude (GitHub Pages, servidor próprio, arquivo local), ele entra em modo demonstração e não salva nada.** Para hospedar por conta própria é preciso trocar a camada de dados por um backend real. Os pontos de troca estão concentrados no objeto `store` e nas funções `init`, `subscribeAg`, `fetchHist` e `drawPront`.

## Pendências antes de usar com dados reais

- Backend próprio com login por usuário e perfis (recepção, clínico, administração).
- Sigilo do prontuário por profissional ou por área, trilha de auditoria e registros imutáveis.
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
