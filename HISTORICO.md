# Histórico técnico e resultados

## 1. Preparação do ambiente

A análise foi transferida para uma VM Linux dedicada no Proxmox, separada do ambiente operacional do painel. Verificaram-se permissões da conta de análise, ausência de compartilhamentos com arquivos pessoais, restrições de rede, separação dos outros serviços e funcionamento das ferramentas em amostras de teste.

A conta de análise foi mantida sem privilégios administrativos por `sudo`. A existência de uma VM não foi tratada como comprovação suficiente de isolamento; as restrições foram verificadas separadamente.

## 2. Identificação das unidades

Foram utilizadas duas câmeras, denominadas A e B. Os dados e resultados foram separados desde o início. Nome comercial semelhante não foi considerado prova de equivalência entre hardware, firmware ou configuração.

A identificação considerou os artefatos e informações retornadas pelo equipamento. Diferenças entre versões podem alterar arquivos, funções, formatos de configuração, autenticação, integridade e recuperação.

## 3. Preservação dos originais

Pacotes e dumps foram preservados antes das alterações. Registraram-se origem, tamanho e SHA-256; o trabalho ocorreu sobre cópias, com comparação de leituras independentes quando disponíveis.

Uma tentativa de leitura sem conteúdo foi registrada como falha, nunca como backup. Na unidade investigada por leitura externa, leituras completas independentes coincidiram.

A conexão do programador ao computador não foi confundida com leitura da memória: inicialmente houve identificação inválida, resolvida antes de prosseguir.

A montagem investigada exigiu isolar temporariamente o pino 8 de alimentação da memória SPI NOR e restaurá-lo antes do boot. As dificuldades e fotografias estão descritas em [Hardware](HARDWARE.md).

## 4. Inspeção do firmware com REA

A inspeção identificou Linux ARM de 32 bits, executáveis, sistema de arquivos e áreas de configuração. REA e ferramentas auxiliares apoiaram a localização de componentes, referências e funções.

| Questão | Evidência necessária |
|---|---|
| Um serviço aparece no firmware? | Referências, funções e componentes relacionados |
| O serviço está ativo? | Resposta observada em execução |
| O serviço aceita o usuário autorizado? | Autenticação real e operação concluída |

A presença de referências a RTSP ou ONVIF não foi tomada como acesso disponível. A análise ampla encontrou limitação de memória; o trabalho passou a investigar funções específicas, preservando resultados intermediários.

## 5. Configurações persistentes

Parte das configurações não estava em texto simples. Foram estudados contêineres, cabeçalhos, comprimentos, verificações de integridade e estruturas internas.

```text
Arquivo original → decodificação → representação intermediária
→ reconstrução sem alterações → comparação byte a byte
```

A reconstrução idêntica permitiu verificar preenchimento e metadados. Também foi corrigida a preservação de identificadores de proprietário e grupo em uma reconstrução do sistema de arquivos.

## 6. Autenticação e conta local

Os serviços de vídeo estavam presentes, mas inicialmente não havia autenticação local válida demonstrada. A investigação separou inicialização, carregamento de contas, permissões, autenticação dos serviços e persistência após reinicialização.

A ausência de uma conta em determinados arquivos extraídos não foi considerada prova de ausência em execução. Caminhos de autenticação distintos foram analisados separadamente, sem generalizar conclusões entre serviços.

O resultado foi uma conta local autenticada nos serviços utilizados. A conta não foi demonstrada como usuário de shell, `root` ou usuário com `sudo`.

## 7. Candidatos rejeitados e integridade

Uma abordagem inicial alterava uma região do sistema de arquivos. Embora a reconstrução parecesse estruturalmente correta, a assinatura falhou na região modificada. O candidato foi rejeitado para gravação.

O verificador foi revisado para exigir a estrutura completa esperada, incluindo padding e digest, evitando aceitação por comparação incompleta. Posteriormente, foi construída uma alteração restrita à configuração, preservando as regiões assinadas.

| Verificação | O que demonstra |
|---|---|
| Sistema de arquivos extraível | Estrutura reconhecível pelas ferramentas |
| CRC ou hash coerente | Consistência segundo aquela verificação |
| Assinatura aprovada | Aceitação pelo verificador reproduzido |
| Boot e autenticação reais | Funcionamento observado no equipamento |

Nenhuma dessas verificações substitui todas as outras. A chave pública utilizada para verificar assinaturas não permite produzir assinaturas do fabricante.

## 8. Validação no hardware

Após verificações e autorização para intervenção, houve alteração controlada na configuração de uma unidade. Os registros incluem comparação prévia com o backup, verificação posterior completa e preservação das regiões assinadas.

Depois da restauração física, confirmou-se a identidade, testou-se autenticação, consultaram-se serviços, abriu-se o vídeo e repetiram-se verificações após reinicialização controlada.

A representação da credencial foi normalizada pelo gerenciador nativo do firmware. A segunda câmera foi tratada separadamente, com seus próprios dados e verificações.

Resultado: acesso local autenticado e persistente após reinicialização nas duas unidades testadas. Isso não constitui procedimento universal para todas as revisões comerciais.

## 9. Atualização por cartão

Foram analisados cabeçalhos, limites de componentes, CRCs, assinaturas e comportamento de gravação. Os testes incluíram arquivos oficiais e alterações controladas em memória para verificar rejeição de conteúdo modificado.

Foi possível verificar o formato dos pacotes examinados, mas não foi demonstrado um instalador por cartão capaz de implantar a conta local. A presença de um componente com nome semelhante a script não foi tomada como prova de execução de comandos personalizados.

## 10. Gravação e áudio

Configuraram-se perfis diferentes: A por detecção de pessoas e B com gravação contínua. Testes reais confirmaram geração e reprodução de gravações.

O pós-evento observado foi compatível com aproximadamente 30 segundos. A precisão exata do pré-evento configurado em 2 segundos não foi certificada.

Na câmera contínua, uma amostra de dez minutos apresentou continuidade nos timestamps dos arquivos originais. Isso não comprova ausência de falhas durante todas as 24 horas.

A captura do microfone do navegador e o envio de áudio foram validados nas duas unidades. Essa validação não certifica qualidade acústica em qualquer ambiente.

## 11. Implantação e verificação

O painel foi migrado para um contêiner não privilegiado no Proxmox. Testaram-se autenticação, vídeo, gravações e funcionamento após reinicialização do contêiner.

Na entrega dessa etapa, registraram-se 54 testes aprovados no computador e 34 testes de núcleo, segurança e codificação no Linux. São resultados daquela etapa, não uma nova execução nesta compilação.

O backup completo do contêiner passou pela verificação de integridade, mas não por restauração de teste. A revisão de segurança identificou pontos de manutenção; não se concluiu que todos os problemas estavam resolvidos.

## Resultados e limites

| Área | Resultado registrado |
|---|---|
| Análise com REA e ferramentas auxiliares | Componentes, configurações e fluxos investigados |
| Preservação de arquivos | Originais mantidos e comparações documentadas |
| Conta local | Autenticação validada nos serviços utilizados |
| Persistência | Confirmada após reinicialização nas unidades testadas |
| Vídeo e áudio | Integração funcional no painel |
| Gravações | Busca e reprodução verificadas |
| Atualização por cartão | Formato estudado; instalação da conta não demonstrada |
| Pré-evento exato | Não certificado |
| Operação inteiramente offline | Não comprovada |
| Desativação da nuvem do fabricante | Não comprovada |
| Acesso remoto final | Validação completa pendente nos registros |
| Recuperação por backup | Integridade verificada; restauração não testada |
