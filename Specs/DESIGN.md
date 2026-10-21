# Sistema de Agendamento de Provas e Trabalhos

## 1. Propósito e solução

O sistema organiza as avaliações de cada turma em um calendário e permite consultar, cadastrar, confirmar, editar, excluir ou denunciar agendamentos conforme as permissões do usuário. A página principal é o calendário, não uma dashboard.

A aplicação será dividida em frontend, API backend e banco de dados. As regras de negócio e as permissões serão verificadas no backend, mesmo quando o frontend ocultar ações não permitidas.

## 2. Arquitetura e tecnologias

- **Frontend:** React + Vite.

- **Navegação frontend:** React Router.

- **Comunicação HTTP:** Axios.

- **Backend API:** Node.js + Express.

- **Banco de dados:** PostgreSQL.

- **ORM:** Sequelize.

- **Autenticação:** JWT validado pelo backend. A aplicação pressupõe que o token e os dados do usuário já são fornecidos; não implementa cadastro ou login.

### Organização de pastas prevista

```text

backend/

  src/

    app.js

    config/database.js

    models/Agendamento.js

    models/Denuncia.js

    routes/agendamentos.routes.js

    routes/turmas.routes.js

    controllers/agendamentos.controller.js

    controllers/denuncias.controller.js

    controllers/carga.controller.js

    middlewares/auth.middleware.js

    middlewares/error.middleware.js

    services/agendamento.service.js

    services/denuncia.service.js

    services/carga.service.js

frontend/

  src/

    pages/CalendarioPage.jsx

    pages/AvaliacoesPage.jsx

    pages/AgendamentoFormPage.jsx

    components/Calendario.jsx

    components/FiltrosAgendamento.jsx

    components/AgendamentoCard.jsx

    components/DetalhesAgendamento.jsx

    components/FormAgendamento.jsx

    components/ModalDenuncia.jsx

    components/AlertaSobrecarga.jsx

    components/StatusBadge.jsx

    components/LoadingState.jsx

    components/EmptyState.jsx

    components/ErrorMessage.jsx

    services/api.js

    services/agendamentos.js

```

## 3. Contrato da API REST

Prefixo de todas as rotas: `/api`. As rotas de agendamento exigem autenticação. Os testes das rotas devem ser feitos no backend antes do desenvolvimento do frontend.

O backend deve obter `autorId` e papel do usuário a partir da identidade validada. Não deve confiar em valores de autoria, confirmação ou permissões enviados pelo cliente.

### 3.1 Rota de verificação do servidor

\| Método e rota | Uso | Resultado esperado |

\|---|---|---|

\| `GET /api/health` | Verificar se a API está disponível | `200 OK` com estado `ok` |

Exemplo de resposta:

```json

{

  "status": "ok"

}

```

### 3.2 Agendamentos

\| Método e rota | Uso | Regras e respostas principais |

\|---|---|---|

\| `GET /api/agendamentos` | Listar agendamentos | Aceita `turmaId`, `disciplinaId`, `tipo`, `situacao`, `dataInicio` e `dataFim`. Retorna `200`. |

\| `GET /api/agendamentos/:id` | Consultar detalhes | Retorna `200` ou `404` se o registro não existir. |

\| `POST /api/agendamentos` | Cadastrar agendamento | Valida os campos e o tipo; define autor e situação no servidor. Retorna `201` ou `400`. |

\| `PUT /api/agendamentos/:id` | Editar agendamento | Verifica autoria, situação e vínculo do professor. Retorna `200`, `403`, `404` ou `400`. |

\| `DELETE /api/agendamentos/:id` | Excluir agendamento | Aluno autor: permitido somente enquanto não confirmado e não removido por denúncia. Professor vinculado: permitido, exceto para registro removido por denúncia. Retorna `204`, `403`, `404` ou `409`. |

Exemplo de corpo enviado para `POST /api/agendamentos`:

```json

{

  "turmaId": 12,

  "disciplinaId": 4,

  "tipo": "Prova",

  "data": "2026-11-10",

  "descricao": "Avaliação do bimestre",

  "conteudoCobrado": "Capítulos 3 e 4"

}

```

A situação inicial é definida pelo backend: `Não confirmado` para aluno e `Confirmado` para professor vinculado. O campo `autorId` não deve ser aceito como fonte confiável do corpo da requisição.

### 3.3 Confirmação

\| Método e rota | Uso | Regras e respostas principais |

\|---|---|---|

\| `PATCH /api/agendamentos/:id/confirmar` | Confirmar um agendamento | Somente professor vinculado e registro não confirmado. Retorna `200`, `403`, `404` ou `409`. |

### 3.4 Denúncias

\| Método e rota | Uso | Regras e respostas principais |

\|---|---|---|

\| `POST /api/agendamentos/:id/denuncias` | Registrar denúncia | Somente aluno; motivo permitido; uma denúncia por aluno e agendamento. Retorna `201`, `400`, `403` ou `409`. |

\| `GET /api/agendamentos/:id/denuncias` | Consultar denúncias | Retorna a quantidade e os dados necessários à regra de três denúncias. |

Motivos permitidos: `Data incorreta`, `Avaliação não existe`, `Já foi remarcada` e `Registro duplicado`.

Ao registrar uma denúncia, o serviço conta denúncias de alunos distintos. Com três denúncias válidas, muda a situação para `Removido por denúncia` somente se o agendamento ainda não estiver confirmado. O registro não é apagado do banco. Registros nessa situação não podem ser editados nem excluídos manualmente e não entram na contagem da carga semanal.

### 3.5 Carga semanal

\| Método e rota | Uso | Regras e respostas principais |

\|---|---|---|

\| `GET /api/turmas/:turmaId/carga?data=AAAA-MM-DD` | Consultar carga semanal | Conta avaliações da turma que não estejam `Removido por denúncia`, de segunda a sexta-feira da semana da data informada. |

Exemplo de resposta:

```json

{

  "turmaId": 12,

  "inicioSemana": "2026-11-09",

  "fimSemana": "2026-11-13",

  "quantidadeAvaliacoes": 3,

  "sobrecarga": true

}

```

`sobrecarga` será verdadeiro quando houver mais de duas avaliações. A resposta é usada para mostrar um aviso, sem bloquear o cadastro.

### 3.6 Respostas HTTP comuns

- `200 OK`: consulta ou atualização concluída.

- `201 Created`: registro criado.

- `204 No Content`: exclusão concluída.

- `400 Bad Request`: campos ou valores inválidos.

- `401 Unauthorized`: token ausente ou inválido.

- `403 Forbidden`: usuário sem permissão.

- `404 Not Found`: registro inexistente.

- `409 Conflict`: denúncia duplicada ou situação incompatível.

- `500 Internal Server Error`: erro inesperado, sem expor detalhes internos.

## 4. Modelo de dados

### Agendamento

- `id`: chave primária.

- `turmaId`: referência à turma existente.

- `disciplinaId`: referência à disciplina existente.

- `autorId`: usuário que criou o registro.

- `tipo`: Prova, Trabalho, Apresentação ou Entrega de projeto.

- `data`: data da avaliação.

- `descricao`: descrição ou instruções.

- `conteudoCobrado`: conteúdo avaliado.

- `situacao`: Confirmado, Não confirmado ou Removido por denúncia.

- `confirmadoPor`: professor que confirmou, quando aplicável.

- `createdAt` e `updatedAt`: datas de criação e atualização.

### Denuncia

- `id`: chave primária.

- `agendamentoId`: agendamento denunciado.

- `alunoId`: aluno responsável.

- `motivo`: um dos quatro motivos permitidos.

- `createdAt`: data da denúncia.

Criar uma restrição única para a combinação `(agendamentoId, alunoId)`. Um agendamento pode ter várias denúncias; cada denúncia pertence a um agendamento e identifica o aluno responsável.

## 5. Design do frontend por página

### Página 1 — Calendário da turma (`CalendarioPage.jsx`)

**Objetivo:** ser a página inicial e mostrar as avaliações organizadas por data.

Componentes:

- `Calendario`: navegação entre mês/semana e exibição das avaliações por data.

- `FiltrosAgendamento`: seleção de turma, disciplina, tipo, situação e período.

- `AgendamentoCard`: resumo da avaliação selecionada ou listada.

- `StatusBadge`: mostra a situação por texto.

- `LoadingState`, `EmptyState` e `ErrorMessage`: estados de carregamento, lista vazia e falha.

Integração: `GET /api/agendamentos` com filtros.

### Página 2 — Lista de avaliações (`AvaliacoesPage.jsx`)

**Objetivo:** consultar avaliações da turma em formato de lista.

Componentes:

- `FiltrosAgendamento`: filtrar os registros.

- `AgendamentoCard`: exibir disciplina, tipo, data e situação.

- `StatusBadge`: indicar a situação.

- `LoadingState`, `EmptyState` e `ErrorMessage`.

Integração: `GET /api/agendamentos`.

### Página 3 — Detalhes da avaliação

Pode ser apresentada como uma página ou painel de detalhes aberto a partir do calendário/lista, sem criar um dashboard. As ações exibidas na interface são apenas conveniência visual; a autorização definitiva é sempre validada pela API.

Componentes:

- `DetalhesAgendamento`: exibe turma, disciplina, tipo, data, descrição, conteúdo e situação.

- Ações de editar, excluir e confirmar, exibidas conforme papel, autoria e situação.

- Ação de denunciar, disponível para aluno.

- `ModalDenuncia`: seleção do motivo da denúncia.

Integrações: `GET /api/agendamentos/:id`, `PUT /api/agendamentos/:id`, `DELETE /api/agendamentos/:id`, `PATCH /api/agendamentos/:id/confirmar` e rotas de denúncia.

### Página 4 — Cadastro/edição (`AgendamentoFormPage.jsx`)

Componentes:

- `FormAgendamento`: turma, disciplina, tipo, data, descrição e conteúdo avaliado.

- `AlertaSobrecarga`: aviso quando a API informar mais de duas avaliações na semana.

- `ErrorMessage`: apresenta erros de validação ou da API.

Integrações: `POST /api/agendamentos`, `PUT /api/agendamentos/:id` e `GET /api/turmas/:turmaId/carga?data=AAAA-MM-DD`.

O alerta não bloqueia o envio. As validações definitivas permanecem no backend.

## 6. Identidade visual do protótipo

Paleta definida no projeto:

- `#010505`

- `#063730`

- `#097164`

- `#25F4D8`

- `#C20A0A`

- `#610505`

- `#FFFFFF`

Usar tipografia sem serifa, hierarquia visual clara, espaçamento consistente e layout responsivo. Situações e alertas devem conter texto, não depender somente da cor.

## 7. Critérios de aceite técnico

- O backend Node.js/Express está separado do frontend.

- As rotas descritas neste documento podem ser chamadas e testadas independentemente das páginas.

- Cada rota tem método, caminho, finalidade, validações e respostas esperadas documentadas.

- Os testes das rotas e das regras de negócio são executados antes do desenvolvimento do frontend.

- O frontend consome a API, em vez de duplicar regras de negócio.

- As páginas e os componentes implementam somente os requisitos deste documento.
