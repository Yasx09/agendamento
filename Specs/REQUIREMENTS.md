# Sistema de Agendamento de Provas e Trabalhos

## 1. Visão geral

### 1.1 Propósito

O sistema tem como finalidade centralizar as datas de provas, trabalhos, apresentações e entregas de projetos em um calendário organizado por turma. Deve permitir que alunos e professores consultem os agendamentos e executem ações de acordo com suas permissões. Também deve identificar semanas com mais de duas avaliações na mesma turma e apresentar um aviso de sobrecarga.

### 1.2 Problema que o sistema resolve

Quando as datas das avaliações são divulgadas separadamente, os alunos podem ter dificuldade para visualizar a carga de atividades da semana. O sistema reúne essas informações em um único local para facilitar a consulta e ajudar a identificar a concentração de avaliações.

### 1.3 Objetivo principal

Disponibilizar um calendário por turma, com consulta, filtros e gerenciamento de agendamentos, mantendo regras de autorização e integridade dos dados no backend.

### 1.4 Princípios da solução

- O calendário é a página inicial do sistema.
- As ações disponíveis dependem do papel do usuário, da autoria do agendamento, da situação do registro e do vínculo do professor com a turma.
- As permissões são verificadas no backend; ocultar um botão no frontend não substitui a autorização da API.
- A sobrecarga semanal gera apenas um aviso e nunca bloqueia o cadastro.
- Agendamentos removidos por denúncia são preservados no banco de dados e identificados pela situação `Removido por denúncia`.

## 2. Usuários e limites do sistema

### 2.1 Aluno

O aluno pode:

- consultar o calendário e os detalhes das avaliações;
- filtrar agendamentos por turma, disciplina, tipo, situação e intervalo de datas;
- cadastrar um agendamento, que começa como `Não confirmado`;
- editar somente os próprios agendamentos enquanto estiverem `Não confirmado`;
- denunciar um agendamento, selecionando um dos motivos permitidos;
- excluir manualmente o próprio agendamento somente enquanto estiver `Não confirmado` e não tiver sido removido por denúncia.

O aluno não pode confirmar agendamentos, editar registros de outros usuários nem editar ou excluir registros confirmados ou removidos por denúncia.

### 2.2 Professor

O professor pode:

- consultar o calendário e os detalhes das avaliações;
- filtrar agendamentos;
- cadastrar agendamentos somente para turmas às quais está vinculado; esses registros começam como `Confirmado`;
- confirmar agendamentos `Não confirmado` de turmas às quais está vinculado;
- editar agendamentos das turmas às quais está vinculado, inclusive os confirmados;
- excluir agendamentos das turmas às quais está vinculado, desde que não estejam `Removido por denúncia`.

O professor não pode executar ações de gerenciamento em uma turma à qual não esteja vinculado.

### 2.3 Dependências externas e itens fora da responsabilidade do sistema

A aplicação pressupõe que outro serviço ou contexto já fornece a identidade do usuário, o papel (`Aluno` ou `Professor`), as turmas, as disciplinas e os vínculos entre professores e turmas. O backend deve validar o token de autenticação recebido e utilizar a identidade validada para autorizar as operações.

O sistema não deve implementar cadastro de usuários, login, recuperação de senha, gerenciamento de turmas, gerenciamento de disciplinas ou criação dos vínculos entre professores e turmas.

## 3. Requisitos funcionais

### RF01 — Visualizar calendário

O sistema deve apresentar o calendário como página inicial, organizando os agendamentos por data e turma. O usuário deve poder selecionar a turma e navegar entre períodos do calendário. Cada item exibido deve permitir identificar, no mínimo, a disciplina, o tipo da avaliação, a data e a situação.

**Critérios de aceite:**
- Ao abrir a aplicação, o usuário é direcionado ao calendário, não a uma dashboard.
- Os agendamentos são agrupados ou posicionados conforme a data.
- A turma selecionada é respeitada na consulta.
- A interface informa quando não há agendamentos para o período consultado.
- A interface apresenta uma mensagem compreensível se a consulta falhar.

### RF02 — Consultar detalhes do agendamento

O sistema deve permitir abrir os detalhes de um agendamento a partir do calendário ou da lista. Os detalhes devem apresentar turma, disciplina, tipo, data, descrição, conteúdo avaliado e situação.

**Critérios de aceite:**
- Todos os campos do agendamento são apresentados com rótulos compreensíveis.
- Um identificador inexistente resulta em mensagem de não encontrado.
- As ações disponíveis são exibidas conforme o papel, a autoria, a situação e o vínculo com a turma.

### RF03 — Cadastrar agendamento

O sistema deve permitir cadastrar um agendamento com os campos `turmaId`, `disciplinaId`, `tipo`, `data`, `descricao` e `conteudoCobrado`. O autor deve ser identificado a partir do usuário autenticado, e não por um valor confiado ao frontend.

**Critérios de aceite:**
- Os campos necessários são validados no backend.
- O tipo deve corresponder a um dos valores permitidos no RF04.
- O autor é associado com base na identidade autenticada.
- A situação inicial é determinada pelo backend conforme o papel do autor e o vínculo com a turma, de acordo com o RF05.
- O sistema retorna uma resposta de sucesso quando o registro é criado e uma mensagem adequada quando os dados são inválidos.

### RF04 — Validar tipos de avaliação

O sistema deve aceitar exclusivamente os seguintes tipos:

- `Prova`
- `Trabalho`
- `Apresentação`
- `Entrega de projeto`

Qualquer outro valor deve ser rejeitado pelo backend, mesmo que seja enviado diretamente à API.

### RF05 — Definir a situação inicial

O backend deve definir a situação inicial de acordo com o autor do cadastro:

- cadastro feito por aluno: `Não confirmado`;
- cadastro feito por professor vinculado à turma: `Confirmado`.

O frontend não pode escolher livremente a situação inicial nem enviar um valor que substitua a regra do backend.

### RF06 — Filtrar agendamentos

O sistema deve permitir filtrar a lista e o calendário pelos seguintes critérios:

- turma (`turmaId`);
- disciplina (`disciplinaId`);
- tipo de avaliação;
- situação;
- data inicial e data final do período consultado.

**Critérios de aceite:**
- Cada filtro pode ser aplicado à consulta.
- Os filtros podem ser combinados.
- O intervalo de datas deve respeitar as datas informadas.
- Valores inválidos devem gerar resposta de erro adequada, sem interromper o servidor.

### RF07 — Confirmar agendamento

O sistema deve permitir que somente um professor vinculado à turma confirme um agendamento cuja situação atual seja `Não confirmado`. Ao confirmar, a situação passa para `Confirmado`, e o sistema deve registrar qual professor realizou a confirmação, quando aplicável.

**Critérios de aceite:**
- Alunos não podem confirmar agendamentos.
- Professores sem vínculo com a turma não podem confirmar o registro.
- Um registro já confirmado ou removido por denúncia não pode ser confirmado novamente.
- Tentativas não permitidas devem ser rejeitadas pela API.

### RF08 — Denunciar agendamento

O sistema deve permitir que um aluno denuncie um agendamento e selecione um único motivo entre os seguintes:

- `Data incorreta`
- `Avaliação não existe`
- `Já foi remarcada`
- `Registro duplicado`

A denúncia deve registrar o agendamento, o aluno autenticado, o motivo e a data de criação.

**Critérios de aceite:**
- Somente usuários com papel de aluno podem registrar denúncias.
- O motivo deve pertencer à lista permitida.
- O aluno responsável é identificado pelo token validado.
- A denúncia deve estar associada a um agendamento existente.

### RF09 — Impedir denúncias duplicadas

O sistema deve permitir no máximo uma denúncia por aluno para cada agendamento. A regra deve ser validada na aplicação e garantida também por uma restrição única no banco de dados sobre a combinação `(agendamentoId, alunoId)`.

**Critérios de aceite:**
- Uma segunda denúncia do mesmo aluno para o mesmo agendamento é rejeitada.
- A repetição deve retornar `409 Conflict`.
- Alunos diferentes podem denunciar o mesmo agendamento.

### RF10 — Tratar denúncias

O sistema deve contar denúncias feitas por alunos distintos. Quando o agendamento alcançar três denúncias válidas de alunos diferentes, sua situação deve mudar para `Removido por denúncia` somente se ainda não estiver `Confirmado`.

A remoção por denúncia é lógica: o registro do agendamento e suas denúncias devem permanecer no banco de dados. Um agendamento confirmado não deve ser removido automaticamente por essa regra.

**Critérios de aceite:**
- A contagem considera alunos distintos, sem duplicar a contribuição de um mesmo aluno.
- Com três denúncias de alunos diferentes, um agendamento `Não confirmado` passa para `Removido por denúncia`.
- Um agendamento `Confirmado` permanece confirmado mesmo que receba três ou mais denúncias.
- Registros `Removido por denúncia` não podem ser editados ou excluídos manualmente.
- Registros removidos por denúncia não entram na contagem da carga semanal.

### RF11 — Editar agendamento

O sistema deve aplicar as seguintes regras de edição:

- aluno: pode editar somente um agendamento de sua autoria cuja situação seja `Não confirmado`;
- professor: pode editar agendamentos de turmas às quais está vinculado, inclusive os confirmados;
- qualquer usuário: não pode editar um registro `Removido por denúncia`.

A autorização deve ser verificada no backend a cada solicitação de edição.

### RF12 — Excluir agendamento

O sistema deve permitir exclusão manual conforme as seguintes regras:

- aluno: pode excluir somente um agendamento de sua autoria que esteja `Não confirmado` e não esteja `Removido por denúncia`;
- professor: pode excluir agendamentos de turmas às quais está vinculado, exceto os que estejam `Removido por denúncia`.

A exclusão manual autorizada remove o agendamento da consulta normal. Já a remoção por denúncia deve preservar o registro no banco e alterar sua situação, conforme o RF10.

### RF13 — Calcular carga semanal

O sistema deve contar os agendamentos da mesma turma cuja data esteja entre segunda-feira e sexta-feira da semana que contém a data consultada. Registros com situação `Removido por denúncia` não devem entrar na contagem.

O cálculo deve considerar a semana correspondente à data enviada na consulta, e não uma janela móvel de sete dias.

### RF14 — Avisar sobre sobrecarga

O sistema deve indicar sobrecarga quando houver mais de duas avaliações contabilizadas para a mesma turma entre segunda-feira e sexta-feira da semana consultada.

**Critérios de aceite:**
- Zero, uma ou duas avaliações não caracterizam sobrecarga.
- Três ou mais avaliações caracterizam sobrecarga.
- O aviso deve ser exibido na interface quando a API indicar `sobrecarga: true`.
- O aviso não bloqueia nem impede o envio do cadastro.
- A consulta de carga semanal deve utilizar a rota `GET /api/turmas/:turmaId/carga?data=AAAA-MM-DD`.

## 4. Requisitos não funcionais

### RNF01 — Backend

O backend deve ser implementado em Node.js com Express e disponibilizar uma API REST sob o prefixo `/api`.

### RNF02 — Persistência de dados

Os agendamentos e as denúncias devem ser armazenados em PostgreSQL. A aplicação deve manter os relacionamentos necessários entre esses registros.

### RNF03 — Mapeamento objeto-relacional

O Sequelize deve ser utilizado para definir os modelos, os relacionamentos e as operações de persistência no banco de dados.

### RNF04 — Segurança e autorização

O backend deve validar o JWT recebido e verificar papel, autoria e vínculo com a turma antes de executar operações protegidas. Dados sensíveis à autorização, como `autorId`, situação inicial e professor responsável pela confirmação, devem ser determinados ou validados no servidor.

### RNF05 — Integridade dos dados

Os valores de tipo, situação e motivo de denúncia devem respeitar as listas definidas neste documento. A unicidade de denúncia por aluno e agendamento deve ser garantida no banco de dados, além da validação na aplicação.

### RNF06 — Responsividade

As páginas devem se adaptar a telas de desktop, tablet e celular, mantendo disponíveis as funções de consulta e gerenciamento previstas no escopo.

### RNF07 — Acessibilidade e clareza

Os formulários devem ter rótulos visíveis, os elementos interativos devem apresentar foco identificável e os textos devem ter contraste adequado. Situações e alertas não podem depender apenas de cores para comunicar seu significado.

### RNF08 — Tratamento de erros

A API deve responder com códigos HTTP apropriados para sucesso, dados inválidos, ausência de autenticação, falta de permissão, recurso inexistente, conflito e falhas inesperadas. Respostas de erro não devem expor detalhes internos do servidor.

### RNF09 — Separação de responsabilidades

O frontend deve ser responsável pela apresentação e interação; o backend deve concentrar as validações definitivas, regras de negócio e autorização; o PostgreSQL deve persistir os dados. O frontend não deve substituir as regras implementadas na API.

### RNF10 — Testabilidade

As rotas e regras de negócio do backend devem ser implementadas e testadas antes do início do frontend. A primeira entrega técnica deve ser uma API executável, incluindo a rota `GET /api/health`.

## 5. Regras de negócio consolidadas

1. Agendamentos criados por alunos começam como `Não confirmado`.
2. Agendamentos criados por professores vinculados começam como `Confirmado`.
3. Somente professores vinculados à turma podem confirmar agendamentos não confirmados.
4. Um aluno pode denunciar determinado agendamento apenas uma vez.
5. Três denúncias de alunos distintos removem logicamente apenas agendamentos ainda não confirmados.
6. Alunos editam somente os próprios agendamentos não confirmados.
7. Professores vinculados podem editar agendamentos da turma, inclusive confirmados, mas não os removidos por denúncia.
8. Alunos podem excluir os próprios agendamentos não confirmados; professores vinculados podem excluir agendamentos da turma. Nenhum deles pode excluir manualmente um registro removido por denúncia.
9. A sobrecarga corresponde a mais de duas avaliações não removidas por denúncia, na mesma turma, de segunda a sexta-feira.
10. O aviso de sobrecarga não bloqueia o cadastro.
11. O backend é responsável por verificar a identidade, o papel, a autoria e o vínculo com a turma.
12. A exclusão manual e a remoção por denúncia são operações diferentes: a segunda preserva o registro e suas denúncias.

## 6. Páginas e escopo do protótipo

### 6.1 Calendário da turma

Página inicial da aplicação. Deve permitir selecionar uma turma, navegar entre períodos e visualizar as avaliações organizadas por data. Deve oferecer acesso aos detalhes e aos filtros definidos no RF06.

### 6.2 Lista de avaliações

Página de consulta em formato de lista, com os mesmos filtros previstos para o calendário. Cada item deve apresentar informações resumidas e permitir abrir os detalhes.

### 6.3 Detalhes da avaliação

Página ou painel acessado a partir do calendário ou da lista. Deve apresentar os dados completos e as ações permitidas para o usuário autenticado.

### 6.4 Cadastro e edição de avaliação

Formulário com turma, disciplina, tipo, data, descrição e conteúdo avaliado. Deve apresentar erros de validação e, quando aplicável, o aviso de sobrecarga semanal.

### 6.5 Denunciar avaliação

Formulário ou modal acessado a partir dos detalhes do agendamento. Deve permitir selecionar somente um dos quatro motivos definidos no RF08.

### 6.6 Regras gerais de interface

- O calendário deve ser a página inicial.
- Não deve existir dashboard independente.
- Não devem ser adicionadas páginas ou funcionalidades sem correspondência com os requisitos.
- A interface deve apresentar estados de carregamento, lista vazia e erro nas consultas pertinentes.
- As ações exibidas no frontend devem refletir as permissões conhecidas, mas a API continua responsável pela autorização definitiva.

## 7. Fora do escopo

- Cadastro, login, recuperação de senha e gerenciamento de usuários.
- Criação ou gerenciamento de turmas, disciplinas e vínculos entre professores e turmas.
- Lançamento de notas ou correção de avaliações.
- Integração com calendários externos.
- Notificações automáticas.
- Aprovação de agendamentos pela coordenação.
- Bloqueio de cadastro por sobrecarga semanal.
- Remoção automática de agendamentos confirmados por denúncias.
- Tipos de avaliação, situações ou motivos de denúncia diferentes dos definidos neste documento.
- Dashboard independente, relatórios avançados ou funcionalidades não descritas nos requisitos.

## 8. Critérios gerais de aceite

O sistema será considerado coerente com os requisitos quando:

- o calendário for a página inicial e apresentar os agendamentos por turma e data;
- os filtros definidos no RF06 estiverem disponíveis;
- os dados obrigatórios e os valores permitidos forem validados;
- a situação inicial for determinada pelo backend;
- as permissões de aluno e professor forem verificadas em todas as operações protegidas;
- a confirmação estiver limitada a professores vinculados e a registros não confirmados;
- denúncias duplicadas forem impedidas pela aplicação e pelo banco;
- três denúncias de alunos distintos removerem logicamente apenas registros não confirmados;
- registros removidos por denúncia forem preservados e excluídos da contagem de carga semanal;
- a sobrecarga for calculada de segunda a sexta-feira e gerar apenas um aviso;
- as páginas forem responsivas e apresentarem mensagens compreensíveis de carregamento, ausência de dados e erro;
- as rotas e regras do backend forem testadas antes do desenvolvimento do frontend;
- o protótipo e a implementação não incluírem funcionalidades fora do escopo.
