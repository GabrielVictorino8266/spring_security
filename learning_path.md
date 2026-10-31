# Trilha de Aprendizagem — Segurança de APIs com Spring Security

## Objetivo

Esta trilha orienta o estudo necessário para responder, com suas próprias palavras, à atividade **Segurança de APIs com Spring Security**, usando o projeto `workshop-microservices` como contexto. O objetivo não é implementar uma solução completa, mas compreender os conceitos, relacioná-los ao projeto e produzir um PDF argumentativo, identificado com nome e turma.

Ao final, você deverá ser capaz de:

- diferenciar autenticação de autorização;
- explicar como a `SecurityFilterChain` protege requisições;
- propor e justificar uma política de acesso baseada em papéis;
- distinguir respostas HTTP 401 e 403;
- discutir riscos de identidades descentralizadas em microserviços;
- citar fontes confiáveis sem apenas reproduzi-las.

## Como estudar

Para cada etapa, siga este ciclo:

1. Leia o material indicado.
2. Registre uma definição de duas ou três frases sem consultar a fonte.
3. Crie um exemplo relacionado ao workshop.
4. Responda às perguntas de verificação.
5. Revise sua explicação comparando-a com a documentação.

Ao encontrar algo que o projeto não esclareça, declare uma suposição. Exemplo: “Para esta análise, considero que o serviço de produtos oferece `GET /produtos`, `POST /produtos` e `DELETE /produtos/{id}`.”

## Etapa 0 — Conhecer o contexto

Antes de estudar segurança, examine o `workshop-microservices` e anote:

- quais microserviços existem;
- qual serviço administra produtos;
- quais operações de consulta, criação e exclusão estão disponíveis;
- se há API Gateway, serviço de identidade ou configuração de segurança;
- como os serviços se comunicam.

Não invente características como se já existissem. Diferencie claramente **o que foi observado** do que será usado como **hipótese para a atividade**.

### Evidência de aprendizagem

Produza um parágrafo descrevendo o fluxo hipotético de uma requisição de exclusão de produto, desde o cliente até o serviço responsável.

## Etapa 1 — Autenticação e autorização

### Conceitos

- **Autenticação** verifica quem está fazendo a requisição. Exemplos de credenciais incluem usuário e senha ou token de acesso.
- **Autorização** decide o que uma identidade autenticada pode fazer, considerando papéis ou autoridades, como `LEITOR` e `ADMIN`.

A ordem lógica é importante: primeiro o sistema determina a identidade; depois avalia suas permissões.

### Leituras

- [Authentication — Spring Security](https://docs.spring.io/spring-security/reference/features/authentication/)
- [Servlet Authentication Architecture — Spring Security](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)
- [Authorization Architecture — Spring Security](https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html)

### Aplicação ao workshop

Pense em dois momentos diferentes:

1. Um usuário apresenta credenciais para provar sua identidade.
2. O usuário tenta excluir um produto e o sistema verifica se ele possui o papel `ADMIN`.

### Verificação

- Uma senha correta garante acesso a qualquer operação? Por quê?
- Um usuário pode estar autenticado e ainda receber acesso negado?
- Em qual momento os papéis `LEITOR` e `ADMIN` são usados?

### Entrega parcial

Escreva dois parágrafos: um explicando autenticação e outro explicando autorização. Inclua um exemplo distinto de cada conceito no serviço de produtos.

## Etapa 2 — `SecurityFilterChain` e proteção das requisições

### Conceito

No suporte Servlet do Spring Security, a requisição passa por filtros de segurança antes de chegar ao controlador. A `SecurityFilterChain` reúne os filtros aplicáveis e permite configurar regras por caminho, método HTTP e condição de acesso. Filtros de autenticação devem atuar antes da decisão de autorização.

Uma configuração conceitual poderia expressar regras como:

```java
authorize.requestMatchers(HttpMethod.GET, "/produtos/**").permitAll();
authorize.requestMatchers(HttpMethod.POST, "/produtos/**").hasRole("ADMIN");
authorize.requestMatchers(HttpMethod.DELETE, "/produtos/**").hasRole("ADMIN");
authorize.anyRequest().authenticated();
```

O trecho serve apenas para visualizar as regras; a atividade não exige implementação.

### Leituras

- [Architecture — Spring Security](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [Authorize HTTP Requests — Spring Security](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html)
- [Securing a Web Application — guia introdutório gratuito](https://spring.io/guides/gs/securing-web/)

### Verificação

- Por que a ordem das regras pode alterar o resultado?
- Qual é a diferença entre `permitAll()`, `authenticated()` e `hasRole("ADMIN")`?
- O que deve acontecer se nenhuma regra específica corresponder à requisição?

### Entrega parcial

Desenhe este fluxo e explique cada decisão:

```text
requisição → filtros → autenticação → regra de autorização → controlador
```

## Etapa 3 — Política de acesso e menor privilégio

Construa uma matriz antes de escrever a justificativa:

| Operação | Método e rota assumidos | Visitante | LEITOR | ADMIN |
|---|---|---:|---:|---:|
| Consultar produtos | `GET /produtos` | Permitido | Permitido | Permitido |
| Criar produto | `POST /produtos` | Negado | Negado | Permitido |
| Excluir produto | `DELETE /produtos/{id}` | Negado | Negado | Permitido |

Esta é uma proposta, não uma verdade universal. A consulta pública facilita o acesso ao catálogo, enquanto criação e exclusão alteram dados e ficam restritas ao `ADMIN`. O papel `LEITOR` possui acesso somente de leitura, de acordo com seu nome. A decisão segue o **princípio do menor privilégio**: cada perfil recebe apenas as permissões necessárias.

### Questões para aprofundar

- Os produtos do workshop são públicos ou contêm informações internas?
- Um `LEITOR` deveria consultar um produto individual e uma lista completa?
- A criação poderia pertencer a um papel separado, como `EDITOR`?
- A exclusão deveria ser lógica para preservar histórico?

### Entrega parcial

Adapte a tabela às rotas reais do workshop e escreva uma justificativa curta para cada linha. Se mantiver a proposta acima, explique por que ela é adequada ao contexto.

## Etapa 4 — Credenciais e respostas 401 e 403

### Modelo mental

| Situação | Identidade válida? | Permissão suficiente? | Resposta esperada |
|---|---:|---:|---:|
| Sem credenciais em recurso protegido | Não | Não avaliada | `401 Unauthorized` |
| Credenciais inválidas ou expiradas | Não | Não avaliada | `401 Unauthorized` |
| Autenticado sem a permissão exigida | Sim | Não | `403 Forbidden` |
| Autenticado e autorizado | Sim | Sim | A operação prossegue |

Apesar do nome histórico de `401 Unauthorized`, ele indica, na prática, ausência de autenticação válida. O `403 Forbidden` indica que o servidor reconheceu a identidade, mas recusou a operação por falta de permissão.

No Spring Security, o `ExceptionTranslationFilter` transforma falhas de segurança em respostas HTTP. Problemas de autenticação acionam um `AuthenticationEntryPoint`; acesso negado a uma identidade autenticada é tratado por um `AccessDeniedHandler`.

### Leituras

- [Handling Security Exceptions — Spring Security](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-exceptiontranslationfilter)
- [401 Unauthorized — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/401)
- [403 Forbidden — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/403)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)

### Verificação

- Por que credenciais inválidas não deveriam produzir 403?
- Repetir uma requisição 403 com as mesmas credenciais resolveria o problema?
- Qual resposta um `LEITOR` autenticado receberia ao executar `DELETE /produtos/10`?

### Entrega parcial

Explique cada uma das três situações pedidas pela atividade e finalize com uma frase que contraste diretamente 401 e 403.

## Etapa 5 — Identidade em uma arquitetura de microserviços

Se cada microserviço mantiver usuários e senhas próprios, podem surgir:

- **duplicação e inconsistência:** alteração ou bloqueio de uma conta precisa ser repetido em vários bancos;
- **experiência ruim:** o usuário pode precisar de credenciais diferentes para cada serviço;
- **maior superfície de ataque:** vários serviços armazenam senhas e precisam implementar recuperação, bloqueio e proteção corretamente;
- **políticas divergentes:** requisitos de senha, papéis e expiração podem variar entre serviços;
- **auditoria difícil:** torna-se mais trabalhoso descobrir quem acessou diferentes serviços;
- **alto custo operacional:** provisionar ou remover um usuário exige coordenação entre sistemas.

Estude a ideia de um provedor de identidade centralizado, mas evite afirmar que centralização elimina todos os riscos. Ela reduz duplicação e facilita políticas consistentes, porém exige disponibilidade, proteção e integração adequadas.

### Leituras complementares

- [OAuth 2.0 Resource Server — Spring Security](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html)
- [OAuth 2.0 — visão geral do protocolo](https://oauth.net/2/)

### Verificação

- O que ocorre quando um funcionário desligado permanece ativo em um dos serviços?
- Como garantir a mesma política de senha em bancos independentes?
- Por que copiar uma senha entre serviços não resolve o problema arquitetural?

### Entrega parcial

Escolha pelo menos dois problemas e descreva causa, consequência e uma possível direção de solução. Relacione-os ao workshop, em vez de apresentar apenas definições genéricas.

## Etapa 6 — Produção do documento final

Estrutura sugerida para `docs/atividade.md` e para o PDF:

1. **Capa:** instituição, disciplina, título, nome, turma e data.
2. **Introdução:** objetivo e contexto do `workshop-microservices`.
3. **Autenticação e autorização:** conceitos e exemplos.
4. **Proteção das requisições:** papel da `SecurityFilterChain`.
5. **Política de acesso:** tabela e justificativas.
6. **Cenários de acesso:** ausência ou invalidade de credenciais, falta de permissão, 401 e 403.
7. **Decisão de arquitetura:** problemas de cadastros separados.
8. **Conclusão:** síntese do que foi aprendido.
9. **Fontes consultadas:** título e link de cada material realmente utilizado.

### Método de redação

- Use as fontes para validar conceitos, não para copiar parágrafos.
- Prefira frases curtas e exemplos ligados ao workshop.
- Diferencie fatos observados no projeto de decisões propostas.
- Não inclua código desnecessário no PDF.
- Depois de escrever, tente explicar oralmente cada seção sem ler. Se não conseguir, retorne ao tópico.

## Plano sugerido de sete sessões

| Sessão | Atividade | Resultado verificável |
|---:|---|---|
| 1 | Inspecionar o workshop e definir hipóteses | Mapa dos serviços e rotas |
| 2 | Estudar autenticação e autorização | Dois conceitos e dois exemplos próprios |
| 3 | Estudar filtros e regras HTTP | Explicação do fluxo da requisição |
| 4 | Construir a política de acesso | Tabela justificada |
| 5 | Estudar 401, 403 e tratamento de exceções | Três cenários explicados |
| 6 | Analisar identidade em microserviços | Dois ou mais problemas discutidos |
| 7 | Redigir, revisar e exportar | Markdown final e PDF conferido |

## Organização no GitHub

Uma estrutura enxuta pode ser usada para demonstrar o processo:

```text
spring-security-atividade/
├── README.md
├── learning_path.md
├── referencias.md
├── estudos/
│   ├── 01-autenticacao-autorizacao.md
│   ├── 02-security-filter-chain.md
│   ├── 03-status-401-403.md
│   └── 04-identidade-em-microservicos.md
└── docs/
    ├── atividade.md
    └── atividade.pdf
```

Commits progressivos ajudam a evidenciar a aprendizagem:

```text
docs: adiciona trilha de aprendizagem
docs: registra estudo sobre autenticação e autorização
docs: explica SecurityFilterChain no contexto do workshop
docs: define política de acesso aos produtos
docs: diferencia respostas 401 e 403
docs: discute identidade em microserviços
docs: adiciona versão final da atividade
```

No `README.md`, apresente o propósito acadêmico, os tópicos estudados, a estrutura do repositório e um link para o PDF. Não publique credenciais, tokens, dados pessoais desnecessários ou materiais restritos da disciplina.

## Checklist de revisão

- [ ] O texto está identificado com nome e turma.
- [ ] Todos os exemplos fazem sentido no contexto do workshop.
- [ ] Autenticação e autorização foram diferenciadas claramente.
- [ ] O papel da `SecurityFilterChain` foi explicado sem depender somente de código.
- [ ] A tabela contempla visitante, `LEITOR` e `ADMIN`.
- [ ] Todas as permissões foram justificadas.
- [ ] Ausência e invalidade de credenciais foram distinguidas de falta de permissão.
- [ ] 401 e 403 foram explicados corretamente.
- [ ] Pelo menos dois problemas de cadastros separados foram discutidos.
- [ ] A documentação oficial do Spring Security aparece nas fontes.
- [ ] Todas as fontes listadas foram realmente consultadas.
- [ ] O PDF foi aberto e conferido depois da exportação.

## Referências principais

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Securing a Web Application — Spring Guide](https://spring.io/guides/gs/securing-web/)
- [Authentication — Spring Security](https://docs.spring.io/spring-security/reference/features/authentication/)
- [Authorization Architecture — Spring Security](https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html)
- [Servlet Architecture — Spring Security](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [HTTP response status codes — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)

Use apenas as referências que efetivamente contribuírem para seu texto final e mantenha o título e o link, conforme solicitado na atividade.
