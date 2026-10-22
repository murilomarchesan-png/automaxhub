# PontoCerto — instruções para o Claude Code

**Antes de qualquer coisa, leia `SPEC.md` inteiro.** Este arquivo é só o resumo e as regras que valem em toda sessão.

## O que é
Plataforma de análise de ponto comercial: endereço + tipo de negócio → relatório com nota 0–10, concorrência, público, fluxo, aluguel, faturamento estimado, tendência do setor, percepção da região (IA), PDF, histórico e comparação. Dados: IBGE (setores, Censo 2022, CNEFE, POF), Receita Federal (CNPJ), Google Places, OpenStreetMap.

## Stack
- VPS Hostinger com Docker Swarm + Traefik + Portainer (já existentes).
- n8n (já existente) orquestra a análise. Workflows novos sempre com prefixo `PC — `.
- Serviços novos no stack `pontocerto`: `pc-db` (PostGIS), `pc-web` (Next.js), `pc-gotenberg`, `pc-etl` (Python).
- Lógica espacial em funções SQL (`db/functions/`); nota em `engine/score.js` (com testes), sincronizado no Code node do n8n.

## Regras invioláveis
1. Não alterar serviços existentes (n8n, Evolution API, Postgres atual, Redis, Traefik) nem workflows existentes.
2. Nunca inventar dado: todo número tem fonte e data. Dado ausente → o relatório diz que não está disponível.
3. Validar na fonte oficial tudo que a SPEC marca com ⚠️ VALIDAR.
4. Respeitar os termos do Google (não montar base própria com dados do Places; só guardar `place_id`).
5. LGPD: base CNPJ sem e-mail, telefone e sócios; não exibir razão social de MEI.
6. Nenhum segredo no git.
7. Não copiar textos, marca, layout ou exemplos do site de referência.
8. Decisões em aberto (SPEC §15): perguntar ao Murilo antes da fase em que são necessárias.

## Fase atual
**MVP só no n8n — v0.1** (SPEC §14.0). Sem site por enquanto: entrada pelo Form Trigger do n8n, relatório em HTML. Atualize esta linha ao concluir cada degrau/fase.

## Infra conhecida
VPS Hostinger KVM 4 (`srv1308006.hstgr.cloud`): 15 GiB RAM (~13 GiB livres em out/2026), 193 GB de disco (182 GB livres), sem swap. `pc-db` sempre com limite de memória/CPU no `stack.yml`; cargas pesadas de madrugada.

## Ao terminar cada fase
Entregar um resumo curto: o que foi feito, como testar, pendências, decisões tomadas e divergências encontradas com a SPEC.
