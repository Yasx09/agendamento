# Sistema de Agendamento de Provas e Trabalhos

## Regra de execução

Desenvolver primeiro o backend e testar suas rotas. O frontend só começa depois que as rotas necessárias e as regras de negócio estiverem implementadas e testadas. A primeira entrega técnica deve ser uma API executável e testável, não uma dashboard.

## Fase 1 — Backend inicial e primeira rota testável

- [ ] **TASK01 — Criar o projeto backend Node.js:** configurar `package.json`, scripts de execução e de testes.

- [ ] **TASK02 — Configurar Express:** criar o servidor e o tratamento básico de requisições e erros.

- [ ] **TASK03 — Criar `GET /api/health`:** retornar `200 OK` e `{ "status": "ok" }` para verificar que a API está funcionando.

- [ ] **TASK04 — Testar a primeira rota:** executar uma requisição a `GET /api/health` e registrar o resultado esperado. Não iniciar o frontend antes desse teste.

Exemplo de teste da rota inicial:

```http

GET /api/health

```

Resposta esperada:

```json

{

  "status": "ok"

}

```

## Fase 2 — Banco de dados e modelos do backend

- [ ] **TASK05 — Configurar a conexão PostgreSQL:** utilizar variáveis de ambiente para os dados de conexão.

- [ ] **TASK06 — Instalar e configurar Sequelize:** validar a conexão com o banco.

- [ ] **TASK07 — Criar o model `Agendamento`:** implementar os campos e os valores permitidos descritos no `DESIGN.md`.

- [ ] **TASK08 — Criar o model `Denuncia`:** implementar os campos e os motivos permitidos.

- [ ] **TASK09 — Configurar relacionamentos e restrição única:** relacionar denúncia e agendamento e impedir duplicidade por `(agendamentoId, alunoId)`.

- [ ] **TASK10 — Testar persistência:** criar e consultar registros de teste no PostgreSQL.

## Fase 3 — Autenticação e permissões no backend

- [ ] **TASK11 — Implementar middleware JWT:** validar o token fornecido pelo sistema de autenticação existente.

- [ ] **TASK12 — Implementar verificação de papel:** identificar operações permitidas para aluno e professor.

- [ ] **TASK13 — Implementar validação do vínculo do professor:** verificar se o professor está vinculado à turma antes de permitir ações.

- [ ] **TASK14 — Testar os middlewares:** verificar token ausente/inválido (`401`) e usuário sem permissão (`403`).

## Fase 4 — Rotas de consulta e cadastro de agendamentos

- [ ] **TASK15 — Implementar `GET /api/agendamentos`:** listar registros e aplicar os filtros definidos no `DESIGN.md`.

- [ ] **TASK16 — Testar listagem e filtros:** testar consulta sem filtros e com cada filtro disponível.

- [ ] **TASK17 — Implementar `GET /api/agendamentos/:id`:** retornar os detalhes ou `404`.

- [ ] **TASK18 — Testar consulta por ID:** testar identificador existente e inexistente.

- [ ] **TASK19 — Implementar `POST /api/agendamentos`:** validar campos, obter o autor do token e definir a situação inicial no backend.

- [ ] **TASK20 — Testar cadastro:** testar aluno, professor vinculado, campo obrigatório ausente e tipo inválido.

## Fase 5 — Rotas de edição, exclusão e confirmação

- [ ] **TASK21 — Implementar `PUT /api/agendamentos/:id`:** aplicar as regras de edição de aluno e professor.

- [ ] **TASK22 — Testar edição:** testar autoria, situação e vínculo do professor.

- [ ] **TASK23 — Implementar `DELETE /api/agendamentos/:id`:** permitir exclusão pelo aluno autor somente enquanto não confirmado ou pelo professor vinculado à turma, preservando registros removidos por denúncia.

- [ ] **TASK24 — Testar exclusão:** testar aluno autor com registro não confirmado, aluno sem permissão, professor vinculado ou não vinculado, registro confirmado e bloqueio de exclusão manual de registro removido por denúncia.

- [ ] **TASK25 — Implementar `PATCH /api/agendamentos/:id/confirmar`:** permitir confirmação somente por professor vinculado e em registro não confirmado.

- [ ] **TASK26 — Testar confirmação:** testar papel, vínculo e situação atual.

## Fase 6 — Rotas de denúncia

- [ ] **TASK27 — Implementar `POST /api/agendamentos/:id/denuncias`:** validar motivo e registrar o aluno autenticado.

- [ ] **TASK28 — Impedir denúncia duplicada:** usar a restrição única e responder `409 Conflict` em caso de repetição.

- [ ] **TASK29 — Implementar a regra das três denúncias:** contar alunos distintos e mudar a situação somente se o registro ainda não estiver confirmado.

- [ ] **TASK30 — Implementar `GET /api/agendamentos/:id/denuncias`:** retornar quantidade e dados necessários.

- [ ] **TASK31 — Testar denúncias:** testar motivos inválidos, repetição, três alunos diferentes e preservação de agendamento confirmado.

## Fase 7 — Rota de carga semanal

- [ ] **TASK32 — Implementar o serviço de carga semanal:** contar agendamentos da mesma turma que não estejam `Removido por denúncia`, de segunda a sexta-feira.

- [ ] **TASK33 — Implementar `GET /api/turmas/:turmaId/carga?data=AAAA-MM-DD`:** retornar período, quantidade e indicador de sobrecarga.

- [ ] **TASK34 — Testar a rota de carga:** testar semanas com zero, duas e mais de duas avaliações e confirmar que agendamentos removidos por denúncia não entram na contagem.

- [ ] **TASK35 — Garantir que a sobrecarga não bloqueie o cadastro:** testar `POST /api/agendamentos` quando a turma já tiver mais de duas avaliações na semana e confirmar que o agendamento é criado normalmente. A carga semanal deve ser consultada pela rota `GET /api/turmas/:turmaId/carga?data=AAAA-MM-DD`.

## Fase 8 — Testes finais da API

- [ ] **TASK36 — Executar testes de todas as rotas:** validar métodos, parâmetros, autenticação e respostas HTTP.

- [ ] **TASK37 — Executar testes das regras de negócio:** verificar situação inicial, edição, exclusão, confirmação, denúncias e sobrecarga.

- [ ] **TASK38 — Corrigir falhas do backend:** repetir os testes depois das correções e confirmar que as rotas estão prontas para consumo.

## Fase 9 — Configuração do frontend

Começar esta fase somente após concluir os testes do backend.

- [ ] **TASK39 — Criar projeto React + Vite.**

- [ ] **TASK40 — Configurar React Router:** criar as rotas das páginas de calendário, lista e cadastro/edição.

- [ ] **TASK41 — Configurar Axios:** definir a base da API e enviar o token existente.

- [ ] **TASK42 — Criar o serviço frontend de agendamentos:** organizar chamadas para as rotas já testadas.

## Fase 10 — Página principal: calendário

- [ ] **TASK43 — Criar `CalendarioPage.jsx`:** implementar o calendário como página inicial.

- [ ] **TASK44 — Criar o componente `Calendario`:** mostrar avaliações por data e permitir navegar pelo período.

- [ ] **TASK45 — Criar `FiltrosAgendamento`:** implementar filtros por turma, disciplina, tipo, situação e período.

- [ ] **TASK46 — Criar `AgendamentoCard` e `StatusBadge`:** mostrar resumo e situação textual.

- [ ] **TASK47 — Integrar calendário com `GET /api/agendamentos`:** carregar os dados reais da API.

- [ ] **TASK48 — Implementar estados de carregamento, vazio e erro:** apresentar mensagens claras para cada estado.

## Fase 11 — Lista e detalhes da avaliação

- [ ] **TASK49 — Criar `AvaliacoesPage.jsx`:** mostrar as avaliações em lista, usando filtros.

- [ ] **TASK50 — Criar `DetalhesAgendamento`:** apresentar os dados completos de uma avaliação.

- [ ] **TASK51 — Integrar consulta por ID:** consumir `GET /api/agendamentos/:id`.

- [ ] **TASK52 — Exibir ações conforme permissão:** mostrar editar, excluir, confirmar e denunciar somente quando aplicável, mantendo a validação no backend.

## Fase 12 — Cadastro e edição

- [ ] **TASK53 — Criar `AgendamentoFormPage.jsx` e `FormAgendamento`:** incluir todos os campos previstos nos requisitos.

- [ ] **TASK54 — Validar campos no frontend:** informar campos obrigatórios e tipos aceitos, sem substituir a validação do backend.

- [ ] **TASK55 — Integrar cadastro e edição:** consumir `POST /api/agendamentos` e `PUT /api/agendamentos/:id`.

- [ ] **TASK56 — Criar `AlertaSobrecarga`:** mostrar o aviso recebido da API, sem impedir o envio do formulário.

## Fase 13 — Confirmação, denúncia e exclusão no frontend

- [ ] **TASK57 — Integrar confirmação:** consumir `PATCH /api/agendamentos/:id/confirmar`.

- [ ] **TASK58 — Criar `ModalDenuncia`:** permitir selecionar apenas os quatro motivos definidos.

- [ ] **TASK59 — Integrar denúncia:** consumir as rotas de denúncia e apresentar sucesso ou erro.

- [ ] **TASK60 — Integrar exclusão:** consumir `DELETE /api/agendamentos/:id` e atualizar a interface após sucesso.

## Fase 14 — Visual, responsividade e protótipo

- [ ] **TASK61 — Aplicar a paleta e tipografia do protótipo:** seguir as cores e padrões documentados no `DESIGN.md`.

- [ ] **TASK62 — Ajustar espaçamentos e consistência dos componentes.**

- [ ] **TASK63 — Adaptar as páginas para desktop, tablet e celular.**

- [ ] **TASK64 — Revisar acessibilidade:** rótulos, contraste, foco e mensagens compreensíveis.

- [ ] **TASK65 — Comparar as páginas com o protótipo:** manter somente telas e elementos dentro do escopo definido.

## Fase 15 — Validação final

- [ ] **TASK66 — Testar o fluxo do aluno:** consultar, filtrar, cadastrar, editar quando permitido e denunciar.

- [ ] **TASK67 — Testar o fluxo do professor:** consultar, cadastrar, confirmar, editar e excluir registros da turma vinculada.

- [ ] **TASK68 — Testar o alerta de sobrecarga:** verificar que aparece quando necessário e não bloqueia o cadastro.

- [ ] **TASK69 — Revisar o escopo:** retirar dashboard independente e funcionalidades que não estejam em `REQUIREMENTS.md`.

- [ ] **TASK70 — Executar a validação integrada:** testar backend e frontend em conjunto e corrigir problemas restantes.

## Critérios de conclusão

- [ ] API Node.js/Express conectada ao PostgreSQL por Sequelize.

- [ ] Rotas documentadas, implementadas e testadas antes do frontend.

- [ ] Permissões e regras de negócio verificadas no backend.

- [ ] Frontend integrado à API e iniciado pelo calendário.

- [ ] Protótipo coerente com `REQUIREMENTS.md` e `DESIGN.md`.

- [ ] Nenhuma dashboard independente ou funcionalidade fora do escopo.
