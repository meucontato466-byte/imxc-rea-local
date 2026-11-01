# Metodologia para equipamentos próprios

Veja também os [exemplos fictícios](EXEMPLOS.md), com perguntas de investigação para o REA e casos mínimos em Python.

## 1. Identificar equipamento e revisão

Estabeleça uma identificação para cada unidade e mantenha seus artefatos separados. Não reutilize a configuração completa de outra câmera como substituta da original. Um mesmo nome comercial pode abranger versões diferentes.

## 2. Preparar o laboratório

Verifique permissões, compartilhamentos e comunicação de rede antes de importar dumps. Não disponibilize ao laboratório chaves pessoais, credenciais de nuvem, arquivos de navegador ou diretórios domésticos compartilhados.

Use uma conta de análise sem privilégios administrativos para o trabalho cotidiano. Confirme as ferramentas em amostras de teste.

## 3. Preservar os originais

Registre origem, tamanho e hashes. Trabalhe sobre cópias e compare leituras independentes quando possível.

Um backup precisa ter tamanho coerente e conteúdo verificável. Um hash confirma igualdade entre arquivos; sozinho, não demonstra que o conteúdo representa corretamente toda a memória.

## 4. Inspecionar antes de executar

Identifique arquitetura, sistemas de arquivos e componentes. Comece pela estrutura do pacote e por perguntas delimitadas. Decompilar um executável inteiro nem sempre é o caminho mais eficiente.

Separe referências estáticas, comportamento em execução e autenticação funcional. Registre a evidência necessária para cada conclusão.

## 5. Usar o REA para perguntas específicas

Organize a investigação por componente ou função. Relacione descobertas do REA com os artefatos originais e confirme hipóteses com ferramentas independentes.

Preserve resultados intermediários. Não transporte conclusões entre versões sem conferir os artefatos correspondentes.

## 6. Validar formatos antes de editar

Antes de alterar um formato desconhecido, demonstre que consegue reconstruí-lo sem mudanças. Exija comparação byte a byte quando o objetivo for uma reconstrução idêntica.

Confira comprimentos, preenchimento, integridade, permissões e metadados. Um arquivo legível após extração ainda pode ser reconstruído incorretamente.

## 7. Interpretar integridade e assinaturas

Não confunda sistema de arquivos válido, CRC coerente e assinatura aprovada. Cada verificação responde a uma pergunta diferente.

Identifique candidatos rejeitados para não confundi-los com versões posteriores. A verificação estática não substitui boot e autenticação reais.

## 8. Planejar recuperação e validar por unidade

Planeje recuperação antes de qualquer escrita. Mantenha os dados exclusivos da unidade e confirme sua identidade antes de transmitir credenciais.

Teste uma unidade por vez e repita verificações após reinicialização. Observe regressões em vídeo, áudio, rede e gravações, além do recurso investigado.

## 9. Separar aplicação e laboratório

O servidor operacional não precisa receber dumps ou ferramentas de alteração. Valide identidade, autenticação, separação de configurações e comportamento de sessão no aplicativo.

Teste atualização, reinicialização e recuperação em etapas distintas. Um backup íntegro ainda precisa de restauração de teste para demonstrar recuperação.

## 10. Documentar limites

Distinga hipótese, teste estático e resultado real. Informe tamanho da amostra e versões investigadas. Não extrapole uma reprodução de minutos para continuidade de um dia inteiro.

Descreva separadamente configuração solicitada, leitura de retorno e comportamento observado: o pré-evento configurado não certifica sozinho o instante efetivo do primeiro quadro.
