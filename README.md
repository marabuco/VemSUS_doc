# VemSUS — backend Python + dados reais da Prefeitura do Recife

```
backend/   FastAPI: consome o CKAN, normaliza, georreferencia, busca e serve /api (+ o site buildado)
frontend/  arquivos ALTERADOS do seu projeto Vite (src/services/api.ts, src/types.ts, src/data/mock.ts)
deploy/    docker-compose + Caddy (HTTPS automático)
docs/      FALHAS_CORRIGIDAS.md · DOMINIO_E_PUBLICACAO.md
```

## Fluxo de dados
`dados.recife.pe.gov.br` (CKAN `datastore_search`, resource `c960077b-…`) → paginação → agrupar linhas por CNES → validar/reparar lat‑lng → (se faltar) geocodificar via Nominatim → SQLite → `/api/units` → site.
Equivalente ao seu curl, feito pelo servidor: `POST /api/3/action/datastore_search` com `Authorization: <CKAN_API_TOKEN>` e `{"resource_id": "...", "limit": 1000, "offset": N}`.

## Rodar local
```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env            # preencha CKAN_API_TOKEN e ADMIN_TOKEN
uvicorn app.main:app --reload --port 8000
curl localhost:8000/api/meta    # ready:true depois da 1ª sincronização
pytest                          # 12 testes
```
Frontend: copie os 3 arquivos de `frontend/src/` por cima dos seus, aplique `frontend/vite.config.proxy-patch.md`, remova `VITE_RECIFE_API_TOKEN` e rode `npm run dev`.

## API
| Rota | Descrição |
|---|---|
| `GET /api/units?q=&services=&type=&child=&lat=&lng=&limit=` | Busca com ranking; com `lat/lng` devolve `distanceKm` |
| `GET /api/units/{id}` | Detalhe da unidade |
| `GET /api/services` | Tags de serviço e tipos de unidade |
| `GET /api/meta` | Fonte, licença ODbL, última sincronização, nº de unidades |
| `GET /api/health` | Saúde do serviço |
| `POST /api/admin/sync` · `GET /api/admin/diagnostico` | Header `X-Admin-Token`; forçam sincronização / mostram o que não foi reconhecido |
