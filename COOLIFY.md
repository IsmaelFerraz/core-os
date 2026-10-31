# Core-OS no Coolify (branch coolify-secure)

Esta branch mantém os arquivos originais e adiciona uma implantação Docker Compose.
Não execute `skill/instalar.sh` numa VPS onde Coolify já está instalado.

## 1. Implantação

1. No Coolify, crie projeto e recurso **Git Repository / Docker Compose**.
2. Repositório: `IsmaelFerraz/core-os`; branch: `coolify-secure`.
3. Arquivo Compose: `/compose.coolify.yaml`.
4. Cadastre as variáveis abaixo no Coolify (segredos privados).
5. Configure os domínios HTTPS nas portas internas: painel `3000`, n8n `5678`, Evolution `8080` apenas se precisar de acesso externo.
6. Faça deploy, verifique logs e teste o login. Restrinja o painel por uma camada adicional de acesso (por exemplo, Cloudflare Access) quando possível.

### Variáveis obrigatórias
- `CORE_OS_PASSWORD`: senha longa para o painel.
- `CORE_OS_HOOK_TOKEN`: token aleatório longo para ponte interna.
- `POSTGRES_PASSWORD`: senha forte sem caracteres que exijam escape em URL PostgreSQL; idealmente use uma senha hexadecimal.
- `EVO_KEY`: segredo da Evolution API.
- `EVOLUTION_IMAGE`: referência de imagem + tag verificada e compatível da Evolution API.
- `EVOLUTION_PUBLIC_URL`: URL HTTPS configurada para Evolution, se aplicável.
- `N8N_ENCRYPTION_KEY`: chave longa para criptografia do n8n.
- `N8N_HOST`: host do domínio n8n, sem protocolo.
- `N8N_PUBLIC_URL`: URL HTTPS completa do n8n terminada em /.
- `N8N_IMAGE`: opcional; **fixe uma versão testada**, em vez de depender de `latest`.

## 2. Contas Claude e ChatGPT
Dentro do terminal do container `coreos`, rodando como usuário `node`:
```sh
claude
codex login --device-auth
codex login status
```
Conclua as autorizações pelo navegador. Pastas `/home/node/.claude` e `/home/node/.codex` persistem via volumes.
Não configure `ANTHROPIC_API_KEY` ou `OPENAI_API_KEY` se deseja autenticar via assinatura.
Respeite limites e políticas dos provedores para automação.

### Antigravity
O código original referencia um executável chamado `agy`. Ele **não** é instalado neste Dockerfile.
A compatibilidade, origem e autenticação deste CLI ainda precisam ser verificadas antes de habilitá-lo.

## 3. Fluxo WhatsApp
O arquivo original está em `skill/stack/fluxos/whatsapp-claude.json`. Importar e revisar manualmente no n8n.
A ponte interna é `http://coreos:3000/api/ponte/whatsapp`, autenticada por `x-core-os-token`.
É necessário adaptar as URLs do fluxo, instância Evolution, webhook e validação do remetente antes de ativá-lo.
A configuração original usa `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` devido ao fluxo legado: verifique essa dependência e remova o acesso em uma revisão posterior.

## 4. Limitações e segurança
- **Não homologado para produção**. O código do projeto ainda permite ferramentas administrativas amplas.
- WhatsApp com poderes de administrar o Core-OS não deve significar acesso irrestrito ao host Docker ou à VPS.
- Não monte `/var/run/docker.sock`, `/root` ou `/data/coolify` dentro do container.
- Não exponha PostgreSQL ou Redis na internet.
- Faça backups dos volumes, especialmente `coreos-brain`, `coreos-data`, `coreos-n8n` e `coreos-postgres`.
- O container usa `no-new-privileges` e usuário não-root, mas ainda requer uma auditoria completa.
- O botão de deploy não cria nem ativa automaticamente o fluxo do WhatsApp.
- A primeira inicialização dos volumes copiará o cérebro de demonstração; edite seus conteúdos no painel depois.
- CLI Google/Antigravity e OpenRouter ficam para fases posteriores.

## 5. Revisão antes de expor publicamente
Confirmar TLS, autenticação adicional, restrição de números WhatsApp, limites de recursos e rede, backup e logs. Trocar todas as credenciais padrão ou reutilizadas. 
