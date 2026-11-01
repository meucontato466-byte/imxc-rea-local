# Exemplos fictícios de investigação com REA

Os nomes, valores, funções e trechos abaixo foram inventados para ensino. Não são código decompilado das câmeras, comandos de ferramentas REA ou especificações do protocolo do produto. Os exemplos executáveis usam apenas a biblioteca padrão do Python e dados sintéticos em memória.

## 1. De uma mensagem de erro até a função responsável

**Problema:** um serviço responde, mas rejeita uma operação. Como encontrar o código responsável em um executável sem nomes de funções?

Considere este pseudocódigo fictício:

```c
int func_A(Request *request) {
    int result = func_B(request);
    if (result == ERR_PERMISSION)
        log_message("DEMO: permission rejected");
    return result;
}

int func_B(Request *request) {
    Session *session = find_session(request->session_id);
    if (session == NULL)
        return ERR_SESSION;
    return has_permission(session, request->operation)
        ? OK : ERR_PERMISSION;
}
```

### Investigação

1. Localizar a string `DEMO: permission rejected` no artefato de teste.
2. Consultar suas referências cruzadas: quais instruções usam esse texto?
3. Abrir a função que contém a referência e examinar o fluxo até o erro.
4. Identificar a chamada cujo retorno determina o ramo: neste exemplo, `func_B`.
5. Examinar `func_B`, seus argumentos e seus chamadores.
6. Separar falha de sessão de falha de permissão.

No fluxo com REA, a solicitação pode ser formulada assim, sem presumir nomes de comandos:

> No artefato de teste selecionado, localize a string exata “DEMO: permission rejected”. Mostre suas referências, a função que contém cada referência e a condição que leva à mensagem. Separe o que está visível no código do que ainda é hipótese.

Depois:

> Analise a função cujo retorno controla esse ramo. Identifique entradas, saídas e chamadas relevantes. Não renomeie uma função como verificador de senha apenas por proximidade com a mensagem.

**Descoberta:** a função próxima do erro verifica sessão e autorização, não a senha. Insistir em mudar a representação da credencial seria investigar a etapa errada.

**Limite:** ausência da string não demonstra ausência da função; mensagens podem ter sido removidas, construídas dinamicamente ou armazenadas em outro componente. O pseudocódigo do decompilador também precisa de conferência de tipos e fluxo.

**Relação com o projeto:** a separação entre inicialização, conta, autenticação e permissões evitou atribuir todo erro ao mesmo mecanismo.

## 2. Dois serviços não significam o mesmo caminho de autenticação

**Problema:** uma opção de configuração parece controlar um serviço. Ela também controla outro que compartilha parte do código?

Imagine este grafo fictício:

```text
entrada_do_player → validar_sessao → conferir_permissao

entrada_do_catalogo → conferir_catalogo_habilitado
                   → validar_sessao → conferir_permissao
```

### Investigação

1. Localizar cada ponto de entrada separadamente.
2. Acompanhar suas chamadas até o verificador compartilhado.
3. Anotar condições que aparecem antes da convergência.
4. Consultar quem lê a opção de configuração.
5. Procurar caminhos alternativos antes de concluir dependência universal.

| Observação fictícia | Conclusão permitida |
|---|---|
| As duas entradas chamam `validar_sessao` | Compartilham essa etapa |
| Apenas o catálogo consulta sua opção de habilitação no caminho examinado | A condição foi observada nesse caminho |
| O player responde com o catálogo desabilitado em teste controlado | Nessa versão e nesse teste, o player funciona sem o catálogo |

Uma pergunta útil ao REA seria:

> Compare os caminhos dos dois pontos de entrada até o verificador compartilhado. Liste as condições exclusivas de cada caminho e os caminhos indiretos ainda não examinados.

**Descoberta:** compartilhar uma função de autenticação não significa compartilhar todas as condições de inicialização.

**Relação com o projeto:** foi necessário distinguir caminhos nativos de vídeo e caminhos de integração. Uma condição encontrada em um deles não podia ser generalizada para todos.

## 3. O “fator X”: mesmos dados, representações diferentes

**Problema:** o valor parece correto ao olhar, mas uma comparação continua falhando. O erro está no dado ou em como ele foi serializado?

Neste exemplo, um verificador didático exige um identificador com quatro posições. Um componente converte o identificador para inteiro e perde os zeros iniciais. O exemplo usa SHA-256 apenas como marcador de igualdade de bytes; não implementa autenticação nem armazenamento de senhas.

```python
from hashlib import sha256

def marcador(texto):
    return sha256(texto.encode("ascii")).digest()

valor_do_arquivo = "0042"
valor_apos_conversao = str(int(valor_do_arquivo))

assert valor_do_arquivo != valor_apos_conversao
assert marcador(valor_do_arquivo) != marcador(valor_apos_conversao)

# Regra inventada deste formato: inteiro decimal com quatro posições.
valor_normalizado = f"{int(valor_apos_conversao):04d}"
assert marcador(valor_do_arquivo) == marcador(valor_normalizado)

print("Valores numericamente iguais podem produzir bytes diferentes.")
```

### Investigação

1. Rastrear a origem do campo e o ponto onde ele é convertido.
2. Conferir se o consumidor espera número, texto ou bytes.
3. Examinar a serialização imediatamente antes da comparação.
4. Construir um caso mínimo com dados sintéticos.
5. Testar um caso positivo e um negativo.
6. Confirmar a regra no consumidor; não inserir uma normalização por tentativa.

Pergunta possível ao REA:

> Acompanhe este campo desde a leitura até a comparação. Identifique conversões de tipo, codificação, preenchimento e comprimento. Mostre onde a representação muda e quais evidências sustentam o formato esperado.

**Descoberta:** o conteúdo numérico estava correto, mas os bytes eram diferentes. Recalcular o marcador sobre a representação errada repetiria a falha.

**Relação com o projeto:** houve dificuldades envolvendo representação e compatibilidade entre componentes de autenticação. O exemplo ensina a investigação dessas diferenças sem apresentar a transformação específica dos equipamentos.

## 4. Reconstrução plausível não é reconstrução idêntica

**Problema:** um arquivo decodifica corretamente e todos os campos parecem iguais, mas o arquivo reconstruído não coincide com o original.

O contêiner inventado abaixo tem comprimento, conteúdo e uma área reservada. Um reconstrutor ingênuo descarta a área reservada:

```python
import struct

conteudo = b"DEMO"
reservado = bytes([17, 34, 51])
original = struct.pack("<H", len(conteudo)) + conteudo + reservado

tamanho = struct.unpack_from("<H", original, 0)[0]
fim = 2 + tamanho
extraido = original[2:fim]

reconstrucao_incompleta = struct.pack("<H", tamanho) + extraido
assert extraido == conteudo
assert reconstrucao_incompleta != original

reconstrucao_exata = (
    struct.pack("<H", tamanho) + extraido + original[fim:]
)
assert reconstrucao_exata == original

print("Round-trip exato: conteúdo e bytes reservados preservados.")
```

### Investigação

1. Reconstruir sem editar nenhum campo.
2. Comparar tamanho e conteúdo byte a byte.
3. Localizar a primeira diferença e verificar se ela se propaga.
4. Examinar preenchimento, campos reservados e metadados.
5. Corrigir a reconstrução antes de interpretar qualquer alteração funcional.

**Descoberta:** o decodificador recuperava o conteúdo, mas isso não comprovava que o reconstrutor preservava o formato inteiro.

**Relação com o projeto:** a validação de ida e volta e a preservação de metadados foram necessárias antes de confiar em reconstruções. Mesmo uma igualdade estrutural não substituiria a validação das assinaturas aplicáveis.

## Como registrar uma descoberta

Para cada hipótese, anote:

| Campo | Exemplo fictício |
|---|---|
| Pergunta | Por que a comparação falha? |
| Evidência | O produtor transforma texto em inteiro |
| Hipótese | O preenchimento é perdido nessa conversão |
| Teste positivo | Representação de quatro posições coincide |
| Teste negativo | Representação sem preenchimento difere |
| Conclusão | O formato exige preservação da representação |
| Limite | Não demonstra como outro produto trata o campo |

Essa organização ajuda a transformar uma pista localizada com REA em uma conclusão verificável, mantendo separado o que foi observado do que ainda precisa de teste.
