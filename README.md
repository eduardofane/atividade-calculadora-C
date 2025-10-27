#include <stdio.h>
#include <math.h>  
#include <stdlib.h>
#include <ctype.h>
#include <string.h> 

#define M_PI 3.14159
#define MAX_HISTORICO 50
#define MAX_ESTATISTICA 100 

typedef struct Calculadora {
    char tipo[30]; 
    double num1, num2;
    double resultado;
    int id;
} Calculadora;


int comparar_doubles(const void *a, const void *b) {
    double val_a = *(const double *)a;
    double val_b = *(const double *)b;
    if (val_a < val_b) return -1;
    if (val_a > val_b) return 1;
    return 0;
}


double calcular_mmc(int a, int b) {
    if (a == 0 || b == 0) return 0.0;
    int abs_a = abs(a);
    int abs_b = abs(b);
    int max = (abs_a > abs_b) ? abs_a : abs_b;
    int mmc = max;
    while (1) {
        if (mmc % abs_a == 0 && mmc % abs_b == 0) {
            return (double)mmc;
        }
        mmc += max;
    }
}


void op_estatistica(const char *op_tipo, Calculadora historico[], int *contador_historico) {
    double dados[MAX_ESTATISTICA];
    int n, i;
    double soma = 0.0;
    Calculadora calculo;
    int operacao_sucesso = 1;

    printf("\n--- Calculo de %s ---\n", op_tipo);
    printf("Quantos elementos (Max %d) deseja inserir? ", MAX_ESTATISTICA);
    if (scanf("%d", &n) != 1 || n <= 0 || n > MAX_ESTATISTICA) {
        printf("Erro! Numero de elementos invalido.\n");
        return;
    }

    printf("Digite os %d elementos:\n", n);
    for (i = 0; i < n; i++) {
        printf("Elemento %d: ", i + 1);
        if (scanf("%lf", &dados[i]) != 1) {
            printf("Erro na leitura do elemento. Abortando.\n");
            int c;
            while ((c = getchar()) != '\n' && c != EOF);
            return;
        }
        soma += dados[i];
    }

    strcpy(calculo.tipo, op_tipo);
    calculo.num1 = (double)n; 
    calculo.num2 = soma; 

    if (strcmp(op_tipo, "Media") == 0) {
        calculo.resultado = soma / n;
    } else if (strcmp(op_tipo, "Mediana") == 0) {
     
        qsort(dados, n, sizeof(double), comparar_doubles);
        if (n % 2 != 0) { 
            calculo.resultado = dados[n / 2];
        } else { 
            calculo.resultado = (dados[n / 2 - 1] + dados[n / 2]) / 2.0;
        }
    } else if (strcmp(op_tipo, "Desvio-Padrao") == 0) {
        double media = soma / n;
        double variancia_soma = 0.0;
        for (i = 0; i < n; i++) {
            variancia_soma += pow(dados[i] - media, 2);
        }
        
        calculo.resultado = sqrt(variancia_soma / n);
    }
    
    printf("Resultado (%s): %.4lf\n", op_tipo, calculo.resultado);

   
    if (operacao_sucesso && *contador_historico < MAX_HISTORICO) {
        calculo.id = *contador_historico + 1;
        historico[*contador_historico] = calculo; 
        (*contador_historico)++;
    } else if (*contador_historico >= MAX_HISTORICO) {
        printf("\nAVISO! Historico cheio (Max %d). O calculo nao foi salvo.\n", MAX_HISTORICO);
    }
}


void ler_matriz(double matriz[3][3], const char *nome) {
    printf("Preencha a Matriz %s (3x3):\n", nome);
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("Elemento %s[%d][%d]: ", nome, i + 1, j + 1);
            scanf("%lf", &matriz[i][j]);
        }
    }
}


void exibir_matriz(double matriz[3][3], const char *nome) {
    printf("\nMatriz %s:\n", nome);
    for (int i = 0; i < 3; i++) {
        printf("|");
        for (int j = 0; j < 3; j++) {
            printf(" %8.2lf ", matriz[i][j]);
        }
        printf("|\n");
    }
}


void op_matriz_3x3(const char *op_tipo) {
    double A[3][3], B[3][3], C[3][3] = {0};
    printf("\n--- Calculo de Matriz 3x3: %s ---\n", op_tipo);
    
    // Leitura das matrizes
    ler_matriz(A, "A");
    ler_matriz(B, "B");

    if (strcmp(op_tipo, "Adicao Matriz") == 0) {
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                C[i][j] = A[i][j] + B[i][j];
            }
        }
    } else if (strcmp(op_tipo, "Subtracao Matriz") == 0) {
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                C[i][j] = A[i][j] - B[i][j];
            }
        }
    } else if (strcmp(op_tipo, "Multiplicacao Matriz") == 0) {
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                C[i][j] = 0;
                for (int k = 0; k < 3; k++) {
                    C[i][j] += A[i][k] * B[k][j];
                }
            }
        }
    }

    exibir_matriz(A, "A");
    printf("%s\n", (strcmp(op_tipo, "Adicao Matriz") == 0) ? "+" : 
                   (strcmp(op_tipo, "Subtracao Matriz") == 0) ? "-" : "x");
    exibir_matriz(B, "B");
    printf("=\n");
    exibir_matriz(C, "Resultado");
    
   
    printf("\n[AVISO: Operacoes de Matriz nao sao salvas no historico.]\n");
}



void exibir_historico(Calculadora historico[], int contador) {
    printf("\n--- Historico de Calculos (%d / %d) ---\n", contador, MAX_HISTORICO);
    if (contador == 0) {
        printf("Historico vazio.\n");
        return;
    }

    for (int i = 0; i < contador; i++) {
        printf("[%d] Tipo: %-25s | ", historico[i].id, historico[i].tipo);
        
      
        if (strcmp(historico[i].tipo, "Media") == 0 ||
            strcmp(historico[i].tipo, "Mediana") == 0 ||
            strcmp(historico[i].tipo, "Desvio-Padrao") == 0) {
            printf("N: %-6.0lf | Soma: %-10.2lf | Resultado: %.4lf\n",
                   historico[i].num1, historico[i].num2, historico[i].resultado);
            continue;
        }

        printf("Op1: %-10.2lf", historico[i].num1);

       
        if (strcmp(historico[i].tipo, "Raiz Quadrada") != 0 &&
            strcmp(historico[i].tipo, "Log Natural") != 0 &&
            strcmp(historico[i].tipo, "Seno") != 0 &&
            strcmp(historico[i].tipo, "Cosseno") != 0 &&
            strcmp(historico[i].tipo, "Tangente") != 0 &&
            strcmp(historico[i].tipo, "Log Base 10") != 0 &&
            strcmp(historico[i].tipo, "Exponencial") != 0 &&
            strcmp(historico[i].tipo, "Valor Absoluto") != 0 &&
            strcmp(historico[i].tipo, "Fatorial") != 0 &&
            strcmp(historico[i].tipo, "Arredondar") != 0 &&
            strcmp(historico[i].tipo, "Area Circulo") != 0 &&
            strcmp(historico[i].tipo, "Euler") != 0) 
        {
            printf("| Op2: %-10.2lf", historico[i].num2);
        }

        printf("| Resultado: %.4lf\n", historico[i].resultado);
    }
}


void modo_continuo_binario(int operacao, Calculadora historico[], int *contador_historico) {
    Calculadora calculo;
    float quociente;
    const char *op_str;

    
    switch (operacao) {
        case 2: op_str = "Soma"; break;
        case 3: op_str = "Subtracao"; break;
        case 4: op_str = "Multiplicacao"; break;
        case 5: op_str = "Divisao"; break;
        case 6: op_str = "Potencia"; break;
        case 14: op_str = "Max"; break;
        case 15: op_str = "Min"; break;
        case 16: op_str = "Modulo"; break;
        case 17: op_str = "Hipotenusa"; break;
        case 20: op_str = "Modulo Fluente"; break;
        case 21: op_str = "Log Base N"; break;
        case 24: op_str = "MMC"; break; // NOVO: MMC
        default: return;
    }

    printf("\n--- Modo Continuo: %s (Digite 0 para Op1 para sair) ---\n", op_str);

    while (1) {
        int operacao_sucesso = 1;
        
        printf("Op1 para %s (ou 0 para sair): ", op_str);
        if (scanf("%lf", &calculo.num1) != 1) {
            printf("Entrada invalida. Saindo do modo continuo.\n");
            int c;
            while ((c = getchar()) != '\n' && c != EOF);
            break;
        }

        if (calculo.num1 == 0.0) {
            printf("Saindo do Modo Continuo de %s.\n", op_str);
            break;
        }

        printf("Op2 para %s: ", op_str);
        if (scanf("%lf", &calculo.num2) != 1) {
            printf("Entrada invalida. Saindo do modo continuo.\n");
            int c;
            while ((c = getchar()) != '\n' && c != EOF);
            break;
        }
        
        strcpy(calculo.tipo, op_str);

        switch (operacao) {
            case 2: calculo.resultado = calculo.num1 + calculo.num2; break;
            case 3: calculo.resultado = calculo.num1 - calculo.num2; break;
            case 4: calculo.resultado = calculo.num1 * calculo.num2; break;
            case 5: 
                if (calculo.num2 != 0) { calculo.resultado = calculo.num1 / calculo.num2; } 
                else { printf("Erro! Divisao por zero nao e permitida.\n"); operacao_sucesso = 0; }
                break;
            case 6: calculo.resultado = pow(calculo.num1, calculo.num2); break;
            case 14: calculo.resultado = fmax(calculo.num1, calculo.num2); break;
            case 15: calculo.resultado = fmin(calculo.num1, calculo.num2); break;
            case 16: 
                if (calculo.num2 != 0) { calculo.resultado = fmod(calculo.num1, calculo.num2); } 
                else { printf("Erro! Divisao por zero nao e permitida.\n"); operacao_sucesso = 0; }
                break;
            case 17: calculo.resultado = hypot(calculo.num1, calculo.num2); break;
            case 20: 
                if (calculo.num2 != 0) {
                    calculo.resultado = fmod(calculo.num1, calculo.num2);
                    quociente = floor(calculo.num1 / calculo.num2);
                    printf("%.2lf = %.0f * %.2lf + %.2lf\n", calculo.num1, quociente, calculo.num2, calculo.resultado);
                } else { printf("Erro! Divisor nao pode ser zero.\n"); operacao_sucesso = 0; }
                break;
            case 21: 
                if (calculo.num1 > 0 && calculo.num2 > 0 && calculo.num2 != 1) { calculo.resultado = log(calculo.num1) / log(calculo.num2); } 
                else { printf("Erro! Base invalida ou numero invalido.\n"); operacao_sucesso = 0; }
                break;
            case 24: 
                if (calculo.num1 >= 0 && floor(calculo.num1) == calculo.num1 && 
                    calculo.num2 >= 0 && floor(calculo.num2) == calculo.num2) {
                    calculo.resultado = calcular_mmc((int)calculo.num1, (int)calculo.num2);
                } else {
                    printf("Erro! MMC requer inteiros nao-negativos.\n");
                    operacao_sucesso = 0;
                }
                break;
        }

        if (operacao_sucesso) {
            if (operacao != 20) { 
                printf("Resultado: %.4lf\n", calculo.resultado);
            }
            if (*contador_historico < MAX_HISTORICO) {
                calculo.id = *contador_historico + 1;
                historico[*contador_historico] = calculo; 
                (*contador_historico)++;
            } else {
                printf("\nAVISO! Historico cheio (Max %d). O calculo nao foi salvo.\n", MAX_HISTORICO);
            }
        }
    }
}


void modo_continuo_unario(int operacao, Calculadora historico[], int *contador_historico) {
    Calculadora calculo;
    const char *op_str;

    switch (operacao) {
        case 7: op_str = "Raiz Quadrada"; break;
        case 8: op_str = "Log Natural"; break;
        case 9: op_str = "Seno"; break;
        case 10: op_str = "Cosseno"; break;
        case 11: op_str = "Tangente"; break;
        case 12: op_str = "Log Base 10"; break;
        case 13: op_str = "Arredondar"; break;
        case 18: op_str = "Area Circulo"; break; 
        case 19: op_str = "Exponencial"; break; 
        case 22: op_str = "Fatorial"; break;
        case 23: op_str = "Valor Absoluto"; break;
        default: return; 
    }

    printf("\n--- Modo Continuo: %s (Digite 0 para o numero para sair) ---\n", op_str);

    while (1) {
        int operacao_sucesso = 1;
        long long res_fat = 1; 
        
        printf("Numero para %s (ou 0 para sair): ", op_str);
        if (scanf("%lf", &calculo.num1) != 1) {
            printf("Entrada invalida. Saindo do modo continuo.\n");
            int c;
            while ((c = getchar()) != '\n' && c != EOF);
            break;
        }

        if (calculo.num1 == 0.0 && operacao != 23) { 
             printf("Saindo do Modo Continuo de %s.\n", op_str);
             break;
        }
        
        calculo.num2 = 0.0;
        strcpy(calculo.tipo, op_str);

        switch (operacao) {
            case 7: 
                if (calculo.num1 >= 0) { calculo.resultado = sqrt(calculo.num1); } 
                else { printf("Erro! Raiz de negativo nao e permitida.\n"); operacao_sucesso = 0; }
                break;
            case 8: 
                if (calculo.num1 > 0) { calculo.resultado = log(calculo.num1); } 
                else { printf("Erro! Logaritmo so para positivos.\n"); operacao_sucesso = 0; }
                break;
            case 9: calculo.resultado = sin(calculo.num1); break;
            case 10: calculo.resultado = cos(calculo.num1); break;
            case 11: calculo.resultado = tan(calculo.num1); break;
            case 12: 
                if (calculo.num1 > 0) { calculo.resultado = log10(calculo.num1); } 
                else { printf("Erro! Logaritmo decimal so para positivos.\n"); operacao_sucesso = 0; }
                break;
            case 13: calculo.resultado = round(calculo.num1); printf("Resultado arredondado: %.0lf\n", calculo.resultado); break;
            case 19: calculo.resultado = exp(calculo.num1); break;
            case 18: 
                if (calculo.num1 > 0) { calculo.resultado = M_PI * calculo.num1 * calculo.num1; } 
                else { printf("Erro! Raio deve ser positivo.\n"); operacao_sucesso = 0; }
                break;
            case 22: 
                if (calculo.num1 >= 0 && floor(calculo.num1) == calculo.num1) {
                    res_fat = 1;
                    for (int i = 1; i <= (int)calculo.num1; i++) { res_fat *= i; }
                    calculo.resultado = (double)res_fat;
                    printf("Resultado: %lld\n", res_fat);
                } else { printf("Erro! Fatorial so para inteiros nao-negativos.\n"); operacao_sucesso = 0; }
                break;
            case 23: calculo.resultado = fabs(calculo.num1); break;
        }

        if (operacao_sucesso) {
            if (operacao != 13 && operacao != 22) {
                 printf("Resultado: %.4lf\n", calculo.resultado);
            }

            if (*contador_historico < MAX_HISTORICO) {
                calculo.id = *contador_historico + 1;
                historico[*contador_historico] = calculo; 
                (*contador_historico)++;
            } else {
                printf("\nAVISO! Historico cheio (Max %d). O calculo nao foi salvo.\n", MAX_HISTORICO);
            }
        }
    }
}



int main() {
    Calculadora historico[MAX_HISTORICO];
    int Operacao;
    int contador_historico = 0;

    while (1) {
        if (contador_historico >= MAX_HISTORICO) {
            printf("\nAVISO! Historico cheio (Max %d). Novas operacoes nao serao salvas.\n", MAX_HISTORICO);
        }

        printf("\n-----------------------------------------------------------------------------\n");
        printf("Escolha a operacao pelo NUMERO (0=Sair):\n");
        
        // Coluna 1
        printf(" 1: Exibir Historico   | 13: Arredondar (A) \n");
        printf(" 2: Soma (+)           | 14: Max (M) \n"); 
        printf(" 3: Subtracao (-)      | 15: Min (I) \n");
        printf(" 4: Multiplicacao (*)  | 16: Modulo (Resto float) (O) \n");
        printf(" 5: Divisao (/)        | 17: Hipotenusa (H) \n");
        printf(" 6: Potencia (^)       | 18: Area do Circulo (PI) (P) \n"); 
        printf(" 7: Raiz Quadrada (R)  | 19: e^x (U) \n");
        printf(" 8: Log Natural (L)    | 20: Modulo Fluente (Z) \n"); 
        printf(" 9: Seno (S)           | 21: Log Base N (N) \n");
        printf("10: Cosseno (C)        | 22: Fatorial (F) \n");
        printf("11: Tangente (T)       | 23: Valor Absoluto (V) \n"); 
        printf("12: Log Base 10 (D)    | 24: MMC (Minimo Multiplo Comum) \n"); 

        printf("\n--- Estatistica ---\n");
        printf("25: Media | 26: Mediana \n");
        printf("27: Desvio-Padrão\n");

        printf("\n--- Matrizes 3x3 (Nao salva historico) ---\n");
        printf("28: Adicao Matrizes | 29: Subtracao Matrizes \n");
        printf("30: Multiplicacao Matrizes\n");

        printf(" 0: Sair (X)\n");
        printf("Opcao: ");
        
        if (scanf("%d", &Operacao) != 1) {
            printf("Entrada invalida. Digite um numero.\n");
            int c;
            while ((c = getchar()) != '\n' && c != EOF);
            continue;
        }

        if (Operacao == 0) {
            printf("Saindo da Calculadora. Ate logo!\n");
            break;
        }
        
        if (Operacao == 1) {
            exibir_historico(historico, contador_historico);
            continue;
        }
        
        if (Operacao == 25) {
            op_estatistica("Media", historico, &contador_historico);
            continue;
        } else if (Operacao == 26) {
            op_estatistica("Mediana", historico, &contador_historico);
            continue;
        } else if (Operacao == 27) {
            op_estatistica("Desvio-Padrao", historico, &contador_historico);
            continue;
        }

        if (Operacao == 28) {
            op_matriz_3x3("Adicao Matriz");
            continue;
        } else if (Operacao == 29) {
            op_matriz_3x3("Subtracao Matriz");
            continue;
        } else if (Operacao == 30) {
            op_matriz_3x3("Multiplicacao Matriz");
            continue;
        }


        
        if ((Operacao >= 2 && Operacao <= 6) || (Operacao >= 14 && Operacao <= 17) || Operacao == 20 || Operacao == 21 || Operacao == 24) {
            modo_continuo_binario(Operacao, historico, &contador_historico);
            continue;
        }
        
        
        if ((Operacao >= 7 && Operacao <= 13) || Operacao == 18 || Operacao == 19 || Operacao == 22 || Operacao == 23) {
            modo_continuo_unario(Operacao, historico, &contador_historico);
            continue;
        }

       
        printf("Opcao invalida (%d). Por favor, escolha um numero valido.\n", Operacao);
    }

    return 0;
}
