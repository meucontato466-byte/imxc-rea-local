# Arquitetura do painel

## Componentes

O painel executa fora do firmware da câmera. Utiliza servidor Python com aiohttp, retransmissão de vídeo e interface no navegador.

```text
Navegador → servidor autenticado → serviços dos equipamentos
                    ↓
             retransmissão de mídia
```

O backend consulta equipamentos cadastrados e confere sua identidade antes do uso das credenciais. O laboratório de análise permanece separado do serviço operacional.

## Funcionalidades

- Visualização simultânea das câmeras.
- Descoberta e verificação de identidade.
- Nomes e configurações individuais.
- Busca e reprodução de gravações do cartão.
- Configuração de rede.
- Controle de orientação e infravermelho.
- Escuta do áudio.
- Fala pelo microfone do navegador.
- Perfis independentes de gravação.

A reprodução no navegador utiliza Media Source Extensions. O caminho de fala recebe áudio do navegador e realiza a conversão necessária no servidor.

O navegador não recebe uma interface genérica para executar chamadas administrativas arbitrárias. As credenciais são utilizadas pelo backend.

## Implantação

O painel foi migrado do computador para um contêiner não privilegiado no Proxmox, permitindo operação independente do PC.

A implantação incluiu HTTPS, autenticação, cookies seguros, proteção CSRF, validação de origem, restrições do serviço, armazenamento protegido de credenciais e administração restrita do retransmissor.

Os testes incluíram funcionamento após reinicialização do contêiner e preservação das configurações individuais.

## Limites

HTTPS protege a conexão navegador-servidor; não implica cifragem dos protocolos nativos entre servidor e câmera.

O contêiner compartilha o kernel do host e não representa isolamento absoluto. Os testes e controles são evidências pontuais, não certificação de segurança absoluta. A integridade do backup não substitui restauração de teste.
