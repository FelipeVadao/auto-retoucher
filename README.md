# Auto Retoucher

Ferramenta que analisa fotos de retrato com IA e gera um laudo técnico de retoque de pele, no estilo de um retocador profissional de Photoshop: para cada problema identificado (olheiras, manchas, poros, rugas etc.), indica a prioridade, a técnica de Photoshop recomendada e a localização exata (x, y) na imagem.

## Como funciona

1. O backend ([api.py](api.py)) recebe o upload de uma imagem via API FastAPI.
2. Um agente de IA ([agent_api.py](agent_api.py)), construído com [Agno](https://github.com/agno-agi/agno) e o modelo Claude da Anthropic, analisa a imagem seguindo o prompt de especialista em [prompts/skin.md](prompts/skin.md).
3. Para cada item do laudo, o agente chama a ferramenta [FalPointTool](tools/fal_point_tool.py), que usa o modelo `moondream3-preview/point` via [fal.ai](https://fal.ai) para localizar o ponto exato do problema na imagem.
4. O resultado estruturado (validado com Pydantic em [tools/models.py](tools/models.py)) é devolvido como JSON e também salvo em `history/` para consulta posterior.
5. Um frontend simples ([templates/index.html](templates/index.html)) permite enviar imagens e visualizar o laudo e os pontos marcados.

## Stack

- Python 3.12+
- [FastAPI](https://fastapi.tiangolo.com/) + Uvicorn
- [Agno](https://github.com/agno-agi/agno) (orquestração do agente)
- Anthropic Claude (análise da imagem)
- fal.ai (`moondream3-preview/point`, localização de pontos)
- Pillow, Pydantic
- Gerenciamento de dependências com [uv](https://docs.astral.sh/uv/)

## Pré-requisitos

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) instalado
- Chaves de API:
  - `ANTHROPIC_API_KEY`
  - `FAL_API_KEY`
  - `GOOGLE_API_KEY` (usado pelas dependências do Agno/Google GenAI)

## Configuração

1. Clone o repositório e instale as dependências:

   ```bash
   uv sync
   ```

2. Crie um arquivo `.env` na raiz do projeto com suas chaves:

   ```env
   ANTHROPIC_API_KEY=sua_chave_aqui
   FAL_API_KEY=sua_chave_aqui
   GOOGLE_API_KEY=sua_chave_aqui
   ```

## Uso

Suba a API:

```bash
uv run uvicorn api:app --reload
```

Acesse `http://localhost:8000` no navegador para usar a interface web, ou use a API diretamente:

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/` | Interface web para upload e visualização do laudo |
| `POST` | `/analyze` | Recebe um arquivo de imagem (`multipart/form-data`) e retorna o laudo estruturado |
| `GET` | `/history` | Lista todas as análises já realizadas |
| `DELETE` | `/history/{filename}` | Remove uma análise e sua imagem do histórico |

Também é possível rodar a análise diretamente via script:

```bash
uv run agent_api.py
```

## Estrutura do projeto

```
agent_api.py        # Agente de IA que analisa a imagem e monta o laudo
api.py               # API FastAPI (upload, histórico, servidor da interface web)
tools/
  fal_point_tool.py  # Ferramenta do agente para localizar pontos via fal.ai
  models.py          # Schemas Pydantic do laudo (ReportItem, SkinAnalysisSchema)
prompts/
  skin.md            # Prompt de especialista em retoque de pele
templates/
  index.html         # Interface web
frontend/            # Protótipos de design system / telas
history/             # Análises salvas (gerado em runtime, ignorado pelo git)
uploads/             # Imagens enviadas (gerado em runtime, ignorado pelo git)
```

## Licença

Projeto pessoal, sem licença definida.
