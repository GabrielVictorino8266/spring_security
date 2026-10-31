# Segurança de APIs com Spring Security

**Aluno:** [SEU NOME]  
**Turma:** [SUA TURMA]  
**Disciplina:** [NOME DA DISCIPLINA]  
**Data:** [DATA]

## Introdução

Este trabalho analisa a proteção de APIs no contexto do projeto `workshop-microservices`. Para tornar os exemplos objetivos, considera-se que o serviço responsável pelos produtos oferece as operações `GET /produtos`, `POST /produtos` e `DELETE /produtos/{id}`. Essas rotas devem ser ajustadas caso o projeto utilize nomes diferentes.

## 1. Autenticação e autorização

**Autenticação** é o processo de confirmar a identidade de quem realiza uma requisição. Em uma API, isso pode acontecer pela validação de usuário e senha ou de um token enviado no cabeçalho HTTP. No workshop, um exemplo seria o sistema receber um token e verificar se ele identifica um usuário válido antes de permitir o acesso a uma operação protegida.

**Autorização** acontece depois da autenticação e determina quais ações o usuário identificado pode executar. Por exemplo, um usuário com papel `LEITOR` pode consultar produtos, enquanto somente um usuário `ADMIN` pode excluí-los. Portanto, estar autenticado não significa possuir acesso a todas as funções do sistema.

## 2. Proteção das requisições

No Spring Security, as requisições passam por filtros de segurança antes de alcançar os controladores da aplicação. A `SecurityFilterChain` representa a cadeia de filtros aplicável e é configurada por meio de `HttpSecurity`. Nela, a aplicação pode definir quais rotas são públicas, quais exigem autenticação e quais dependem de papéis específicos.

As regras podem considerar tanto o endereço quanto o método HTTP. Assim, uma consulta com `GET /produtos` pode ser pública, enquanto `POST /produtos` e `DELETE /produtos/{id}` podem exigir o papel `ADMIN`. Quando uma requisição corresponde a uma regra, o Spring Security verifica a identidade e as autoridades do usuário antes de permitir que ela siga para o controlador. Por segurança, operações não contempladas explicitamente devem exigir autenticação ou ser negadas, evitando acesso acidental.

## 3. Política de acesso

| Operação | Visitante sem autenticação | LEITOR | ADMIN |
|---|---|---|---|
| Consultar produtos (`GET`) | Permitido | Permitido | Permitido |
| Criar produto (`POST`) | Negado | Negado | Permitido |
| Excluir produto (`DELETE`) | Negado | Negado | Permitido |

A consulta foi liberada para visitantes porque se considera que o catálogo de produtos contém informações públicas. O papel `LEITOR` também pode consultar, mas não pode modificar os dados. Criação e exclusão foram reservadas ao `ADMIN`, pois alteram o estado do sistema e uma exclusão pode causar perda de informações. Essa política segue o princípio do menor privilégio: cada perfil recebe somente os acessos necessários para cumprir sua função. Se os produtos contiverem informações internas, a consulta também deverá exigir autenticação.

## 4. Cenários de acesso: 401 e 403

Quando uma requisição chega a uma operação protegida **sem credenciais**, a API deve responder com `401 Unauthorized`, informando que é necessário autenticar-se. A mesma resposta é esperada quando as **credenciais são inválidas, expiradas ou não podem ser verificadas**, pois o sistema não conseguiu confirmar uma identidade válida.

Quando as credenciais são válidas, mas o usuário autenticado **não possui a permissão necessária**, a resposta deve ser `403 Forbidden`. Um exemplo seria um `LEITOR` autenticado tentar executar `DELETE /produtos/10`: sua identidade é conhecida, mas seu papel não autoriza a exclusão.

Assim, `401` representa ausência de autenticação válida, enquanto `403` representa falta de autorização para uma identidade já reconhecida. No Spring Security, falhas que exigem autenticação são encaminhadas a um `AuthenticationEntryPoint`; já a negação de acesso a um usuário autenticado é tratada por um `AccessDeniedHandler`.

## 5. Identidade em microserviços

Manter um cadastro de usuários e senhas em cada microserviço causaria duplicação e inconsistência. Uma alteração de senha ou o bloqueio de um usuário precisaria ser repetido em todos os serviços. Se uma atualização falhasse, uma conta desativada poderia continuar acessando parte do sistema.

Outro problema seria o aumento da superfície de ataque. Cada microserviço passaria a armazenar informações sensíveis e teria de implementar corretamente hash de senhas, recuperação de conta, bloqueio, auditoria e proteção contra tentativas de invasão. Uma falha em qualquer serviço poderia comprometer as credenciais dos usuários.

Também poderiam surgir políticas diferentes de senha e de papéis, múltiplos logins para a mesma pessoa e maior dificuldade para auditar acessos. Uma direção mais adequada é utilizar um serviço central de identidade ou provedor de autenticação. Os microserviços validam a identidade recebida e continuam responsáveis por aplicar suas próprias regras de autorização. Essa centralização reduz duplicações, embora o serviço de identidade também precise ser seguro e altamente disponível.

## Conclusão

O Spring Security separa a confirmação da identidade da decisão sobre permissões. A `SecurityFilterChain` permite aplicar essas decisões antes que a requisição chegue às operações da API. No workshop, a combinação de autenticação centralizada, papéis claros e uma política de menor privilégio reduz acessos indevidos e evita a duplicação de credenciais entre microserviços.

## Fontes consultadas

- **Spring Security Reference — Servlet Architecture.** <https://docs.spring.io/spring-security/reference/servlet/architecture.html>
- **Spring Security Reference — Authentication.** <https://docs.spring.io/spring-security/reference/features/authentication/>
- **Spring Security Reference — Authorization Architecture.** <https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html>
- **Spring Guide — Securing a Web Application.** <https://spring.io/guides/gs/securing-web/>
- **MDN Web Docs — 401 Unauthorized.** <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/401>
- **MDN Web Docs — 403 Forbidden.** <https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/403>
