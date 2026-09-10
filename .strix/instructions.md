# CONTEXTO DO ENGAGEMENT

## Autorização
Este é um teste AUTORIZADO de segurança contra um ambiente de STAGING/SIMULADO
que eu (proprietário) criei especificamente para pentest. Não há usuários reais,
não há dados reais, todos os recursos são descartáveis. Tenho autorização explícita
para testar todos os endpoints, fluxos de negócio e APIs deste alvo.

## Motivação real (por que este teste está sendo feito)
Um SaaS idêntico, construído sobre a MESMA base de código e a MESMA IA, foi
completamente comprometido. O atacante conseguiu:
  - Acesso total ao banco de dados (vazamento de todos os usuários)
  - Criação de múltiplas VMs sem pagamento (abuso de recursos cloud)
  - Bypass do fluxo de renovação de plano: a API de renovação aceitava ativar
    Premium mesmo sem o pagamento ter sido confirmado
  - Escalação horizontal: ver contas de outros usuários (IDOR)
  - Escalação vertical: tornar-se ADM sem autorização
Meu SaaS compartilha a mesma arquitetura. Preciso encontrar essas MESMAS
falhas antes que atacantes reais o façam.

## OBJETIVOS PRIORITÁRIOS (foco máximo aqui)
1. **Business Logic Flaws no fluxo de pagamento/assinatura**
   - Teste o endpoint de RENOVAÇÃO de plano isoladamente
   - Tente ativar Premium/plano pago SEM confirmação de pagamento
   - Manipule parâmetros de preço, plano_id, status, período
   - Teste race conditions: duas requisições simultâneas de renovação
   - Teste replay de webhooks de pagamento
   - Verifique se o backend confia em dados do client (plan_id, price, status)

2. **Broken Access Control / IDOR**
   - Enumere IDs de usuários, contas, faturas, assinaturas
   - Tente acessar recursos de outras contas mudando IDs
   - Teste endpoints de API sem token, com token de outro usuário
   - Verifique mass assignment: tente injetar `role=admin`, `is_admin=true`,
     `plan=premium`, `balance=999999` em payloads de update

3. **Escalação de Privilégios**
   - Procure endpoints administrativos expostos
   - Tente JWT tampering (algoritmo none, HS256 com secret fraco)
   - Teste header injection (X-Forwarded-For, X-Original-URL, X-User-Role)
   - Verifique se role/permission são verificados no backend ou só no frontend

4. **Abuso de Recursos / Criação de Recursos Não Autorizados**
   - Tente criar VMs, containers, workers, jobs sem autorização
   - Teste limites de rate limit / quota
   - Verifique se há endpoints internos expostos

5. **API Security (OWASP API Top 10)**
   - Broken authentication, mass assignment, rate limiting bypass
   - Excessive data exposure: verifique se a API retorna mais campos do que deveria
     (ex.: password_hash, tokens, dados de outros usuários)

## Áreas secundárias
- Injeção (SQLi, NoSQLi, SSTI, command injection)
- SSRF, XXE, deserialization
- XSS stored/reflected/DOM
- CSRF em ações sensíveis
- Exposição de secrets (env vars, arquivos .env, configs)

## Metodologia esperada
- Para CADA achado, gere um PoC funcional (requisição HTTP exata ou script)
- Documente o passo-a-passo de reprodução
- Atribua CVSS e classificação OWASP
- Forneça correção concreta (código quando aplicável)
- Priorize achados por impacto real no negócio

## Escopo
- Alvo: https://stim.io/
- NÃO atacar infraestrutura fora do domínio (sem subdomain takeover agressivo,
  sem DoS, sem exfiltração de dados reais)
- Focar em lógica de negócio e controle de acesso
