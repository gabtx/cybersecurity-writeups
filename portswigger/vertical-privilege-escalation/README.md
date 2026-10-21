# Broken Access Control: Vertical Privilege Escalation

**Plataforma:** PortSwigger Web Security Academy  
**Laboratório:** [User role controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter)  
**Nível:** Apprentice  
**Ferramenta:** Burp Suite Community Edition  
**Resultado:** Resolvido (ambiente educacional autorizado)

## Resumo

Uma aplicação vulnerável confiava no cookie `Admin`, controlado pelo navegador, para determinar privilégios administrativos. Alterando `Admin=false` para `Admin=true`, foi possível acessar o painel `/admin` e executar uma operação restrita usando uma sessão de usuário comum.

## Passo a passo e evidências observadas

1. Autenticação no laboratório com a conta de teste `wiener`.
2. Captura de `GET /my-account?id=wiener` no **Proxy → HTTP history**.
3. Envio da requisição ao **Repeater** e alteração de `Admin=false` para `Admin=true`, mantendo a sessão válida. Resposta: **HTTP/2 200 OK**.
4. Envio de `GET /admin` com o cookie modificado. A resposta renderizada mostrou o painel administrativo e os usuários `wiener` e `carlos`, com links de exclusão.
5. Envio, somente no laboratório, de `GET /admin/delete?username=carlos`. Resposta: **HTTP/2 302 Found**, com `Location: /admin`.
6. A resposta seguinte confirmou **User deleted successfully!**, e o laboratório exibiu **Congratulations, you solved the lab!**.

### Requisições ilustrativas (sem tokens reais)

```http
GET /admin HTTP/2
Host: <dominio-do-laboratorio>
Cookie: Admin=true; session=<REDACTED>
```

```http
GET /admin/delete?username=carlos HTTP/2
Host: <dominio-do-laboratorio>
Cookie: Admin=true; session=<REDACTED>
```

## Impacto

Escalada vertical de privilégios e acesso não autorizado a operações administrativas. Neste laboratório, foi possível excluir uma conta de teste.

**Categoria:** Broken Access Control · **CWE-863:** Incorrect Authorization.

## Causa raiz

O backend confiava em um indicador de privilégio fornecido pelo cliente, em vez de consultar e validar as permissões de uma fonte confiável no servidor.

## Recomendações

- Validar a autorização **no servidor**, em todas as rotas e ações sensíveis.
- Não usar cookies editáveis pelo cliente como fonte de verdade para privilégios.
- Aplicar **deny by default**, menor privilégio e testes automatizados de autorização.
- Proteger operações de alteração de estado contra CSRF e não usar **GET** para exclusões.

## English summary

**Finding:** Vertical privilege escalation through a client-controlled `Admin` cookie.  
**Method:** Modified `Admin=false` to `Admin=true` in Burp Repeater, then verified unauthorized access to `/admin`.  
**Impact:** Access to privileged operations, including deletion of a lab test user.  
**Remediation:** Enforce server-side authorization for every privileged operation and never trust client-controlled role indicators.

## Ética

Atividade executada exclusivamente no laboratório intencionalmente vulnerável da PortSwigger. Dados de sessão e tokens não são reproduzidos neste relatório.
