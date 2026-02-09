# integrarAPI_bps_tiny_driver

Projeto para automação da integração entre APIs (BPS/Tiny) e sincronização de arquivos com o Google Drive.

## Visão geral

Este projeto automatiza tarefas comuns em um fluxo de logística/comércio eletrônico: baixar etiquetas/labels, sincronizar arquivos com o Google Drive, procurar arquivos locais e colar etiquetas nos pedidos, e enviar códigos de rastreio para o Tiny (sistema de gestão). O programa principal é `main.py`, que orquestra várias rotinas na pasta `utils/`.

Principais objetivos:
- Sincronizar uma pasta local com o Google Drive (listar e fazer upload de arquivos novos).
- Baixar etiquetas de pedidos (labels) usando APIs externas.
- Localizar arquivos de etiqueta na pasta local e associá-los a pedidos.
- Enviar códigos de rastreamento para o Tiny (atualizar pedidos com rastreio).

## O que ele faz (resumo das funcionalidades)

- `Colocar_Rastreio_no_tiny`: consulta pedidos e envia códigos de rastreio para o Tiny.
- `baixar_labels`: baixa arquivos de etiqueta a partir de uma API (BPS/Tiny) para uma pasta local.
- `api_google_driver/pegar_lista_driver.py`: encapsula operações de listagem e upload para Google Drive.
- `Procurar_e_colocar_etiquetas`: procura arquivos de etiqueta no diretório local e acopla ao pedido correspondente.
- `main.py`: executa as rotinas em sequência, faz upload de arquivos novos ao Drive e registra erros em `log.txt`.

## Estrutura do projeto

- `main.py` - entrypoint do script. Chama as funções principais e grava erros em `log.txt`.
- `requirements.txt` - dependências do projeto.
- `log.txt` - arquivo de log onde exceções e traces são salvos (gerado em tempo de execução).
- `utils/` - pasta com código utilitário e módulos:
  - `baixar_labels.py` - rotina de download de labels.
  - `Colocar_Rastreio_no_tiny.py` - rotina de envio de rastreios para Tiny.
  - `Procurar_e_colocar_etiquetas.py` - rotina que localiza arquivos e vincula etiquetas a pedidos.
  - `func_BPS.py` - helpers para comunicação com BPS (integração BPS).
  - `func_Tiny.py` - helpers para comunicação com Tiny (API Tiny).
  - `api_google_driver/` - helpers para Google Drive (autenticação, listagem, upload):
    - `pegar_lista_driver.py` - funções para listar e subir arquivos ao Drive.
    - `configs.json` (copiado como `configs copy.json` no repositório) - arquivo de configuração local com o caminho da pasta monitorada e possíveis credenciais/IDs.
  - arquivos de credenciais de exemplo: `credenciais copy.json`, `credentials_change.json` (atenção: estes são exemplos/cópias; substitua pelos seus arquivos de credencial corretos).

## Requisitos / Dependências

As dependências estão listadas em `requirements.txt`. Para criar um ambiente e instalar pacotes:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Dependências principais (exemplo):
- requests
- google-api-python-client
- google-auth
- google-auth-oauthlib

Verifique `requirements.txt` para versões específicas.

## Configuração necessária

1. Google Drive
   - Coloque as credenciais do OAuth/Service Account na pasta `utils/api_google_driver/` conforme exigido pelo código (arquivos atualmente nomeados como `credentials_change.json` e `configs copy.json`).
   - Ajuste `configs.json` (ou o arquivo equivalente que o código realmente usa) com `local_folder_path` apontando para a pasta onde as etiquetas serão salvas/monitoradas.

2. APIs externas (BPS / Tiny)
   - Verifique `utils/func_BPS.py` e `utils/func_Tiny.py` para descobrir como o projeto espera obter tokens/credenciais.
   - Preencha quaisquer arquivos `credenciais` com suas chaves ou configure variáveis de ambiente se o código suportar.

3. Permissões e caminhos
   - Em Windows, use caminhos absolutos nos arquivos de configuração quando necessário.
   - Certifique-se de que o usuário que executa o script tem permissão de leitura/gravação na pasta monitorada.

Observação: Não comitar credenciais sensíveis em repositórios públicos. Use arquivos de configuração locais, variáveis de ambiente ou segredos do CI.

## Como executar

Execute a partir da raiz do projeto com o ambiente virtual ativado:

```powershell
python .\main.py
```

O `main.py` chama, na ordem:
1. `Colocar_Rastreio_no_tiny` (tenta atualizar pedidos com rastreios)
2. `baixar_labels` (baixa novas etiquetas)
3. Sincroniza/faz upload de arquivos locais para o Google Drive
4. `Procurar_e_colocar_etiquetas` (associa etiquetas baixadas a pedidos locais)

Logs e erros serão registrados em `log.txt` na raiz do projeto.

## Erros comuns e dicas de troubleshooting

- "name 'autenticar_bps' is not defined" — significa que alguma função de autenticação não está importada/definida; verifique `func_BPS.py` e as chamadas nos módulos que dependem dele.
- Erros relacionados a `configs.json` ou `local_folder_path` — verifique se o arquivo de configuração foi copiado/renomeado corretamente e se o campo `local_folder_path` aponta para a pasta certa.
- Erros do Google Drive como `KeyError: 'names'` — pode indicar resposta inesperada da API; verifique a versão das bibliotecas do Google, as credenciais e se o método usado para listar arquivos ainda retorna o mesmo formato.
- `NoneType` is not iterable / UnboundLocalError sobre variáveis `pedido(s)` — indica que a rotina não recebeu dados esperados (por exemplo, a API retornou vazio). Verifique respostas das APIs externas (BPS/Tiny) e trate casos de lista vazia.
- Para depuração: inspecione `log.txt` (stack traces são gravados) e reproduza a chamada problemática diretamente no módulo afetado com dados de teste.

## Sugestões de melhorias / próximos passos

- Adicionar validação e mensagens de erro mais claras nas funções que chamam APIs externas.
- Criar testes unitários (ex.: pytest) para os módulos `func_Tiny.py` e `func_BPS.py` simulando respostas de API.
- Adicionar um pequeno script de configuração (setup) que cria `configs.json` a partir de um template e valida caminhos.

