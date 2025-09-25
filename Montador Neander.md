// Modo de Uso:
// 1. Compile e Execute o código (Dê preferência na utilização do Dev C++)
// 2. Ao inicializar o código, será criado automaticamente um arquivo chamado "Entrada.asm", antes de realizar qualquer
//    ação no terminal do código, entre no arquivo, por meio de um bloco de notas e edite conforme o desejado, 
//    ao finalizar a edição, feche o arquivo e no terminal clique no <ENTER> para dar continuidade
// 3. Ao dar sequência no terminal, um arquivo será automaticamente criado, denominado "Saida.mem", onde o conteúdo convertido
//    pelo código do arquivo "Entrada.asm", será transferido para o "Saida.mem". IMPORTANTE: O output do arquivo é em Decimal
// 4. Assim que o processo finalizar, saia do terminal do código, abra o Neander e carrege o arquivo "Saida.mem", dessa forma
//    o código desejado será apresentado

#include <stdio.h>   // Para operações de arquivo (fopen, fclose, fprintf, etc.).
#include <stdlib.h>  // Para exit, etc.
#include <string.h>  // Para strcmp, strcspn e etc.
#include <ctype.h>   // Para toupper (é uma função que converte um caractere minúsculo em maiúsculo).

// Valores Globais (Assim todos os diretórios de código poderão utilizar essas informações - mantida sempre na RAM até o fechamento do código).
#define INPUT_FILENAME "Entrada.asm" // Nome do arquivo de entrada padrão definido para todas as ocasiões dentro do código.
#define OUTPUT_FILENAME "Saida.mem"  // Nome do arquivo de saída padrão definido para todas as ocasiões dentro do código.

// Struct para mapear instruções Assembly para Opcodes(memória).
typedef struct {
    const char *assembly; // Instruções em assembly (ex: "LDA").
    unsigned char opcode; // Opcode correspondente (ex: 0x20).
    int req_operando;      // Indica se a instrução requer um operando (1 para sim, 0 para não).
} Instrucao;

// Esta tabela adapta o conjunto de instruções para seu formato de OpCode.
//Quando 0 ele não está acessando nenhum endereço e, quando 1 acessa um endereço.
Instrucao instrucoes[] = {
    {"NOP", 0x00, 0}, // Sem Operação (NOP - 0x00 0x00).
    {"STA", 0x10, 1}, // Salva o Acumulador (STA - 0x10 <endereço>).
    {"LDA", 0x20, 1}, // Carrega o Acumulador (LDA - 0x20 <endereço>).
    {"ADD", 0x30, 1}, // Adiciona ao Acumulador (ADD - 0x30 <endereço>).
    {"OR",  0x40, 1}, // Logica do OU (OR - 0x40 <endereço>).
    {"AND", 0x50, 1}, // Logica do E (AND - 0x50 <endereço>).
    {"NOT", 0x60, 0}, // Logica do NÃO (NOT - 0x60 0x00).
    {"JMP", 0x80, 1}, // Condicional de Pulo (JUMP - 0x80 <endereço>).
    {"JN",  0x90, 1}, // Não Pula (JN - 0x90 <endereço>).
    {"JZ",  0xA0, 1}, // Pula para Zero (JZ - 0xA0 <endereço>).
    {"HLT", 0xF0, 0}  // Finalização (HLT - 0xF0 0x00).
};

// Contabiliza o número total de itens na tabela acima.
const int NUM_INSTRUCOES = sizeof(instrucoes) / sizeof(Instrucao);

// Procura uma instrução na tabela de instruções.
// Retorna 1 se encontrada, 0 caso contrário. Preenche 'opcode_saida' e 'req_operando_saida'.
int proc_instrucao(const char *assembly_struct, unsigned char *opcode_saida, int *req_operando_saida) {
	int i;
    for (i = 0; i < NUM_INSTRUCOES; i++) {
        if (strcmp(assembly_struct, instrucoes[i].assembly) == 0) { // Analisa as instruções para dar sequência na instrução 'if'.
            *opcode_saida = instrucoes[i].opcode; // Transfere os valores para o ponteiro.
            *req_operando_saida = instrucoes[i].req_operando; // Transfere os valores para o ponteiro.
            return 1; // Se a condição(if) for verdadeira, instrução encontrada.
        }
    }
    return 0; // Se não for encontrada.
}

// Subcódigo para criar o arquivo de entrada padrão com conteúdo de exemplo caso ele não exista.
int criacao_entrada_arquivo_para_exemplo() {
    FILE *f_in = fopen(INPUT_FILENAME, "w"); // "w" cria ou sobrescreve.
    if (f_in == NULL) { // f_in igual a nulo.
        fprintf(stderr, "\nErro: Não foi possivel criar o arquivo '%s'.\n", INPUT_FILENAME); // Caso ocorra algum problema na criação do arquivo, essa mensagem aparecerá.
        return 1; // Se o 'if' verdadeiro, o arquivo não pode ser criado.
    }
    // Configuração de Entrada do arquivo "Entrada.asm" com instruções para o usuário.
    fprintf(f_in, ";\t|Montador Neander|\n");
    fprintf(f_in, "; 1. Este arquivo e editavel.\n");
    fprintf(f_in, "; 2. Linhas que comecam com ';' sao comentarios e serao ignoradas.\n");
    fprintf(f_in, "; 3. Cada instrucao ocupa 2 bytes na memoria de saida.\n");
    fprintf(f_in, "; 4. Abaixo esta um exemplo como base, altere-o para o codigo desejado.\n");
    fprintf(f_in, "\n");
    fprintf(f_in, "    LDA 80       \n");
    fprintf(f_in, "    ADD 81       \n");
    fprintf(f_in, "    STA 82       \n");
    fprintf(f_in, "    HLT          \n");
    fprintf(f_in, "    NOP          \n");
    fprintf(f_in, "    NOT          \n");
    fprintf(f_in, "\n");
    fclose(f_in);
    printf("\nArquivo de entrada padrao '%s' criado com exemplos.\n", INPUT_FILENAME); // Caso ainda não exista um arquivo de "Entrada.asm" essa mensagem aparecerá notificando que o código gerou um novo arquivo.
    return 0; // Finaliza essa partição do código.
}

int main() { // O código principal, de conversão de Assembly para OpCode ocorrerá nessa parte.
    FILE *input_file = NULL; // Ponteiro de entrada com valor incialmente nulo.
    FILE *output_file = NULL; // Ponteiro de saída com valor inicialmente nulo
    char linha[256]; // Vetor linha, para a leitura de valores impressos no arquivo.
    char *sinal; // Ponteiro incialmente vazio.
    char memoria_struct[10]; // Vetor utilizado para fins de valores Assembly.
    int operando_val; // Inteiro para identificar o valor do operando.
    unsigned char opcode; // Para as conversões em OpCode.
    int req_operando; // Inteiro para requerimento de operando.
    int linha_numero = 0; // Identificador de valores - entre as linhas.
    int req_erro = 0; // Para os casos de Erro.
    int i; // Usado nas instruções 'for'.
    char *memoria_sinal = NULL; // Ponteiro que inicialmente está com valor nulo.
    char *terminal; // Valores de termino do código.
    char *operando_terminal; // Operandos de finalização de código.
    
    printf("Iniciando Montador Neander...\n"); // Perfumaria para identificação de que o montador iniciou.

    // 1ª Operação: Verifica se o arquivo de entrada existe. Se não, cria um com exemplos.
    FILE *checar_file = fopen(INPUT_FILENAME, "r"); // "r" lê o conteúdo de um arquivo.
    if (checar_file == NULL) { // checar_file igual a nulo.
        if (criacao_entrada_arquivo_para_exemplo() != 0) { // 'if' para o caso de não conseguir criar o arquivo.
            return 1; // Sai se não conseguir criar o arquivo.
        }
    } else {
        fclose(checar_file); // Fecha o arquivo de checagem caso o arquivo já exista.
    }

    // 2ª Operação: Pausa para o usuário editar o arquivo.
    printf("\n>>> Edite o arquivo '%s' com o codigo Assembly desejado.\n", INPUT_FILENAME); // Identificação do código criado e petição para que o usuário edite-o.
    printf(">>> Salve as suas alteracoes e pressione <ENTER> para continuar o programa...\n"); // Quando finalizar as modificações apertar a tecla 'ENTER'.
    getchar(); // Aguarda o usuário pressionar ENTER.

    // 3ª Operação: Abre o arquivo de entrada para leitura.
    input_file = fopen(INPUT_FILENAME, "r"); // "r" lê o conteúdo de um arquivo.
    if (input_file == NULL) { // 'input_file' igual a nulo.
        fprintf(stderr, "Erro 1: Nao foi possivel abrir o arquivo de entrada '%s' para leitura.\n", INPUT_FILENAME); // Caso ocorra erros na abertura do arquivo, essa mensagem será apresentada.
        return 1; // Finaliza o programa.
    }

    // 4ª Operação: Cria o arquivo de saída para escrita binária.
    output_file = fopen(OUTPUT_FILENAME, "wb"); // Inicia um arquivo de escrita com a função "wb" a fim de armazenar valores binários, para que o Neander aceite os valores correspondêntes.
    if (output_file == NULL) { // 'output_file' igual a nulo.
        fprintf(stderr, "Erro 2: Nao foi possivel criar o arquivo de saida '%s'.\n", OUTPUT_FILENAME); // No caso de erros na criação do arquivo de saída.
        fclose(input_file); // Garante que o arquivo de entrada seja fechado.
        return 1; // Finaliza o programa.
    }
    // Escreve o cabeçalho do arquivo "Saida.mem"
    unsigned char comunic[] = {0x03, 0x4E, 0x44, 0x52}; // Para o perfeito funcionamento do código, o 'comunic' irá informar os valores do cabeçalho.
    fwrite(comunic, sizeof(unsigned char), 4, output_file); // Processa os valores do cabeçalho.

    // 5ª Operação: Processa cada linha do arquivo de entrada.
    while (fgets(linha, sizeof(linha), input_file) != NULL) { // Condição para que a função 'while' ocorra.
        ++linha_numero; // 'linha_numero' + 1.
        linha[strcspn(linha, "\n")] = '\0'; // Remove o caractere de nova 'linha' '\n'.
        
        if (strlen(linha) == 0 || linha[0] == ';') { // Pula linhas vazias ou linhas de comentário (começando com ';').
            continue;
        }

        char *linha_correspondente_copia = strdup(linha); // Duplica a 'linha', pois modifica a string original.
        if (linha_correspondente_copia == NULL) { // 'linha_correspondente_copia' igual a nulo.
            fprintf(stderr, "Erro 3: na linha %d: Falha na alocacao de memoria.\n", linha_numero); // Caso alguma linha de memória não possa ser lida, essa mensagem aparecerá.
            req_erro = 1; // 'req_erro' igual a 1.
            break; // Para o processamento em caso de erro de memória.
        }

        sinal = strtok(linha_correspondente_copia, " \t"); // Obtém o primeiro 'sinal' (pode ser um valor Assembly ou um número de linha).
        if (sinal == NULL) { // 'sinal' igual a nulo.
            free(linha_correspondente_copia); // Libera a memória alocada dinamicamente na 'linha_correspondente_copia'.
            continue; // Linha vazia, ou apenas espaços em branco.
        }

        long num_checar = strtol(sinal, &terminal, 10); // Tenta converter o primeiro 'sinal' para um número (para números de linha).

        if (*terminal == '\0' && sinal != terminal) { // Se o primeiro 'sinal' foi um número válido (e não apenas parte de uma palavra).
            memoria_sinal = strtok(NULL, " \t"); // O valor da memória é o próximo 'sinal'.
            if (memoria_sinal == NULL) { // 'memoria_sinal' igual a nulo.
                fprintf(stderr, "Aviso na linha %d: Numero de linha '%ld' sem instrucao. Ignorando.\n", linha_numero, num_checar); // Avisa, caso tenha alguma linha sem intrução, e que tal linha foi ignorada.
                free(linha_correspondente_copia); // Libera a memória alocada dinamicamente na 'linha_correspondente_copia'.
                continue;
            }
        } else {
            memoria_sinal = sinal; // O primeiro 'sinal' não era um número, então é um valor Assembly.
        }

        strncpy(memoria_struct, memoria_sinal, sizeof(memoria_struct) - 1); // Copia o valor da memória e converte letras minúsculas para maiúsculas para comparação.
        memoria_struct[sizeof(memoria_struct) - 1] = '\0'; // Garante terminação nula.

        for (i = 0; memoria_struct[i]; ++i) {
            memoria_struct[i] = (char)toupper((unsigned char)memoria_struct[i]); // Acessa valores inseridos no 'memoria_struct'.
        }
        
        if (!proc_instrucao(memoria_struct, &opcode, &req_operando)) { // Encontra a instrução na tabela (opcode e se tem operando).
            fprintf(stderr, "Erro 4: na linha %d: Instrucao desconhecida '%s'.\n", linha_numero, memoria_struct); // Se alguma instrução não existir entre as computadas, ele retorna essa mensagem.
            req_erro = 1; // 'req_erro' igual a 1.
            free(linha_correspondente_copia); // Libera a memória alocada dinamicamente na 'linha_correspondente_copia'.
            continue;
        }

        fwrite(&opcode, sizeof(unsigned char), 1, output_file); // Escreve o opcode no arquivo de saída.
        
        unsigned char byte_zero = 0x00; // Sempre adiciona o byte 0x00 após o opcode.
        fwrite(&byte_zero, sizeof(unsigned char), 1, output_file);

        if (req_operando) {
            sinal = strtok(NULL, " \t"); // Se a instrução requer um operando, lê o próximo 'sinal'.
            if (sinal == NULL) { // 'sinal' igual a nulo.
                fprintf(stderr, "Erro 5: na linha %d: Instrucao '%s' requer um operando, mas nenhum foi fornecido.\n", linha_numero, memoria_struct); // Caso tenha-se esquecido um valor de operando (ex: 80).
                req_erro = 1; // 'req_erro' igual a 1.
                free(linha_correspondente_copia); // Libera a memória alocada dinamicamente na 'linha_correspondente_copia'.
                continue;
            }

            operando_val = (int)strtol(sinal, &operando_terminal, 10); // Converte o operando para um valor inteiro.

            if (*operando_terminal != '\0' || operando_val < 0 || operando_val > 255) { // Valida o operando: deve ser um número e estar no intervalo de 0-255.
                fprintf(stderr, "Erro 6: na linha %d: Operando invalido ou fora do alcance (0-255) para '%s'.\n", linha_numero, memoria_struct); // Informativo que um valor ultrapassou o operando.
                req_erro = 1; // 'req_erro' igual a 1.
                free(linha_correspondente_copia); // Libera a memória alocada dinamicamente na 'linha_correspondente_copia'.
                continue;
            }

            unsigned char operando_byte = (unsigned char)operando_val; // Escreve o byte do operando.
            fwrite(&operando_byte, sizeof(unsigned char), 1, output_file);
            
            fwrite(&byte_zero, sizeof(unsigned char), 1, output_file); // Adiciona o byte 0x00 após o operando.
        }
        free(linha_correspondente_copia); // Libera a memória alocada dinamicamente na 'linha_correspondente_copia'.
    }

    // 6ª Operação: Fecha os arquivos.
    fclose(input_file);
    fclose(output_file);

    // 7ª Operação: Mensagem final.
    if (req_erro) { 
        printf("\nMontagem concluida com ERROS. Verifique os código acima.\n");
        return 1; // Indica falha.
    } else {
        printf("\nMontagem concluida com SUCESSO. Arquivo de saida: '%s' criado.\n", OUTPUT_FILENAME);
        return 0; // Indica sucesso.
    }
}
